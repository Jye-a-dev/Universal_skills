---
name: ui-skills
description: "Universal UI/UX Engineering and Design System execution. Enforces responsive mobile-first layouts, strict touch targets (>= 44px), zero layout shift (CLS), comprehensive component states (loading/empty/error/disabled), and WCAG 2.1 AA accessibility standards. Adapts dynamically to any project's existing design tokens and CSS variables. Trigger when implementing UI components, forms, responsive layouts, or auditing UI/UX engineering."
---

# Universal UI/UX Engineering & Design System Directive

Bộ quy chuẩn kỹ thuật UI/UX, Design System Tokens và tiêu chuẩn công thái học giao diện người dùng, áp dụng độc lập cho mọi dự án phần mềm Web, Mobile và Desktop.

---

## Global Target & Constraints

This skill strictly inherits and complies with:
- [target.md](../../.skills/target.md): Universal Portability & Project Auto-Discovery Protocol.
- [contraint.md](../../.skills/contraint.md): Security Red Lines, Zero Project Hardcoding, and Non-destructive Operations.

---

## 1. Project Token Auto-Discovery (Khảo sát Design Tokens)

Trước khi viết bất kỳ component hoặc styling nào, agent BẮT BUỘC phải trích xuất tokens hiện có của project đích:
1. **Quét CSS Variables:** Tìm file `globals.css` / `theme.css` để lấy `--background`, `--foreground`, `--primary`, `--secondary`, `--accent`, `--muted`, `--destructive`, `--border`, `--radius`.
2. **Quét Tailwind / Styling Config:** Tìm cấu hình trong `tailwind.config.ts`, `uno.config.ts`, hoặc theme object của dự án.
3. **Thích ứng động:** Không hardcode mã màu hex mới nếu token tương ứng đã tồn tại trong hệ thống. Luôn dùng utility classes hoặc CSS variables chuẩn của dự án:
   - Nền: `bg-background`, `bg-card`, `bg-muted`
   - Chữ: `text-foreground`, `text-muted-foreground`, `text-primary`
   - Viền: `border-border`, `border-input`

---

## 2. Tiêu Chuẩn Công Thái Học & Tương Tác Mobile-First

### 2.1. Vùng Chạm Tối Thiểu (Strict Touch Targets $\ge 44\text{px}$)
- **Quy tắc bất biến:** Mọi phần tử tương tác (Buttons, Inputs, Checkboxes, Dropdown triggers, Nút đóng X, Tabs, Icon buttons) BẮT BUỘC có diện tích chạm tối thiểu $44 \times 44\text{px}$ (`min-h-[44px] min-w-[44px]` hoặc `h-11`/`h-12`).
- **Khoảng cách giữa các hitbox (Separation):** Tối thiểu `8px` (`gap-2`) để loại bỏ tình trạng ngón tay chạm nhầm nút lân cận trên màn hình cảm ứng.
- **Nút hành động chính (Primary CTA):** Ưu tiên chiều cao `48px` đến `56px` trên mobile (`h-12` hoặc `h-14`, `w-full`).

### 2.2. Kiểm Soát Cử Chỉ Vuốt (Gestures)
- Với các thành phần kéo ngang (Sliders, Carousels, Horizontal Tabs), khai báo rõ thuộc tính CSS để tránh xung đột cuộn trang:
  ```css
  touch-action: pan-y;
  ```

---

## 3. Quản Lý Trạng Thái Component Toàn Diện (Component States Matrix)

Mọi component tương tác dữ liệu PHẢI được thiết kế và xử lý đầy đủ 5 trạng thái:
1. **Idle / Default:** Trạng thái sẵn sàng tương tác, rõ ràng về mặt phân cấp thị giác.
2. **Loading:** Sử dụng Skeleton Screens phù hợp với kích thước nội dung thật (tránh nhảy bố cục Layout Shift) hoặc Spinner/Indicators với `aria-busy="true"`.
3. **Empty / No Data:** Minh họa trạng thái trống rõ ràng, kèm thông điệp hướng dẫn hành động tiếp theo (CTA tạo mới dữ liệu).
4. **Error:** Hiển thị thông báo lỗi cụ thể, dễ hiểu kèm nút "Thử lại" (Retry Action).
5. **Disabled:** Làm mờ rõ rệt (`opacity-50 pointer-events-none cursor-not-allowed`), đảm bảo thuộc tính `aria-disabled="true"`.

---

## 4. Tối Ưu Bố Cục & Chống Giật Màn Hình (Zero Layout Shift - CLS)

- **Kích thước định trước (Reserved Space):** Mọi hình ảnh, video, banner quảng cáo, hoặc dynamic widgets phải khai báo trước tỷ lệ khung hình (`aspect-ratio`, `min-h-[...]`, hoặc width/height tường minh).
- **Tabular Figures cho số liệu động:** Luôn khai báo `tabular-nums` và `font-mono` cho các trường dữ liệu nhảy số liên tục (đồng hồ đếm ngược, tỷ giá, số dư tài chính, phần trăm tải) để ngăn chữ số rung giật ngang.
- **Font Display:** Luôn sử dụng `font-display: swap` hoặc nạp font tối ưu qua framework (`next/font`) để triệt tiêu hiện tượng FOIT (Flash of Invisible Text).

---

## 5. Tiêu Chuẩn Tiếp Cận (Accessibility - WCAG 2.1 AA)

- **Tỷ lệ tương phản màu (Contrast Ratio):** Tối thiểu $4.5:1$ cho văn bản thường và $3:1$ cho văn bản lớn ($\ge 18\text{pt}$ hoặc $\ge 14\text{pt}$ bold) và thành phần UI tương tác.
- **Bàn phím & Focus Ring:** Mọi phần tử tương tác phải truy cập được qua phím Tab; trạng thái focus phải rõ ràng (`focus-visible:ring-2 focus-visible:ring-ring focus-visible:outline-none`).
- **Semantic HTML & ARIA:** Ưu tiên dùng HTML semantic (`<button>`, `<nav>`, `<main>`, `<dialog>`). Chỉ bổ sung `aria-label`, `aria-expanded`, `aria-describedby` khi thẻ gốc không truyền tải đủ ngữ nghĩa cho Screen Readers.
