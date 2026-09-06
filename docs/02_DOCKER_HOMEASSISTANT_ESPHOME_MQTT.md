# AURA - 02 DOCKER HOMEASSISTANT ESPHOME MQTT

## 13. Cài Docker Engine chính thức

### 13.1 Cài công cụ nền

```bash
sudo apt install -y ca-certificates curl
```

- `ca-certificates`: bộ chứng chỉ CA giúp HTTPS xác minh máy chủ.
- `curl`: tải dữ liệu qua HTTP/HTTPS từ terminal.

Kiểm tra:

```bash
curl --version
```

### 13.2 Tạo thư mục keyring

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

- `install -d`: tạo directory.
- `-m 0755`: owner được ghi, mọi người được đọc/đi vào.
- `/etc/apt/keyrings`: nơi lưu key xác minh repository bên thứ ba.

### 13.3 Tải khóa GPG của Docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo tee /etc/apt/keyrings/docker.asc > /dev/null
```

Giải thích theo từng phần:

- `curl -f`: báo lỗi nếu HTTP lỗi.
- `-s`: silent.
- `-S`: nếu lỗi vẫn hiện thông báo.
- `-L`: đi theo redirect.
- `|`: chuyển output của curl sang lệnh kế tiếp.
- `sudo tee ...`: ghi dữ liệu vào file cần quyền root.
- `> /dev/null`: không in lại nội dung key ra màn hình.

Sau lệnh này APT có key dùng để xác minh package từ Docker.

Cho phép APT đọc key:

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### 13.4 Thêm Docker repository

Bản AURA hiện tại dùng:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Giải thích:

- `dpkg --print-architecture`: trả `amd64` trên E7250.
- `. /etc/os-release`: nạp biến mô tả Ubuntu hiện tại.
- `UBUNTU_CODENAME`: bản 26.04 là `resolute`.
- `signed-by=...`: chỉ tin repo này nếu chữ ký khớp key Docker.
- `stable`: kênh package ổn định.
- `tee /etc/apt/sources.list.d/docker.list`: tạo nguồn package mới.

Sau đó:

```bash
sudo apt update
```

Ta đã xác nhận repo trả `resolute/stable amd64 Packages`, tức Docker hỗ trợ Ubuntu 26.04.

### 13.5 Cài Docker

```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Ý nghĩa:

- `docker-ce`: Docker Engine.
- `docker-ce-cli`: lệnh `docker`.
- `containerd.io`: runtime quản lý container.
- `docker-buildx-plugin`: build image hiện đại.
- `docker-compose-plugin`: lệnh `docker compose`.

Kiểm tra service:

```bash
sudo systemctl status docker --no-pager
```

Test thật:

```bash
sudo docker run --rm hello-world
```

Docker sẽ pull image `hello-world`, chạy container và xóa container sau khi kết thúc vì `--rm`.

Nếu thấy `Hello from Docker!`, Docker hoạt động.

### 13.6 Cho user aura chạy Docker không cần sudo

```bash
sudo usermod -aG docker aura
```

- `usermod`: sửa user.
- `-G docker`: thêm vào group docker.
- `-a`: append; rất quan trọng vì nếu thiếu `-a` có thể làm mất các group khác.

Đăng xuất SSH rồi đăng nhập lại để group mới có hiệu lực:

```bash
exit
```

Sau đó:

```powershell
ssh aura@192.168.100.174
```

Test:

```bash
docker ps
```

Nếu không permission denied thì xong.

## 14. Cài Home Assistant Container

Ta chọn Home Assistant Container vì AURA Core còn chạy ESPHome, Mosquitto và về sau AURA Controller dưới Docker Compose.

**Lưu ý:** Home Assistant Container không có hệ thống Apps/Add-ons giống Home Assistant OS; các dịch vụ phụ sẽ chạy thành container riêng. Đây là lựa chọn có chủ ý trong kiến trúc AURA.

### 14.1 Tạo thư mục config

```bash
mkdir -p ~/homeassistant/config
```

- `mkdir`: tạo thư mục.
- `-p`: tạo cả thư mục cha nếu cần và không lỗi nếu đã tồn tại.
- `~`: home của user `aura`, tức `/home/aura`.

### 14.2 Lần chạy đầu

Lệnh đã dùng:

```bash
docker run -d \
  --name homeassistant \
  --restart unless-stopped \
  --privileged \
  --network host \
  -e TZ=Asia/Ho_Chi_Minh \
  -v ~/homeassistant/config:/config \
  ghcr.io/home-assistant/home-assistant:stable
```

