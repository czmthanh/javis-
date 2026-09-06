# AURA - 03 MAINTENANCE CLONE AI

## 22. Những gì đã hoàn thành

- Ubuntu Server 26.04.1 LTS trên Dell E7250.
- SSH headless hoạt động.
- Đóng nắp không suspend.
- DHCP reservation `192.168.100.174`.
- Docker Engine + Compose hoạt động.
- Home Assistant chạy và đã onboarding.
- ESPHome Device Builder hoạt động.
- AURA Switch V1 YAML validate thành công.
- Mosquitto MQTT có authentication.
- MQTT publish/sub test PASS.
- Home Assistant kết nối MQTT thành công.

## 23. Những gì chưa test

- PCB AURA Switch V1 chưa về nên chưa flash ESP32-C3 trên bo thật.
- Chưa test relay 220 VAC thật.
- Chưa kết nối sensor/microphone thật.
- AURA Controller chưa triển khai.
- AI Node chưa kết nối vào Home Assistant.

Không được viết tài liệu như thể các mục trên đã hoàn thành.

## 24. Bảo trì hàng ngày / hàng tuần

### Xem stack

```bash
cd ~/aura-stack
docker compose ps
```

### Xem log Home Assistant

```bash
docker logs homeassistant --tail 100
```

### Xem log ESPHome

```bash
docker logs esphome --tail 100
```

### Xem log Mosquitto

```bash
docker logs mosquitto --tail 100
```

### Restart một service

```bash
docker compose restart homeassistant
```

Thay tên service nếu cần.

### Restart toàn stack

```bash
docker compose restart
```

### Dừng stack

```bash
docker compose down
```

`down` xóa container/network do Compose tạo nhưng **không xóa bind-mounted config trên host**.

### Chạy lại

```bash
docker compose up -d
```

## 25. Cập nhật container an toàn

Trước khi update: backup config.

Sau đó:

```bash
cd ~/aura-stack
docker compose pull
docker compose up -d
```

- `pull`: tải image mới, chưa thay container đang chạy.
- `up -d`: recreate service nếu image/config thay đổi.

Kiểm tra:

```bash
docker compose ps
docker logs homeassistant --tail 50
```

Không nên auto-update vô điều kiện trên hệ smart-home production; cần biết bản mới có breaking changes hay không.

## 26. Backup tối thiểu cần giữ

Quan trọng nhất:

```text
/home/aura/homeassistant/config/
/home/aura/aura-stack/compose.yaml
/home/aura/aura-stack/esphome/
/home/aura/aura-stack/mosquitto/config/mosquitto.conf
/home/aura/aura-stack/mosquitto/config/password.txt   (BÍ MẬT - không đẩy Git)
/home/aura/aura-stack/mosquitto/data/
```

Tạo tar backup ví dụ:

```bash
mkdir -p ~/backups
sudo tar -czf ~/backups/aura-core-$(date +%F).tar.gz \
  /home/aura/homeassistant/config \
  /home/aura/aura-stack
```

- `tar`: đóng gói file.
- `-c`: create.
- `-z`: gzip.
- `-f`: tên file output.
- `$(date +%F)`: chèn ngày dạng YYYY-MM-DD.

**Trước khi đẩy Git phải loại secrets.** Không commit password.txt, Home Assistant secrets hoặc file chứa Wi-Fi password.

## 27. Clone AURA Core trên máy mới

Checklist cho người mới:

1. Chuẩn bị laptop/mini PC x86-64, RAM >= 4 GB, khuyến nghị 8 GB, SSD >= 64 GB, Ethernet.
2. Backup dữ liệu cũ.
3. Tạo USB Ubuntu Server.
4. Boot UEFI và cài Ubuntu.
5. Chọn OpenSSH.
6. SSH vào máy.
7. Tắt sleep nếu là laptop.
8. Reserve IP ở router.
9. Update Ubuntu.
10. Cài Docker Engine chính thức.
11. Clone repository AURA.
12. Tạo các thư mục data/config cần thiết.
13. Tạo MQTT password riêng tại máy, không lấy password từ Git.
14. Chạy `docker compose config`.
15. Chạy `docker compose up -d`.
16. Onboard Home Assistant.
17. Kết nối MQTT.
18. Tạo/khôi phục ESPHome nodes.
19. Chỉ sau khi validate mới flash phần cứng.

## 28. Git và repository

Clone:

```bash
git clone https://github.com/czmthanh/javis-.git
cd javis-
```

Không nên clone repo trực tiếp đè lên thư mục data đang chạy. Repo nên chứa **template + docs + cấu hình không bí mật**; data runtime đặt ngoài repo.

File `.gitignore` nên chặn:

