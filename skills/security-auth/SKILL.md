---
name: security-auth
description: "Universal security, authentication, and authorization standards. Covers JWT, OAuth2/OIDC, session cookies, AuthGuards, route/API middleware, RBAC/ABAC permission checks, token rotation, rate limiting, OWASP Top 10 hardening, and secret management. Use when designing, implementing, or auditing security, auth flows, login/signup, access control, route protection, or API defenses across web, mobile, or backend systems."
---

# Universal Security, Auth & AuthGuard Directive

Quy chuẩn kỹ thuật toàn diện về Bảo mật, Xác thực (Authentication), Phân quyền (Authorization), và Hàng rào bảo vệ (AuthGuards / Route & API Middleware) áp dụng độc lập cho mọi dự án phần mềm.

---

## Global Target & Constraints

This skill strictly inherits and complies with:
- [target.md](../../.skills/target.md): Universal Portability & Project Auto-Discovery Protocol.
- [contraint.md](../../.skills/contraint.md): Security Red Lines, Zero Project Hardcoding, and Non-destructive Operations.

---

## 1. Authentication Architecture (Xác thực danh tính)

### 1.1. Lựa chọn cơ chế Auth theo ngữ cảnh dự án
Tự động phát hiện kiến trúc của dự án đích để áp dụng mô hình xác thực phù hợp:

1. **Stateful Session Cookies (Phù hợp Web/Monolith SSR - Next.js, Django, Rails):**
   - Lưu session ID trong HTTP-only Cookie (`HttpOnly; Secure; SameSite=Lax; Path=/`).
   - Session store đặt tại Redis hoặc Database.
   - Bắt buộc kích hoạt CSRF protection (Double Submit Cookie hoặc CSRF token header).

2. **Stateless JWT with Token Rotation (Phù hợp Mobile Apps, SPA tách biệt API, Microservices):**
   - **Access Token:** Ngắn hạn (15–30 phút). Mang theo `sub` (userId), `role`, `permissions`.
   - **Refresh Token:** Dài hạn (7–30 ngày), lưu trữ trong `HttpOnly` Cookie (web) hoặc Keychain / EncryptedSharedPreferences (mobile).
   - **Refresh Token Rotation (RTR):** Mỗi khi cấp Access Token mới, Refresh Token cũ bị thu hồi và thay thế bằng Refresh Token mới. Nếu phát hiện Refresh Token cũ được tái sử dụng, lập tức vô hiệu hóa toàn bộ session của user (chống Token Theft).

3. **OAuth2 / OIDC (Google, GitHub, Apple SSO):**
   - Sử dụng chuẩn PKCE (Proof Key for Code Exchange) cho cả SPA và Mobile client.
   - Xác thực `state` parameter để chống tấn công CSRF đăng nhập.

---

## 2. Authorization, AuthGuards & Route/API Middleware

Mọi hệ thống cần triển khai nguyên tắc **Defense in Depth** với 2 tầng kiểm soát quyền độc lập:

### 2.1. Tầng Client: Route Guards & UX Protection
- Chặn người dùng chưa xác thực truy cập vào route bảo vệ; tự động lưu `returnUrl` và chuyển hướng tới `/login`.
- Chặn người dùng thiếu quyền truy cập vào route quản trị; chuyển hướng tới trang `/403` (Access Denied). Tuyệt đối không redirect về `/login` nếu user đã đăng nhập.
- Xem chi tiết triển khai mẫu tại [auth-guards-and-rbac.md](reference/auth-guards-and-rbac.md).

### 2.2. Tầng Server: Endpoint Middleware Guards
- Mọi API endpoint nhận dữ liệu hoặc trả dữ liệu nhạy cảm BẮT BUỘC phải đi qua Auth Middleware / Guard.
- Xác thực chữ ký token, thời hạn hết hạn, và kiểm tra phân quyền cấp độ hạt mịn (Fine-grained Permissions) trước khi chạm tới Controller/Handler.
- Phân biệt rõ mã trạng thái HTTP:
  - **`401 Unauthorized`:** Token thiếu, hết hạn, hoặc chữ ký sai.
  - **`403 Forbidden`:** Danh tính hợp lệ nhưng không đủ quyền trong Role-Permission Matrix.

---

## 3. API Hardening & OWASP Top 10 Defenses

1. **Input Validation (Zero Trust):**
   - Validate 100% dữ liệu đầu vào (Body, Query, Params, Headers) bằng Schema Validator kiểu tĩnh (Zod, Joi, Pydantic, class-validator).
   - Khử khuẩn chuỗi đầu vào, chặn tuyệt đối việc nối chuỗi truy vấn SQL hoặc chèn HTML thô.

2. **Rate Limiting & Anti-Brute-Force:**
   - Áp dụng Rate Limit cho toàn bộ API công khai (ví dụ 100 req/phút/IP).
   - Áp dụng Rate Limit nghiêm ngặt cho auth endpoints (`/login`, `/register`, `/forgot-password`, `/reset-password`): tối đa 5 lần thử trong 15 phút.

3. **CORS & Security Headers:**
   - Khai báo Whitelist cụ thể cho Allowed Origins; cấm dùng `*` khi `credentials: true`.
   - Cài đặt đầy đủ Security Headers: CSP, HSTS, X-Frame-Options (`DENY`), X-Content-Type-Options (`nosniff`).
   - Xem chi tiết checklist và code mẫu tại [api-and-data-hardening.md](reference/api-and-data-hardening.md).

---

## 4. Secret & Token Hygiene

1. **Quản lý biến môi trường:**
   - Không commit `.env` chứa bí mật vào git. Luôn có file `.env.example` với các khóa mẫu.
   - Validate biến môi trường khi ứng dụng khởi động (Fail-Fast nếu thiếu JWT Secret hoặc Database URL).
2. **Mã hóa dữ liệu nhạy cảm (Encryption at Rest):**
   - Mật khẩu phải băm bằng `Argon2id` hoặc `bcrypt` với work factor $\ge 12$.
   - Thông tin cá nhân nhạy cảm (PII), API credentials lưu trữ trong database phải được mã hóa bằng `AES-256-GCM`.
