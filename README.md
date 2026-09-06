# AURA Smart Home

AURA là dự án nhà thông minh local-first: một laptop cũ làm **AURA Core** chạy Ubuntu Server + Docker + Home Assistant + ESPHome + Mosquitto; các node ESP32 làm công tắc/cảm biến; PC Windows chạy AI khi cần.

> Mục tiêu thiết kế: nhà vẫn hoạt động bình thường khi AI Node tắt. Home Assistant giữ trạng thái/automation cơ bản; AI chỉ là lớp hiểu ngôn ngữ và ra quyết định có kiểm soát.

## Bắt đầu từ đâu?

Nếu bạn là người mới, hãy đọc tài liệu từng bước:

- [Hướng dẫn dựng AURA từ A đến Z cho người mới](docs/AURA_BEGINNER_SETUP_GUIDE.md)
- [Docker Compose mẫu](deploy/compose.yaml)
- [Mosquitto config mẫu](deploy/mosquitto/mosquitto.conf)
- [ESPHome AURA Switch V1 mẫu](esphome/aura-switch-v1.example.yaml)
- [Quy tắc cho Codex/AI khi bảo trì repo](AGENTS.md)

## Baseline hiện tại - 2026-09-06

- Dell Latitude E7250 chạy Ubuntu Server 26.04.1 LTS.
- SSH headless hoạt động; đóng nắp không suspend.
- IP được reserve: `192.168.100.174`.
- Docker Engine + Docker Compose hoạt động.
- Home Assistant Container hoạt động.
- ESPHome Device Builder hoạt động.
- Mosquitto MQTT có authentication; publish/sub test PASS.
- Home Assistant đã kết nối MQTT.
- AURA Switch V1 YAML đã validate; đang chờ PCB để test thật.

## Nguyên tắc repo

Không commit mật khẩu, token, private key hoặc `secrets.yaml` thật. File trong repo chỉ là tài liệu/template; secrets phải được tạo riêng trên từng máy.