```gitignore
# Secrets
**/password.txt
**/secrets.yaml
*.key
*.pem
.env

# Home Assistant runtime/database
*.db
*.db-shm
*.db-wal
__pycache__/

# ESPHome generated/build
.esphome/
```

## 29. Quy tắc để Codex/AI tiếp tục bảo trì repo

Khi giao việc cho Codex:

1. Đọc `README.md` và tài liệu này trước khi sửa.
2. Không thay đổi GPIO đã chốt nếu chưa có quyết định mới.
3. Không commit secret.
4. Mọi thay đổi Compose phải chạy `docker compose config`.
5. Mọi thay đổi ESPHome phải validate trước khi flash.
6. Ghi thay đổi vào CHANGELOG/DECISIONS nếu repo có.
7. Phân biệt rõ: đã test thực tế / mới thiết kế / đang chờ phần cứng.
8. Không tự mở quyền điều khiển thiết bị cho LLM.

## 30. Kiến trúc AI dự kiến tiếp theo

AURA Controller sẽ nằm giữa AI và Home Assistant. LLM không được gọi tùy ý mọi API. Controller chỉ cung cấp tool hạn chế, ví dụ:

```text
get_home_state()
turn_on(entity)
turn_off(entity)
activate_scene(scene)
do_nothing()
```

Policy: nếu người dùng chỉ nhận xét hoặc câu lệnh mơ hồ, ưu tiên `do_nothing` hoặc hỏi lại. Điều này xuất phát từ benchmark thực tế: Qwen đã từng over-action khi nghe câu “Hôm nay ngoài trời nóng thật”. Vì vậy an toàn không thể giao hoàn toàn cho LLM.

## 31. Xử lý sự cố nhanh

### SSH không vào được

Từ PC:

```powershell
ping 192.168.100.174
```

Nếu ping không được: kiểm tra LAN/router/IP reservation. Nếu ping được nhưng SSH lỗi: tại laptop kiểm tra:

```bash
sudo systemctl status ssh
```

### Home Assistant không mở port 8123

```bash
docker compose ps
docker logs homeassistant --tail 100
```

### ESPHome không mở 6052

```bash
docker logs esphome --tail 100
```

### MQTT kết nối lỗi

```bash
docker logs mosquitto --tail 100
```

Kiểm tra file config, quyền password file và username/password.

### Compose báo trùng key

Ví dụ:

```text
mapping key "mosquitto" already defined
```

Mở `compose.yaml`, tìm hai service có cùng tên và chỉ giữ một. Sau đó:

```bash
docker compose config
```

### Terminal hiện dấu `>` liên tục

Thường do quote `'` hoặc `"` chưa đóng. Nhấn `Ctrl+C` để hủy lệnh, rồi nhập lại một dòng ngắn hơn.

## 32. Quy tắc an toàn điện cho AURA Switch

PCB AURA Switch làm việc với 220 VAC. Chưa có PCB thật nên phần này chưa được nghiệm chứng toàn hệ thống.

- Không test AC khi bo chưa kiểm tra continuity/short.
- Giữ cách ly HV/LV.
- Không chạm bo khi đang cấp điện lưới.
- Test firmware/relay logic bằng nguồn thấp trước khi nối tải AC nếu có thể.
- Relay mặc định OFF.
- Dùng cầu chì/MOV đúng thiết kế.

## 33. Mốc phiên bản

**AURA Core Baseline 2026-09-06**

- Core online và headless.
- HA + ESPHome + Mosquitto chạy trong Docker Compose.
- MQTT integration PASS.
- AURA Switch V1 cấu hình đã validate, chờ PCB.
- Bước tiếp theo: xây AURA Controller bằng virtual entities trước khi PCB về.

---

## Phụ lục A - Lệnh quan trọng nhất

```bash
# Vào stack
cd ~/aura-stack

# Kiểm tra config
docker compose config

# Xem trạng thái
docker compose ps

# Chạy / cập nhật stack
docker compose up -d

# Log
docker logs homeassistant --tail 100
docker logs esphome --tail 100
docker logs mosquitto --tail 100

# Update Ubuntu
sudo apt update && sudo apt upgrade -y

# Xem IP
ip -4 addr show eno1

# Xem RAM
free -h

# Xem uptime
uptime
```

## Phụ lục B - Những thông tin không được đưa lên Git

- Password MQTT.
- Wi-Fi SSID/password nếu private.
- `secrets.yaml` chứa token/password.
- Home Assistant long-lived access token.
- SSH private key.
- API keys của các dịch vụ.

Tài liệu Git phải dùng placeholder như `CHANGE_ME`, `${MQTT_PASSWORD}` hoặc hướng dẫn tạo secret tại máy.
