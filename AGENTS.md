# AGENTS.md - Quy tắc cho Codex/AI khi làm việc với AURA

Tài liệu này áp dụng cho Codex và mọi AI/code agent sửa repository.

## Trước khi sửa

1. Đọc `README.md`.
2. Đọc `docs/AURA_BEGINNER_SETUP_GUIDE.md` và tài liệu chương liên quan.
3. Phân biệt rõ ba trạng thái: **đã test thực tế**, **đã validate nhưng chưa test phần cứng**, **mới đề xuất**.

## Quy tắc bắt buộc

- Không commit secret: mật khẩu MQTT, Wi-Fi password, token Home Assistant, API key, SSH private key, `secrets.yaml` thật.
- Không thay GPIO AURA Switch V1 nếu chưa có quyết định phần cứng mới. GPIO hiện chốt: CH1=GPIO5, CH2=GPIO6.
- Mọi thay đổi `compose.yaml` phải chạy `docker compose config` trước khi deploy.
- Mọi thay đổi ESPHome phải validate thành công trước khi flash.
- Không mô tả một hạng mục là hoàn thành nếu chưa có log/test thực tế chứng minh.
- Không cho LLM quyền gọi tùy ý Home Assistant API. Lớp AI phải đi qua AURA Controller/tool policy giới hạn.
- Với intent mơ hồ hoặc câu nhận xét, ưu tiên `do_nothing`/hỏi lại thay vì tự bật tắt thiết bị.

## Khi sửa tài liệu

- Viết cho người mới, giải thích từ viết tắt và ý nghĩa từng lệnh.
- Với mỗi lệnh quan trọng, ghi: lệnh làm gì, vì sao cần, nó thay đổi gì, cách kiểm tra, cách hoàn tác nếu phù hợp.
- Ưu tiên link tài liệu chính thức.
- Cập nhật mốc phiên bản/baseline khi có thay đổi đã kiểm chứng.

## Cấu trúc mong muốn

- `docs/` - tài liệu cho người mới và bảo trì.
- `deploy/` - template Docker/Mosquitto không chứa secrets.
- `esphome/` - YAML mẫu không chứa secrets.
- Runtime data của Home Assistant/Mosquitto/ESPHome không đưa vào Git.
