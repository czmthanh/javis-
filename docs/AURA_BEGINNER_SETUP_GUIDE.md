# AURA - Hướng dẫn dựng hệ thống từ A đến Z cho người mới

Tài liệu này được chia thành 3 phần để người mới dễ theo dõi. Hãy làm đúng thứ tự, không nhảy bước.

## Phần 1 - Phần cứng, USB, Ubuntu, SSH và mạng

Đọc: [01_HARDWARE_UBUNTU_NETWORK.md](01_HARDWARE_UBUNTU_NETWORK.md)

Bạn sẽ hiểu và làm được:

- AURA là hệ thống gì và các thành phần liên hệ với nhau thế nào.
- Cần chuẩn bị laptop, USB, Ethernet và phần cứng gì.
- Backup USB cứu hộ trước khi xóa.
- Tạo USB Ubuntu bằng Rufus.
- Boot UEFI trên Dell Latitude E7250.
- Cài Ubuntu Server từng màn hình.
- Bật OpenSSH và điều khiển laptop từ PC Windows.
- Tắt sleep/suspend để laptop chạy 24/7 khi đóng nắp.
- Reserve IP `192.168.100.174` trên router.
- Update Ubuntu và hiểu từng câu lệnh.

## Phần 2 - Docker, Home Assistant, ESPHome và MQTT

Đọc: [02_DOCKER_HOMEASSISTANT_ESPHOME_MQTT.md](02_DOCKER_HOMEASSISTANT_ESPHOME_MQTT.md)

Bạn sẽ hiểu và làm được:

- Cài Docker Engine từ repository chính thức.
- Hiểu GPG key, APT repository, Docker Engine, containerd và Compose.
- Cài Home Assistant Container.
- Chuyển từ `docker run` sang Docker Compose mà không mất dữ liệu.
- Cài ESPHome Device Builder.
- Tạo cấu hình AURA Switch V1, CH1=GPIO5 và CH2=GPIO6.
- Cài Mosquitto MQTT có username/password.
- Test publish/subscribe trước khi kết nối Home Assistant.
- Kết nối Home Assistant với broker MQTT.

## Phần 3 - Bảo trì, backup, clone, Git và AI safety

Đọc: [03_MAINTENANCE_CLONE_AI.md](03_MAINTENANCE_CLONE_AI.md)

Bạn sẽ hiểu và làm được:

- Phân biệt phần đã hoàn thành và phần chưa test.
- Xem log, restart service và cập nhật container an toàn.
- Backup AURA Core.
- Dựng lại hệ thống trên một máy mới.
- Clone repository mà không làm lộ secrets.
- Giao repository cho Codex/AI bảo trì an toàn.
- Hiểu vì sao AURA Controller phải nằm giữa LLM và Home Assistant.
- Xử lý các lỗi đã gặp thực tế như YAML trùng key, terminal `>` và MQTT.

## Link chính thức

- Ubuntu Server: https://ubuntu.com/download/server
- Rufus: https://rufus.ie/
- Docker Engine: https://docs.docker.com/engine/install/ubuntu/
- Home Assistant: https://www.home-assistant.io/installation/
- ESPHome: https://esphome.io/install/getting-started/
- Eclipse Mosquitto: https://mosquitto.org/
- Ollama: https://ollama.com/download
- Git: https://git-scm.com/downloads

## Baseline hiện tại

Ngày 06/09/2026:

- AURA Core chạy Ubuntu Server 26.04.1 LTS trên Dell E7250.
- SSH/headless PASS.
- Home Assistant + ESPHome + Mosquitto chạy bằng Docker Compose.
- MQTT publish/sub PASS và Home Assistant đã kết nối MQTT.
- AURA Switch V1 YAML đã validate nhưng PCB chưa về, vì vậy chưa được phép ghi là đã test phần cứng.

## Cho Codex

Trước khi sửa repo, Codex phải đọc [../AGENTS.md](../AGENTS.md). Mọi thay đổi phải giữ nguyên nguyên tắc: không commit secret, không tự thay GPIO, Compose phải validate và phần chưa test phải được ghi rõ là chưa test.
