---
trigger: always_on
---

# Agent Discovery & Skills Orchestration Protocol

Tất cả AI Coding Agents khi khởi tạo phiên làm việc hoặc phân tích bất kỳ yêu cầu nào trong workspace này **BẮT BUỘC** thực hiện quy trình nạp ngữ cảnh theo thứ tự sau trước khi viết code:

1. **Ràng buộc & Phòng vệ cốt lõi:**
   - Đọc `.skills/contraint.md` để tuân thủ tuyệt đối các giới hạn bảo mật (OWASP, Zero Credential Leaks, No Bypass), chống hardcode dữ liệu riêng lẻ và chống phá hủy dữ liệu.

2. **Mục tiêu kiến trúc & Khảo sát dự án:**
   - Đọc `.skills/target.md` để thực thi *Project Auto-Discovery Protocol* (tự động nhận diện manifest, cấu trúc repo topology, CSS tokens và design system của dự án hiện tại).

3. **Định tuyến kỹ năng tự động (Skill Routing):**
   - Quét bảng chỉ mục tại `skills/llms.txt` để xác định kỹ năng phù hợp nhất với yêu cầu.
   - Nạp toàn bộ nội dung file `skills/<skill-name>/SKILL.md` (và các tài liệu trong `references/` nếu được chỉ định) trước khi triển khai.

## Thứ tự ưu tiên giải quyết xung đột
`.skills/contraint.md` > `.skills/target.md` > `.agent/rules/skills-discovery.md` > `skills/<skill-name>/SKILL.md`
