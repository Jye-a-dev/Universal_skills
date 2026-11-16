---
name: coding-skills
description: "Universal software architecture, clean code standards, and engineering workflows. Enforces Clean/Hexagonal Architecture, SOLID principles, Zero-Regression TDD, strict typing, max 400 lines per file, and zero-warning builds across any tech stack (TypeScript, Python, Go, Rust, Java). Trigger when writing, refactoring, or structuring production codebase logic."
---

# Universal Software Architecture & Clean Code Directive

Bộ quy chuẩn kỹ thuật toàn diện về Kiến trúc phần mềm, Clean Code và quy trình kỹ thuật sản xuất, áp dụng độc lập cho mọi dự án phần mềm và công nghệ.

---

## Global Target & Constraints

This skill strictly inherits and complies with:
- [target.md](../../.skills/target.md): Universal Portability & Project Auto-Discovery Protocol.
- [contraint.md](../../.skills/contraint.md): Security Red Lines, Zero Project Hardcoding, and Non-destructive Operations.

---

## 1. Clean Architecture & Layer Separation (Phân tách tầng kiến trúc)

Mã nguồn mọi hệ thống nghiệp vụ cần phân định rõ ràng các ranh giới kiến trúc độc lập:

1. **Domain Layer (Entities & Value Objects):**
   - Chứa quy tắc nghiệp vụ cốt lõi (Core Business Rules).
   - Tuyệt đối độc lập với frameworks, databases (ORM, SQL), giao thức mạng, hoặc thư viện bên ngoài.
   - Định nghĩa các Ports / Interfaces mà các tầng ngoài phải triển khai.

2. **Application / Use Case Layer:**
   - Điều phối luồng nghiệp vụ cụ thể của ứng dụng (Commands, Queries, Workflows).
   - Phụ thuộc vào Domain Layer; tương tác với hệ thống ngoài thông qua Interfaces; độc lập với giao thức truyền tải (HTTP, gRPC, CLI, WebSocket).

3. **Infrastructure Layer (Adapters & External Integrations):**
   - Triển khai cụ thể các Ports/Interfaces: Database Clients (Prisma, TypeORM, SQLAlchemy, GORM), Cache (Redis), Message Queues (Kafka, RabbitMQ), Email/SMS providers, External APIs.
   - Quản lý cấu hình kết nối, logging, secret loading và telemetry.

4. **Presentation / Transport Layer:**
   - Controllers, Routers, GraphQL Resolvers, RPC Handlers, UI Components.
   - Chịu trách nhiệm deserialize/validate request DTOs và serialize response; chuyển giao việc thực thi cho Use Cases; không chứa logic nghiệp vụ.

---

## 2. Zero-Regression Protocol (Quy trình chống hồi quy lỗi)

Mọi thay đổi (refactor, thêm tính năng, sửa lỗi) phải bảo đảm không phá vỡ tính tương thích ngược:

1. **Phân tích Dependency Graph:**
   - Trước khi sửa đổi bất kỳ Public API, Database Schema, hoặc Shared Interface nào, phải truy vết toàn bộ các điểm gọi hàm (callsites) và consumer downstream.
   - Nếu có breaking change, bắt buộc thiết lập lộ trình chuyển tiếp (Deprecation Warning $\to$ Migration Guide $\to$ Graceful Cutover).

2. **Contract-First & Schema Validation:**
   - Mọi payload trao đổi qua mạng (API REST, WebSocket, Event Bus) phải có Schema xác thực chặt chẽ (Zod, Pydantic, Protobuf, JSON Schema).
   - Migration dữ liệu phải kiểm tra tính tương thích hai chiều (Forward & Backward Compatible).

3. **Automated Verification:**
   - Mọi thay đổi code phải được kiểm thử tự động (Unit Test cho Domain Logic, Integration Test cho Adapters).
   - Chặn merge code nếu có bất kỳ test case nào thất bại hoặc độ bao phủ suy giảm nghiêm trọng.

---

## 3. Quy Tắc Giới Hạn File (Max 400 Lines per File)

Nhằm đảm bảo nguyên lý đơn nhiệm (Single Responsibility Principle) và duy trì khả năng đọc hiểu:

- **Giới hạn cứng:** Không một tệp mã nguồn nào (`.ts`, `.tsx`, `.py`, `.go`, `.rs`, `.java`) được vượt quá **400 dòng code**.
- **Giải pháp xử lý khi file phình to:**
  - Tách logic phụ trợ thành các helper modules hoặc custom hooks.
  - Tách các kiểu dữ liệu và interfaces ra file `types.ts` hoặc schema riêng.
  - Sử dụng Composition: Chia nhỏ component cha thành các sub-components độc lập.
  - Tách query/mutation hooks ra khỏi giao diện hiển thị.

---

## 4. Strict Typing & Zero-Warning Builds

- **Chống Type Escape:**
  - Tuyệt đối cấm sử dụng kiểu thoát `any`, `unknown` không có type guard, hoặc ép kiểu ép buộc bừa bãi (`as any`).
  - Mọi hàm công khai phải khai báo tường minh kiểu tham số đầu vào và kiểu trả về.
- **Loại bỏ Dead Code & Placeholders:**
  - Tuyệt đối không để lại các chú thích giữ chỗ sơ sài (`// TODO`, `// FIX ME`, `pass`).
  - Xóa bỏ toàn bộ biến không sử dụng (unused variables), imports thừa và console logs thử nghiệm.
- **Zero-Warning Standard:** Mã nguồn phải vượt qua trình linter (`eslint`, `ruff`, `golangci-lint`, `clippy`) và trình biên dịch với 0 errors và 0 warnings.

---

## 5. Xử Lý Lỗi An Toàn & Tường Minh (Safe Error Handling)

- Không bao giờ nuốt lỗi trong khối `catch` trống (`catch (e) {}` không xử lý).
- Phân biệt rõ ràng giữa **Lỗi nghiệp vụ dự kiến** (Expected Domain Errors - ví dụ `InsufficientFunds`, `UserNotFound`) và **Lỗi hệ thống bất ngờ** (Unexpected Infrastructure Errors - ví dụ `DatabaseTimeout`, `NetworkCrash`).
- Ưu tiên mô hình Result/Either hoặc Exception có kiểu cụ thể (Typed Custom Exceptions) thay vì quăng lỗi chuỗi thô (`throw new Error("fail")`).
