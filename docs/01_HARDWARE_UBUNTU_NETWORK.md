# AURA - 01 HARDWARE UBUNTU NETWORK

## 1. Ta đang xây cái gì?

Mục tiêu của AURA là biến một ngôi nhà bình thường thành nhà thông minh **mà không cần thay đổi thiết kế hiện có**. Các module ESP32 sẽ được giấu phía sau công tắc hoặc đặt tại các điểm cảm biến. Một laptop cũ làm máy chủ trung tâm 24/7. Máy tính Windows mạnh hơn sẽ chạy AI khi cần.

Kiến trúc hiện tại:

```text
Công tắc / cảm biến / ESP32
          │
          ├── ESPHome native API ─────┐
          └── MQTT ───────────────────┤
                                      ▼
                            AURA Core (laptop)
                       Ubuntu Server + Docker
                         Home Assistant Core
                         ESPHome + Mosquitto
                                      │
                                      ▼
                           AURA Controller (sau)
                                      │
                                      ▼
                          AURA AI Node (Windows)
                              Ollama + LLM
```

Điểm quan trọng: **nhà vẫn phải điều khiển được khi PC AI tắt**. Home Assistant chịu trách nhiệm trạng thái thiết bị và automation cơ bản; AI chỉ là lớp hiểu ngôn ngữ và ra quyết định có kiểm soát.

## 2. Phần cứng cần chuẩn bị

### 2.1 AURA Core đã dùng trong bản hiện tại

- Dell Latitude E7250.
- CPU Intel Core i5-5300U.
- RAM 8 GB (Ubuntu nhìn thấy khoảng 7.1 GiB).
- SSD mSATA 256 GB LITEON.
- Ethernet Intel I218-LM.
- Một USB 8 GB trở lên để tạo bộ cài Ubuntu; ta dùng USB 64 GB.
- Một bàn phím USB cho giai đoạn BIOS/cài Ubuntu nếu bàn phím laptop hỏng.
- Dây mạng Ethernet từ laptop tới router.

### 2.2 Máy AI Node hiện có

PC Windows vẫn được giữ để dùng cho công việc khác và AI. Ollama đã cài. Kiến trúc này tránh bắt AURA Core phải chạy LLM nặng.

### 2.3 Node đầu tiên - AURA Switch V1

PCB đang chờ về. Thiết kế hiện tại dùng ESP32-C3 Super Mini, 2 relay 5 V, nguồn AC-DC cách ly, đầu cảm biến và microphone. GPIO đã chốt:

| Chức năng | GPIO |
|---|---:|
| Relay CH1 | GPIO5 |
| Relay CH2 | GPIO6 |
| MIC BCLK | GPIO7 |
| MIC WS | GPIO4 |
| MIC DATA | GPIO3 |
| I2C SDA | GPIO1 |
| I2C SCL | GPIO10 |
| SENSOR GPIO A | GPIO20 |
| SENSOR GPIO B | GPIO21 |
| SENSOR ADC | GPIO0 |

## 3. Phần mềm và link tải chính thức

Chỉ nên tải từ trang chính thức:

- Ubuntu Server 26.04.1 LTS: https://ubuntu.com/download/server
- Rufus: https://rufus.ie/
- Docker Engine trên Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- Home Assistant: https://www.home-assistant.io/installation/
- ESPHome: https://esphome.io/install/getting-started/
- Eclipse Mosquitto: https://mosquitto.org/
- Ollama: https://ollama.com/download
- Git: https://git-scm.com/downloads
- Repository AURA hiện tại: https://github.com/czmthanh/javis-

## 4. Trước khi xóa laptop: sao lưu USB cứu hộ

USB cứu hộ thường có nhiều phân vùng, boot sector, EFI hoặc WinPE. **Copy file bằng Explorer không đảm bảo khôi phục được khả năng boot.** Vì vậy ta backup nguyên USB thành image.

Quy trình:

1. Mở Disk Management để xác định đúng USB.
2. Kiểm tra dung lượng và các phân vùng.
3. Dùng công cụ tạo raw image, ví dụ USB Image Tool.
4. Lưu file image ở ổ cứng khác.
5. Chỉ sau khi backup xong mới dùng Rufus ghi Ubuntu lên USB.

