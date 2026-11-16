---
trigger: always_on
---

# Gemini & Antigravity Workspace Rules

Khi khởi tạo phiên làm việc hoặc phân tích bất kỳ yêu cầu nào trong workspace này, bạn **BẮT BUỘC** thực hiện quy trình nạp ngữ cảnh sau trước khi viết code:

1. **Đọc ràng buộc & Mục tiêu hệ thống:**
   - Đọc `.skills/contraint.md` để tuân thủ tuyệt đối các giới hạn bảo mật, chống hardcode và chống xóa dữ liệu.
   - Đọc `.skills/target.md` để thực thi *Project Auto-Discovery Protocol* (tự động nhận diện tech stack, cấu trúc thư mục và design tokens của dự án hiện tại).

2. **Định tuyến kỹ năng tự động (Skill Routing):**
   - Quét chỉ mục tại `skills/llms.txt` để tìm kỹ năng phù hợp nhất với yêu cầu của người dùng.
   - Khi xác định được kỹ năng phù hợp, bắt buộc đọc toàn bộ nội dung file `skills/<skill-name>/SKILL.md` (và các file trong `references/` nếu được chỉ định) trước khi triển khai.

3. **Thứ tự ưu tiên:**
   `.skills/contraint.md` > `.skills/target.md` > `.agent/rules/skills-discovery.md` > `skills/<skill-name>/SKILL.md`