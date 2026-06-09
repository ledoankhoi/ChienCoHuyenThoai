---
name: engineering-diagnose
description: Diagnose and fix bugs systematically — analyze, hypothesize, test, fix
---

## Quy trình Diagnose (mttpocock/skills style)

Khi gặp lỗi, thực hiện theo 4 bước:

### Bước 1: Analyze
- Đọc log lỗi, xác định file và dòng lỗi
- Tái hiện lỗi nếu có thể

### Bước 2: Gather Context
- Đọc code xung quanh vị trí lỗi
- Kiểm tra imports, types, và API contracts

### Bước 3: Hypothesize
- Xây dựng 1-3 giả thuyết nguyên nhân
- Chọn giả thuyết có khả năng nhất

### Bước 4: Fix & Verify
- Sửa lỗi với surgical change (tối đa 5 dòng)
- Chạy lint và test để xác nhận
- Nếu vẫn lỗi, quay lại bước 3

## Git Guardrails
- Không dùng `--force` push
- Không commit node_modules, .env, secrets
- Viết commit message rõ ràng