Trong hệ thống thực tế, USB 64 GB có hai phân vùng: một NTFS khoảng 54.75 GB và một FAT32 khoảng 5 GB. Đây là lý do phải backup nguyên disk.

## 5. Tạo USB cài Ubuntu bằng Rufus

Tải ISO Ubuntu Server và Rufus. Trong Rufus:

- Device: đúng USB cần xóa.
- Boot selection: chọn `ubuntu-26.04.1-live-server-amd64.iso`.
- Partition scheme: MBR.
- Target system: BIOS or UEFI.
- File system: Large FAT32.
- Persistence: 0.

**Vì sao chọn MBR + BIOS or UEFI?** Dell E7250 đời cũ hỗ trợ UEFI nhưng MBR cho bộ cài độ tương thích rộng hơn. Ta vẫn boot mục UEFI trong F12 để cài theo UEFI.

Nếu Rufus hỏi kiểu ghi, chọn ISO Image mode (Recommended).

## 6. Boot Dell Latitude E7250 từ USB

1. Tắt laptop.
2. Cắm USB, dây LAN và bàn phím USB.
3. Bật máy, nhấn F12 liên tục.
4. Trong One-Time Boot Menu chọn mục `UEFI: <tên USB>`.
5. Trong GRUB chọn `Try or Install Ubuntu Server`.

**Không chọn mục USB ở phần Legacy** vì ta muốn Ubuntu boot UEFI và tạo EFI boot entry đúng chuẩn.

## 7. Cài Ubuntu Server - giải thích từng màn hình

### 7.1 Loại cài đặt

Chọn `Ubuntu Server`, không chọn `Ubuntu Server (minimized)`.

**Tại sao?** AURA Core sẽ chạy Docker, Home Assistant, ESPHome, MQTT và công cụ quản trị. Bản Server chuẩn vẫn gọn nhưng ít thiếu gói tiện ích hơn.

### 7.2 Network

Ubuntu nhận card `eno1` và DHCP cấp IP. Trong lần cài thực tế IP là `192.168.100.174/24`.

Giữ DHCP ở giai đoạn cài đặt.

**Tại sao không đặt static IP ngay?** Giảm rủi ro nhập sai gateway/DNS làm mất mạng trong lúc cài. Sau khi hệ thống chạy ổn, ta reserve IP ở router.

### 7.3 Proxy

Để trống.

**Proxy là gì?** Proxy là máy trung gian cho HTTP/HTTPS. Router nhà không phải proxy, vì vậy không điền `192.168.100.1` vào ô này.

### 7.4 Ubuntu mirror

Giữ mirror mà installer tự chọn nếu kiểm tra thành công. Máy thực tế dùng mirror Việt Nam và tải package bình thường.

### 7.5 Storage

Chọn `Use an entire disk` vì laptop được dành riêng cho AURA Core và không giữ Windows.

Giữ `Set up this disk as an LVM group`.

Không bật LUKS encryption trong phiên bản hiện tại.

**LVM là gì?** LVM là lớp quản lý volume giúp việc thay đổi dung lượng linh hoạt hơn partition cố định.

**Vì sao không mã hóa toàn ổ?** AURA Core chạy headless 24/7. LUKS thường yêu cầu nhập passphrase khi boot; điều này gây khó cho máy tự khởi động lại sau mất điện.

### 7.6 Profile

Cấu hình đã dùng:

```text
Your name:          AURA
Server name:        aura-core
Username:           aura
Password:           tự đặt, không ghi vào tài liệu
```

Tên `aura-core` giúp dễ nhận ra máy trên mạng. User `aura` là tài khoản quản trị dùng với SSH và sudo.

### 7.7 Ubuntu Pro

Chọn `Skip for now`.

Ubuntu Pro không bắt buộc để AURA hoạt động. Có thể bật sau.

### 7.8 SSH

Chọn:

```text
[X] Install OpenSSH server
[X] Allow password authentication over SSH
```

**SSH là gì?** SSH cho phép điều khiển dòng lệnh của laptop từ PC khác qua mạng. Sau khi SSH hoạt động, không cần ngồi trước laptop nữa.

