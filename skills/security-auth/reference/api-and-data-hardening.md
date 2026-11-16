# API Hardening & Data Protection Reference

Checklist kỹ thuật phòng chống lỗ hổng bảo mật OWASP Top 10 và quy chuẩn mã hóa dữ liệu.

---

## 1. OWASP Top 10 Hardening Checklist

| Hạng mục OWASP | Rủi ro chính | Giải pháp phòng chống bắt buộc |
| :--- | :--- | :--- |
| **A01: Broken Access Control** | Người dùng xem/sửa dữ liệu của người khác (IDOR) | Không tin cậy ID từ client gửi lên; luôn so khớp `ownerId == currentUserId` trong query ORM/SQL; áp dụng AuthGuards và RBAC trên mọi endpoint. |
| **A02: Cryptographic Failures** | Lộ lọt mật khẩu, token, dữ liệu PII | Mật khẩu băm bằng Argon2id/bcrypt (salt riêng, work factor $\ge 12$). Dữ liệu nhạy cảm lưu DB mã hóa bằng AES-256-GCM. Buộc dùng TLS 1.3. |
| **A03: Injection (SQL/NoSQL/Cmd)** | Kẻ tấn công thực thi mã/câu lệnh ngoài ý muốn | Dùng Parameterized Queries / Prisma / TypeORM / SQLAlchemy. Tuyệt đối không nối chuỗi SQL hoặc truyền dữ liệu thô vào `eval()`, `exec()`. |
| **A04: Insecure Design** | Thiếu Rate Limiting, Business Logic bypass | Áp dụng Rate Limit dựa trên IP + UserID (ví dụ: `express-rate-limit` hoặc Redis token bucket). Khóa tài khoản tạm thời sau 5 lần login sai. |
| **A05: Security Misconfiguration** | Lộ stacktrace, mở CORS bừa bãi, thiếu Headers | Tắt debug mode trên Production. Cấu hình HTTP Security Headers (Helmet). CORS chỉ cho phép whitelist domain cụ thể. |
| **A06: Vulnerable Components** | Dependencies có lỗ hổng CVE | Chạy `npm audit` / `pip audit` / `cargo audit` định kỳ trong CI/CD pipeline. Khóa phiên bản bằng lockfile (`pnpm-lock.yaml`, `poetry.lock`). |
| **A07: Identification & Auth Failures** | Brute force mật khẩu, session hijacking | Bắt buộc mật khẩu mạnh ($\ge 8$ ký tự, chữ hoa/thường, số, ký tự đặc biệt). Cookie session gắn cờ `HttpOnly`, `Secure`, `SameSite=Strict` hoặc `Lax`. |
| **A08: Software & Data Integrity** | Tải script độc hại, webhook giả mạo | Xác thực HMAC SHA-256 signature trên mọi Webhook đến (Stripe, GitHub, payment gateways). Kiểm tra Subresource Integrity (SRI) cho CDN scripts. |
| **A09: Security Logging & Monitoring** | Không phát hiện được xâm nhập kịp thời | Log các sự kiện: Login thất bại, thay đổi mật khẩu, đổi quyền admin, 403 Forbidden. Tuyệt đối không log mật khẩu thô hoặc thẻ tín dụng vào log file. |
| **A10: Server-Side Request Forgery** | Server bị lừa gọi API nội bộ (SSRF) | Validate và whitelist URL khi fetch dữ liệu bên ngoài; chặn dải IP private (`127.0.0.1`, `10.0.0.0/8`, `192.168.0.0/16`, AWS metadata `169.254.169.254`). |

---

## 2. HTTP Security Headers Configuration

Bắt buộc cấu hình đầy đủ HTTP headers để chặn các cuộc tấn công Clickjacking, MIME sniffing và XSS.

### 2.1. Next.js (`next.config.js`)

```javascript
const securityHeaders = [
  { key: 'X-DNS-Prefetch-Control', value: 'on' },
  { key: 'Strict-Transport-Security', value: 'max-age=63072000; includeSubDomains; preload' },
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'origin-when-cross-origin' },
  {
    key: 'Content-Security-Policy',
    value: "default-src 'self'; script-src 'self' 'unsafe-eval' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; font-src 'self' data:;"
  }
];

module.exports = {
  async headers() {
    return [
      {
        source: '/:path*',
        headers: securityHeaders,
      },
    ];
  },
};
```

### 2.2. Express.js (`helmet`)

```typescript
import express from 'express';
import helmet from 'helmet';

const app = express();

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      objectSrc: ["'none'"],
      upgradeInsecureRequests: [],
    },
  },
  crossOriginEmbedderPolicy: true,
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  frameguard: {
    action: 'deny',
  },
}));
```

---

## 3. Data Protection & Encryption at Rest

### 3.1. Password Hashing (Argon2 / Bcrypt)

```typescript
import argon2 from 'argon2';

export async function hashPassword(plainPassword: string): Promise<string> {
  return await argon2.hash(plainPassword, {
    type: argon2.argon2id,
    memoryCost: 2 ** 16, // 64 MB
    timeCost: 3,
    parallelism: 1,
  });
}

export async function verifyPassword(hashedPassword: string, candidate: string): Promise<boolean> {
  return await argon2.verify(hashedPassword, candidate);
}
```

### 3.2. Symmetrical Encryption for PII (AES-256-GCM)

```typescript
import crypto from 'crypto';

const ALGORITHM = 'aes-256-gcm';
const IV_LENGTH = 12; // 96 bits for GCM

export function encryptField(text: string, secretKeyHex: string): string {
  const iv = crypto.randomBytes(IV_LENGTH);
  const key = Buffer.from(secretKeyHex, 'hex');
  const cipher = crypto.createCipheriv(ALGORITHM, key, iv);

  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  const authTag = cipher.getAuthTag().toString('hex');

  // Format: iv:authTag:encrypted
  return `${iv.toString('hex')}:${authTag}:${encrypted}`;
}

export function decryptField(encryptedPayload: string, secretKeyHex: string): string {
  const [ivHex, authTagHex, encryptedData] = encryptedPayload.split(':');
  const key = Buffer.from(secretKeyHex, 'hex');
  const decipher = crypto.createDecipheriv(ALGORITHM, key, Buffer.from(ivHex, 'hex'));

  decipher.setAuthTag(Buffer.from(authTagHex, 'hex'));
  let decrypted = decipher.update(encryptedData, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

---

## 4. Environment & Secret Hygiene

1. **File `.gitignore`:** Luôn đảm bảo `.env`, `.env.local`, `.env.production`, `*.pem`, `*.key`, `secrets/` có mặt trong `.gitignore`.
2. **File `.env.example`:** Luôn cung cấp `.env.example` có đầy đủ danh sách tên biến môi trường cùng mô tả, nhưng gán giá trị rỗng hoặc giá trị dummy công khai (ví dụ `JWT_ACCESS_SECRET=your_32_byte_secret_here`).
3. **Secret Validation tại Startup:** Sử dụng Zod hoặc Envalid để kiểm tra biến môi trường ngay khi server khởi động (Fail-Fast):

```typescript
import { z } from 'zod';

const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'test', 'production']).default('development'),
  PORT: z.string().default('3000').transform(Number),
  DATABASE_URL: z.string().url(),
  JWT_ACCESS_SECRET: z.string().min(32, 'Secret must be at least 32 characters'),
  CORS_ALLOWED_ORIGINS: z.string().transform(str => str.split(',')),
});

export const env = envSchema.parse(process.env);
```