Giải thích:

- `docker run`: tạo và chạy container.
- `-d`: chạy nền.
- `--name homeassistant`: đặt tên dễ quản lý.
- `--restart unless-stopped`: tự chạy lại sau reboot/crash, trừ khi người quản trị chủ động stop.
- `--privileged`: cho container quyền thiết bị rộng hơn; hiện dùng để tránh giới hạn khi HA truy cập phần cứng local. Về sau có thể siết quyền nếu không cần.
- `--network host`: HA dùng trực tiếp network stack của máy host, thuận lợi cho auto-discovery trong LAN.
- `-e TZ=...`: timezone Việt Nam.
- `-v host:container`: mount config ra host để dữ liệu không mất khi xóa/recreate container.
- image `stable`: nhánh ổn định của Home Assistant.

Mở trình duyệt:

```text
http://192.168.100.174:8123
```

Tạo tài khoản admin. Bản hiện tại dùng username `aura-admin`.

## 15. Chuyển Home Assistant sang Docker Compose

`docker run` tốt để test nhanh, nhưng khi có nhiều service thì Compose dễ bảo trì hơn.

Tạo stack:

```bash
mkdir -p ~/aura-stack
cd ~/aura-stack
nano compose.yaml
```

Home Assistant service:

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - /home/aura/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped
    privileged: true
    network_mode: host
```

Kiểm tra YAML:

```bash
docker compose config
```

**Lệnh này không chạy container.** Nó parse YAML và in cấu hình chuẩn hóa. Nếu YAML sai thụt dòng hoặc trùng key, lệnh báo trước khi ta làm thay đổi hệ thống.

Chuyển container cũ sang Compose mà không mất dữ liệu:

```bash
docker stop homeassistant
docker rm homeassistant
docker compose up -d
```

**Tại sao không mất Home Assistant?** Config nằm ở `/home/aura/homeassistant/config` trên host, không nằm trong filesystem tạm của container.

Kiểm tra:

```bash
docker compose ps
```

## 16. Thêm ESPHome Device Builder

Trong `compose.yaml` thêm:

```yaml
  esphome:
    container_name: esphome
    image: ghcr.io/esphome/esphome:latest
    volumes:
      - /home/aura/aura-stack/esphome:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped
    network_mode: host
```

Sau đó:

```bash
docker compose config
docker compose up -d
docker compose ps
```

Mở:

```text
http://192.168.100.174:6052
```

Chọn `Expert` vì AURA dùng PCB custom và ta cần sửa YAML trực tiếp.

## 17. Cấu hình AURA Switch V1 trong ESPHome

Board: `ESP32-C3 Super Mini`.

Tên:

```yaml
esphome:
  name: aura-switch-v1
  friendly_name: AURA Switch V1
```

Hai relay:

```yaml
switch:
  - platform: gpio
    pin: GPIO5
    name: "AURA Switch CH1"
    id: relay_ch1
    restore_mode: RESTORE_DEFAULT_OFF

  - platform: gpio
    pin: GPIO6
    name: "AURA Switch CH2"
    id: relay_ch2
    restore_mode: RESTORE_DEFAULT_OFF
```

**Vì sao `RESTORE_DEFAULT_OFF`?** Khi ESP32 reboot hoặc mất điện rồi có điện lại, relay ưu tiên OFF thay vì tự bật tải ngoài ý muốn. Đây là default an toàn cho prototype.

ESPHome đã validate và báo `Configuration is valid!`. PCB chưa về nên chưa flash/test relay thật.

## 18. Cài MQTT Mosquitto

MQTT là message bus nhẹ. Không phải mọi ESPHome node đều cần MQTT vì ESPHome có native API. AURA dùng MQTT như bus chung cho những thành phần cần publish/subscribe tách rời, đặc biệt hữu ích cho controller/AI hoặc node không dùng native API.

### 18.1 Tạo thư mục

```bash
cd ~/aura-stack
mkdir -p mosquitto/config mosquitto/data mosquitto/log
```

### 18.2 Cấu hình Mosquitto

File `~/aura-stack/mosquitto/config/mosquitto.conf`:

```conf
persistence true
persistence_location /mosquitto/data/

log_dest stdout

listener 1883
allow_anonymous false

