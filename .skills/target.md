# Global Skill Target & Architecture Directives

Hệ thống chỉ thị mục tiêu kiến trúc áp dụng bắt buộc và kế thừa xuống toàn bộ các kỹ năng con (`skills/*`).

---

## 1. Project Auto-Discovery Protocol (Giao thức tự động khảo sát dự án)

Trước khi thực hiện bất kỳ hành động sinh mã, cập nhật file cấu hình hoặc tư vấn kiến trúc, mọi sub-skill PHẢI tự động quét và phân tích ngữ cảnh dự án hiện tại:

1. **Nhận diện Manifest & Package Configuration:**
   - **Node.js / Web:** Kiểm tra `package.json`, `pnpm-workspace.yaml`, `lerna.json`, `tsconfig.json`, `next.config.js|ts`, `vite.config.js|ts`.
   - **Python:** Kiểm tra `pyproject.toml`, `requirements.txt`, `Pipfile`, `setup.py`, `poetry.lock`.
   - **Go:** Kiểm tra `go.mod`, `go.sum`.
   - **Rust:** Kiểm tra `Cargo.toml`, `Cargo.lock`.
   - **Java / Kotlin:** Kiểm tra `pom.xml`, `build.gradle`, `build.gradle.kts`.

2. **Khảo sát cấu trúc Repo (Repository Topology):**
   - Xác định mô hình: Monorepo (Turborepo, Nx, PNPM Workspaces) hay Polyrepo / Single-project.
   - Định vị cây thư mục nguồn: `src/`, `app/`, `pages/`, `lib/`, `packages/`, `apps/`, `services/`, `internal/`, `pkg/`.
   - Phát hiện các cấu hình môi trường mẫu: `.env.example`, `.env.template`.

3. **Khảo sát Design System & Style Tokens:**
   - Tìm kiếm cấu hình styling: `tailwind.config.js|ts`, `postcss.config.js`, `uno.config.ts`, CSS modules, SCSS, Vanilla Extract, StyleX, hoặc styled-components.
   - Quét file CSS gốc (`globals.css`, `index.css`, `app.css`) để trích xuất CSS variables (`--background`, `--foreground`, `--primary`, `--radius`, palette tokens).
   - Thích ứng tuyệt đối với tokens hiện có, không được tự ý ghi đè màu sắc nếu dự án đã có palette.

---

## 2. Universal Portability Target (Mục tiêu khả chuyển phổ quát)

1. **Framework-Agnostic & Stack-Agnostic Core:**
   - Mọi giải pháp và chỉ dẫn phải trừu tượng hóa các nguyên tắc kiến trúc cốt lõi (Separation of Concerns, Inversion of Control, Clean Boundaries) độc lập với ngôn ngữ.
   - Khi cung cấp mã mẫu hoặc giải pháp thực thi, tự động chuyển đổi sang cú pháp của framework mà dự án đích đang sử dụng (React, Vue, Svelte, Angular, Express, NestJS, FastAPI, Django, Gin, Actix-web).

2. **Zero Project-Assumption:**
   - Không được giả định dự án đích có cấu trúc thư mục hay quy ước đặt tên riêng của một khách hàng hay sản phẩm cụ thể.
   - Luôn sử dụng đường dẫn tương đối hoặc alias path được định nghĩa trong `tsconfig.json` / `pyproject.toml` của dự án đích (ví dụ `@/*`, `~/*`).

---

## 3. Production-Readiness Target (Mục tiêu sẵn sàng sản xuất)

1. **Type-Safety & Zero Placeholders:**
   - Luôn cung cấp mã nguồn có kiểu tường minh (Strict TypeScript, Python type hints `typing/mypy`, Go static typing).
   - Không chèn comment giữ chỗ sơ sài dạng `// TODO: implement later`, `// logic goes here`, `/* handle error */`. Mọi luồng logic được đề xuất phải hoàn chỉnh, có xử lý ngoại lệ (Error handling).

2. **Cross-Platform Compatibility:**
   - Mọi scripts, công cụ tự động hóa hoặc lệnh CLI đi kèm skill phải tương thích trên cả Linux, macOS và Windows (hỗ trợ cả PowerShell và POSIX shell / Git Bash / WSL).
   - Sử dụng chuẩn phân tách đường dẫn tương thích đa nền tảng (`path.join`, `pathlib.Path`), tránh hardcode dấu `/` hoặc `\` tùy tiện.

---

## 4. Cascade & Inheritance Rule (Quy tắc kế thừa tầng bậc)

1. **Hiệu lực tối thượng:** Mọi file `skills/<skill-name>/SKILL.md` đều kế thừa các định hướng tại file này.
2. **Quy tắc giải quyết xung đột:** Nếu có mâu thuẫn giữa hướng dẫn chi tiết trong sub-skill và `.skills/target.md`, ưu tiên cao nhất luôn thuộc về:
   $$\text{.skills/contraint.md} \;>\; \text{.skills/target.md} \;>\; \text{skills/<skill-name>/SKILL.md}$$
3. **Cơ chế nạp:** Agent khi kích hoạt bất kỳ skill con nào bắt buộc phải kiểm tra và áp dụng context của `.skills/target.md` vào giải pháp đưa ra cho người dùng.

---

## 5. Project-Specific Overrides (Template Parameters)

<!-- 
[CLONE-AND-GO TEMPLATE PLACEHOLDERS]
Khi clone repository này sang một dự án mới, người dùng có thể tùy ý điền các thông số bên dưới để định hình ngữ cảnh nhanh cho AI Agent (hoặc để trống để AI kích hoạt 100% cơ chế Auto-Discovery ở Mục 1):
-->

- **Project Name:** `[Tên dự án mới - e.g., MyAwesomeApp]`
- **Primary Tech Stack:** `[Framework & Runtime - e.g., Next.js 15 (App Router), TypeScript, Tailwind CSS, Prisma]`
- **Repository Structure:** `[Monorepo / Polyrepo / Single App - e.g., src/app, src/components, src/lib]`
- **Design System / Theme:** `[UI Tokens & Styling - e.g., shadcn/ui, Slate + Indigo palette, 8px radius]`
- **Custom Architecture Notes:** `[Bất kỳ quy ước kiến trúc bổ sung nào của riêng dự án này]`

