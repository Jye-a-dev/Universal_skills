# Agent Discovery & Governance Protocol

Tất cả AI Coding Agents (Claude, Cursor, Copilot, Windsurf, Cline, Codex) khi khởi tạo ngữ cảnh trong workspace này **BẮT BUỘC** tuân thủ giao thức sau trước khi thực hiện bất kỳ tác vụ nào.

## 1. Giao thức Nạp quy chuẩn & Chỉ mục (Mandatory Init Phase)
Trước khi phân tích, chỉnh sửa mã hoặc đưa ra tư vấn, Agent bắt buộc đọc và kế thừa 3 tệp cấu hình cốt lõi theo thứ tự ưu tiên:

1. **Ràng buộc tối thượng:** [`.skills/contraint.md`](.skills/contraint.md)
   - Cấm tuyệt đối: Hardcode thông tin dự án riêng lẻ, lưu trữ credential, hạ rào bảo mật, thao tác xóa phá hủy dữ liệu.
2. **Mục tiêu kiến trúc:** [`.skills/target.md`](.skills/target.md)
   - Bắt buộc thực thi *Project Auto-Discovery Protocol*: Khảo sát manifest, topology và design tokens của dự án hiện hành trước khi sinh mã.
3. **Chỉ mục định tuyến kỹ năng:** [`skills/llms.txt`](skills/llms.txt)
   - Quét bảng chỉ mục kỹ năng (*Single-Pass Scan*) để nạp đúng `skills/<skill-name>/SKILL.md` theo yêu cầu của người dùng.

## 2. Quy tắc kích hoạt kỹ năng nhanh (Core Routing)
*(Đối với các chuyên ngành khác như Tài liệu Office, MCP, Testing, Data..., tra cứu trực tiếp trong `skills/llms.txt`)*

- **UI Styling & Design:**
  - Khởi tạo thẩm mỹ, typography độc bản từ đầu: nạp `skills/frontend-design/SKILL.md`.
  - Kỹ nghệ component, design tokens, WCAG AA, responsive: nạp `skills/ui-skills/SKILL.md`.
  - Dọn dẹp giao diện AI-generated (purple gradients, badge cliché): nạp `skills/unslop/SKILL.md`.
- **Hoạt ảnh GSAP:**
  - Dự án React / Next.js: bắt buộc dùng `skills/gsap-react/SKILL.md` (cấm dùng `gsap-core`).
  - Dự án Vue / Nuxt / Svelte: bắt buộc dùng `skills/gsap-frameworks/SKILL.md`.
  - Vanilla JS hoặc SVG thuần: dùng `skills/gsap-core/SKILL.md`.
- **Kỹ nghệ phần mềm & Kiến trúc:**
  - TDD, Clean Architecture, Refactoring, giới hạn 400 dòng/file: nạp `skills/coding-skills/SKILL.md`.
- **Bảo mật & Phân quyền:**
  - JWT, OAuth2, RBAC, AuthGuards, OWASP Top 10: nạp `skills/security-auth/SKILL.md`.

## 3. Thứ tự ưu tiên giải quyết xung đột
`.skills/contraint.md` > `.skills/target.md` > `AGENTS.md` > `skills/<skill-name>/SKILL.md`