### 7.9 Featured server snaps

Không chọn gói nào.

**Tại sao?** Ta muốn tự quản lý Docker bằng Docker Engine chính thức, không trộn Docker Snap với Docker CE.

## 8. Lần boot đầu tiên và SSH

Sau khi installer báo Reboot Now:

1. Reboot.
2. Rút USB khi được yêu cầu.
3. Giữ dây LAN.
4. Chờ màn hình `aura-core login:`.

Từ Windows PowerShell:

```powershell
ssh aura@192.168.100.174
```

Lần đầu sẽ hỏi fingerprint. Gõ `yes`, sau đó nhập password.

Khi thấy:

```text
aura@aura-core:~$
```

thì đã điều khiển AURA Core từ xa thành công.

## 9. Kiểm tra hệ thống sau cài

### 9.1 Kiểm tra hostname và hệ điều hành

```bash
hostnamectl
```

**Lệnh làm gì?** Hiển thị tên máy, hệ điều hành, kernel, kiến trúc và model phần cứng. Không thay đổi hệ thống.

### 9.2 Kiểm tra IPv4 của Ethernet

```bash
ip -4 addr show eno1
```

- `ip`: công cụ xem/quản lý network.
- `-4`: chỉ IPv4.
- `addr show`: xem địa chỉ.
- `eno1`: card Ethernet của E7250.

Lệnh này chỉ đọc thông tin, không thay đổi mạng.

### 9.3 Kiểm tra RAM và swap

```bash
free -h
```

- `free`: xem RAM/swap.
- `-h`: human readable, hiển thị MiB/GiB dễ đọc.

Máy hiện dùng khoảng 500 MiB sau boot, còn hơn 6 GiB RAM trống; swap 4 GiB.

## 10. Cho laptop chạy 24/7 khi đóng nắp

Chạy:

```bash
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Giải thích:

- `sudo`: chạy lệnh với quyền quản trị.
- `systemctl`: quản lý các service/target của systemd.
- `mask`: khóa một unit bằng cách trỏ nó về `/dev/null`, mạnh hơn disable.
- `sleep.target`: sleep nói chung.
- `suspend.target`: suspend RAM.
- `hibernate.target`: hibernate ra disk.
- `hybrid-sleep.target`: kết hợp suspend + hibernate.

**Thay đổi sau lệnh:** Ubuntu không thể kích hoạt bốn trạng thái ngủ này bằng systemd. Điều này giúp laptop đóng nắp mà vẫn tiếp tục chạy server.

Kiểm tra:

```bash
systemctl status sleep.target suspend.target hibernate.target hybrid-sleep.target
```

Mong đợi `Loaded: masked` và `Active: inactive (dead)`.

Test thực tế: giữ SSH mở, đóng nắp 30 giây, chạy:

```bash
uptime
```

Nếu vẫn trả kết quả, laptop vẫn chạy.

**Khôi phục nếu muốn bật sleep lại:**

```bash
sudo systemctl unmask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

## 11. Cố định IP bằng DHCP Reservation

AURA Core đang dùng MAC Ethernet:

```text
84:7b:eb:0f:fb:23
```

Router FPT có LAN `192.168.100.1`. Ta reserve:

```text
MAC 84:7b:eb:0f:fb:23 -> IP 192.168.100.174
```

**Tại sao dùng reservation thay vì static IP trong Ubuntu?** Router vẫn quản lý subnet/gateway/DNS, nhưng luôn cấp cùng IP cho cùng MAC. Dễ bảo trì và giảm nguy cơ xung đột IP.

Lệnh lấy MAC:

```bash
ip link show eno1
```

Tìm dòng `link/ether`.

## 12. Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

Giải thích:

- `apt update`: tải danh sách package mới nhất; chưa cài gì.
- `&&`: chỉ chạy phần sau nếu phần trước thành công.
- `apt upgrade`: nâng các package đã cài lên phiên bản mới phù hợp.
- `-y`: tự trả lời Yes.

Ubuntu có thể báo một số package `Not upgrading yet due to phasing`. Đây là cơ chế phased update, không phải lỗi.