password_file /mosquitto/config/password.txt
```

Giải thích:

- `persistence true`: lưu state cần thiết ra disk.
- `persistence_location`: nơi lưu dữ liệu persistent.
- `log_dest stdout`: log đi ra Docker log.
- `listener 1883`: mở MQTT TCP cổng chuẩn 1883.
- `allow_anonymous false`: bắt buộc đăng nhập.
- `password_file`: file chứa hash password; không phải plaintext.

### 18.3 Tạo user MQTT

```bash
docker run --rm -it \
  -v ~/aura-stack/mosquitto/config:/mosquitto/config \
  eclipse-mosquitto:2 \
  mosquitto_passwd -c /mosquitto/config/password.txt aura
```

- Ta dùng image Mosquitto tạm thời chỉ để chạy tool `mosquitto_passwd`.
- `--rm`: xóa container tool sau khi xong.
- `-it`: cho nhập password tương tác.
- `-v`: mount thư mục config thật để file password được tạo trên host.
- `-c`: tạo file mới.
- `aura`: username MQTT.

**Không ghi password vào Git, tài liệu hay screenshot.**

Sửa quyền:

```bash
sudo chown 1883:1883 mosquitto/config/password.txt
sudo chmod 600 mosquitto/config/password.txt
sudo chown -R 1883:1883 mosquitto/data mosquitto/log
```

- UID/GID 1883 là user Mosquitto trong image.
- `chmod 600`: chỉ owner đọc/ghi password file.
- `chown`: giúp process trong container đọc/ghi đúng file/thư mục.

### 18.4 Service Mosquitto trong Compose

```yaml
  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto:2
    volumes:
      - /home/aura/aura-stack/mosquitto/config:/mosquitto/config
      - /home/aura/aura-stack/mosquitto/data:/mosquitto/data
      - /home/aura/aura-stack/mosquitto/log:/mosquitto/log
    restart: unless-stopped
    network_mode: host
```

Sau khi sửa Compose luôn chạy:

```bash
docker compose config
```

Trong quá trình dựng thực tế từng gặp lỗi:

```text
mapping key "mosquitto" already defined
```

Nguyên nhân: file có hai khối `mosquitto:` cùng cấp. Xóa khối bị lặp rồi chạy `docker compose config` lại.

## 19. Test MQTT trước khi kết nối Home Assistant

Terminal 1:

```bash
docker exec mosquitto mosquitto_sub -h localhost -u aura -P 'MAT_KHAU' -t 'aura/test' -C 1
```

Giải thích:

- `docker exec mosquitto`: chạy lệnh bên trong container đang chạy.
- `mosquitto_sub`: subscriber.
- `-h localhost`: broker cùng máy.
- `-u aura`: username.
- `-P`: password.
- `-t aura/test`: topic cần nghe.
- `-C 1`: nhận một message rồi thoát.

Terminal 2:

```bash
docker exec mosquitto mosquitto_pub -h localhost -u aura -P 'MAT_KHAU' -t 'aura/test' -m 'AURA MQTT OK'
```

- `mosquitto_pub`: publisher.
- `-m`: message.

Nếu terminal 1 hiện `AURA MQTT OK`, broker + authentication + publish + subscribe đều hoạt động.

**Lưu ý bảo mật:** mật khẩu đã từng xuất hiện trong một screenshot khi test. Phải đổi mật khẩu trước khi coi hệ thống là production.

## 20. Kết nối MQTT vào Home Assistant

Home Assistant:

`Settings -> Devices & services -> Add integration -> MQTT`

Điền:

```text
Broker:   192.168.100.174
Port:     1883
Protocol: 5
Username: aura
Password: mật khẩu MQTT hiện hành
```

Sau khi lưu, trang MQTT hiển thị broker `192.168.100.174`, tức kết nối thành công.

## 21. Compose hoàn chỉnh hiện tại

```yaml
services:
  homeassistant:
    container_name: homeassistant
    image: ghcr.io/home-assistant/home-assistant:stable
    volumes:
      - /home/aura/homeassistant/config:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped
    privileged: true
    network_mode: host

  mosquitto:
    container_name: mosquitto
    image: eclipse-mosquitto:2
    volumes:
      - /home/aura/aura-stack/mosquitto/config:/mosquitto/config
      - /home/aura/aura-stack/mosquitto/data:/mosquitto/data
      - /home/aura/aura-stack/mosquitto/log:/mosquitto/log
    restart: unless-stopped
    network_mode: host

  esphome:
    container_name: esphome
    image: ghcr.io/esphome/esphome:latest
    volumes:
      - /home/aura/aura-stack/esphome:/config
      - /etc/localtime:/etc/localtime:ro
    environment:
      - TZ=Asia/Ho_Chi_Minh
    restart: unless-stopped
    network_mode: host
```
