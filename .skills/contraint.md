# Global Skill Constraints & Prohibitions

Danh mục các điều cấm tuyệt đối áp dụng bắt buộc trên toàn bộ thư viện `skills/`.

---

## 1. Zero Project Hardcoding (Cấm tuyệt đối hardcode đặc thù dự án)

1. **Cấm tên dự án riêng lẻ:** Tuyệt đối không đề cập hoặc quy định tên thương hiệu/dự án cụ thể (như `"TrustPassz"`, tên client cá nhân) bên trong chỉ dẫn skill dùng chung.
2. **Cấm dải màu sản phẩm cố định:** Không hardcode các mã màu hex hoặc tên token của một sản phẩm cá biệt (ví dụ gán cứng chỉ được dùng nền `Slate-950` hay nhấn `Emerald-500` cho mọi app). Mọi styling phải trích xuất động từ design system của repo đích.
3. **Cấm đường dẫn monorepo cá biệt:** Không giả định các đường dẫn thư mục chuyên biệt như `apps/cl_user`, `apps/mb_user`, `apps/cl_admin`, `packages/contracts`. Chỉ dùng các quy ước tiêu chuẩn (`src/`, `app/`, `components/`, `lib/`).
4. **Cấm domain nội bộ & URL cục bộ:** Không hardcode các domain staging, production hay internal network IP vào mã nguồn chung.

---

## 2. Security Red Lines (Lằn ranh đỏ về bảo mật)

1. **Tuyệt đối không lưu trữ Credential:**
   - Cấm chèn API keys, JWT secret keys, DB passwords, private keys, access tokens vào mã nguồn, file cấu hình, tài liệu tham khảo hoặc test mocks.
   - Mọi credential phải thông qua biến môi trường (`process.env`, `os.environ`) hoặc Secret Manager Vault.

2. **Cấm vô hiệu hóa hàng rào bảo vệ (No Bypass):**
   - Không được phép tắt hoặc hướng dẫn tắt AuthGuards, JWT verification middleware, Role-Based Access Control (RBAC), Attribute-Based Access Control (ABAC).
   - Không cho phép cấu hình CORS mở toàn bộ: `origin: "*"` đi kèm `credentials: true`.
   - Không tắt CSRF protection trên các endpoint nhận cookie xác thực có trạng thái (stateful session).

3. **Cấm giải pháp mã hóa lỗi thời & Nguy cơ Injection:**
   - Cấm sử dụng các hàm băm yếu (`MD5`, `SHA1`, plain `SHA256` không salt) để lưu trữ mật khẩu người dùng. Bắt buộc dùng `Argon2id`, `bcrypt`, hoặc `PBKDF2`.
   - Cấm sử dụng `eval()`, `new Function()`, `exec()` với chuỗi đầu vào không khử khuẩn.
   - Cấm nối chuỗi thủ công để tạo truy vấn SQL / NoSQL (chống SQL Injection); luôn dùng Parameterized Queries, Prepared Statements hoặc Typed ORM/ODM.
   - Cấm chèn HTML thô (`dangerouslySetInnerHTML`, `v-html`) mà không qua bộ lọc Sanitizer (chống XSS).

---

## 3. Zero Resource Bloat & Trigger Hijacking (Cấm phình to tài nguyên & xung đột)

1. **Cấm nhúng Git Repo & Test Suites cồng kềnh:**
   - Không sao chép nguyên vẹn kho lưu trữ git bên ngoài vào thư mục skill (không chứa `.git`, `.github`, `.gitattributes`, Dockerfile của bên thứ ba).
   - Không chứa các thư mục `tests/` đồ sộ, `worked/` examples hàng nghìn file, hoặc dependencies (`node_modules/`, `venv/`). Mọi skill phải giữ dung lượng tinh gọn tối đa.
2. **Cấm trùng lặp tài nguyên (Zero Asset Duplication):**
   - Không nhân bản các bộ schema lớn (như XML Schemas XSD, fonts, fixtures) qua nhiều thư mục riêng lẻ. Phải tập trung về thư mục dùng chung `skills/_shared/`.
3. **Cấm chiếm dụng từ khóa (Anti-Trigger Hijacking):**
   - Mỗi sub-skill phải định nghĩa phạm vi hẹp và chính xác trong `description`.
   - Không được khai báo từ khóa bao quát làm vô hiệu hóa khả năng kích hoạt của các skill chuyên biệt khác (ví dụ: `gsap-core` không được chiếm quyền kích hoạt của `gsap-react` khi người dùng hỏi về React).

---

## 4. Zero Destructive Operations (Cấm thao tác phá hủy ngầm)

1. **Không tự ý xóa file:** Tuyệt đối không thực thi các lệnh xóa tệp mã nguồn (`rm -rf`, `Remove-Item`) hoặc drop database/tables của dự án đích khi chưa có sự xác nhận của người dùng.
2. **Không ghi đè cấu hình âm thầm:** Khi chỉnh sửa file cấu hình cốt lõi (`tsconfig.json`, `package.json`, `.env`), phải bảo tồn toàn bộ các cấu hình hiện hữu, chỉ bổ sung hoặc cập nhật trường cần thiết.
