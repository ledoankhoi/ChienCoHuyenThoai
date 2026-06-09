# Chiến Cơ Huyền Thoại — Agents & Context

## Dự án
Chiến Cơ Huyền Thoại là game mobile thể loại nhập vai / bắn súng với chiến cơ (spaceship).
Người chơi điều khiển phi công chính, sưu tầm chiến cơ qua gacha, kết hợp hiệu ứng nguyên tố để chiến thắng.

## Cơ chế cốt lõi
- **Chỉ số**: ATK, DEF, HP, CRIT, DMG CRIT, Tăng ATK cuối, Miễn ST cuối
- **Hệ đạn**: Lửa, Nước, Gió, Đất, Vô Tính, Ánh Sáng, Bóng Tối, Hỗn Độn
- **Hiệu ứng nguyên tố**: 10 tầng → Hiệu ứng Bậc 1 → 10 lần → Hiệu ứng Bậc 2
- **15 nhánh kết hợp chéo**: Băng+Bỏng, Xé Nát+Nứt Vỡ, ...
- **Hệ thống gacha**: 10 → S, 100 → SS ngẫu nhiên, 1000 → SS tự chọn
- **Nâng cấp**: Tăng sao, tăng cấp, tư chất, cường hóa (+)
- **Phẩm chất**: D (trắng), C (lục), B (xanh), A (tím), S (vàng), SS (đỏ)

## Team AI Agents
| Agent | Vai trò |
|---|---|
| @game-designer | Thiết kế cân bằng chỉ số, hiệu ứng, gameplay |
| @system-architect | Kiến trúc tổng thể game, database, API |
| @content-writer | Viết cốt truyện, lore, mô tả kỹ năng |
| @ui-ux-designer | Thiết kế giao diện mobile, UX flow |
| @data-analyst | Phân tích dữ liệu người chơi, cân bằng |
| @data-system | Thiết kế database, data pipeline |
| @animation | Animation cho chiến cơ, hiệu ứng, UI |
| @art-designer | Thiết kế concept art, nhân vật, icon |
| @qa-tester | Kiểm thử gameplay, bug, cân bằng |

## Tech Stack (tentative)
- **Game Engine**: Unity (C#) hoặc React Native
- **Backend**: Node.js (Express + database)
- **Database**: MongoDB hoặc PostgreSQL

---

## Engineering Rules
- Mỗi function ≤ 30 dòng, tối đa 2 cấp lồng, tối đa 3 tham số
- Dùng early return, tránh else không cần thiết
- Commit message: `<type>(<scope>): <subject>`
- Luôn chạy kiểm tra trước commit

<!-- CODEGRAPH_START -->
## CodeGraph
Project này có CodeGraph MCP server. Dùng codegraph tools để tra cứu cấu trúc code.
<!-- CODEGRAPH_END -->
