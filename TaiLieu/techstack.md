# Cấu Hình Công Nghệ (Tech Stack) - Dự Án Web POS Nội Thất

Hệ thống được phát triển theo kiến trúc hiện đại, tối ưu hóa hiệu năng render phía client, đảm bảo tính an toàn dữ liệu và dễ dàng tùy biến giao diện theo tone màu chủ đạo (Beige).

---

## 1. Công Nghệ Cốt Lõi (Core Stack)

* **Framework chính:** `Next.js` (App Router) - Đóng vai trò xử lý toàn diện cả giao diện người dùng (Frontend - React) và các đường dẫn API hệ thống (Backend - API Routes).
* **Giao diện (UI Framework):** `Tailwind CSS` kết hợp với `shadcn/ui`. Bộ thư viện component cao cấp này giúp tối ưu hóa UI/UX, hỗ trợ tùy biến các biến màu hệ thống (CSS Variables) để đồng bộ tone màu Beige chủ đạo một cách mượt mà.
* **Trình quản lý DB (ORM):** `Prisma ORM` - Giúp giao tiếp, truy vấn và quản lý cấu trúc cơ sở dữ liệu (Schema) một cách mạnh mẽ, an toàn và tường minh.
* **Cơ sở dữ liệu (Database):** `MySQL` - Đảm bảo tính ổn định, lưu trữ toàn vẹn dữ liệu hàng hóa, hóa đơn và khách hàng.
* **Xác thực & Phân quyền (Authentication):** `NextAuth.js` - Quản lý các phiên đăng nhập bảo mật và thực hiện phân quyền chặt chẽ cho 3 vai trò: Admin (Chủ cửa hàng), Quản lý, và Nhân viên.

---

## 2. Môi Trường Chạy & Công Cụ Phát Triển

### Môi trường hệ thống
* **Node.js & npm:** (Bắt buộc) Môi trường chạy JavaScript phía server và trình quản lý gói để cài đặt các thư viện phụ thuộc.
* **MySQL Server:** Hệ quản trị cơ sở dữ liệu (Có thể sử dụng giải pháp tích hợp sẵn như **XAMPP** để khởi chạy nhanh dịch vụ MySQL cục bộ).

### Công cụ lập trình (IDE & Extensions)
* **Phần mềm chính:** `Visual Studio Code` (VS Code)
* **Các Extension khuyên dùng để hỗ trợ viết code chuẩn (Clean Code):**
  * **ESLint:** Kiểm tra và phát hiện sớm các lỗi cú pháp, logic JavaScript/TypeScript.
  * **Prettier:** Tự động căn chỉnh, định dạng mã nguồn đồng bộ và đẹp mắt.
  * **Tailwind CSS IntelliSense:** Gợi ý thông minh các class của Tailwind, giúp tăng tốc độ thiết kế giao diện tone Beige.
  * **Prisma:** Extension chính thức hỗ trợ highlight cú pháp, tự động gợi ý (IntelliSense) và định dạng file dữ liệu `schema.prisma`.

### Nâng cao
AI và API hỗ trợ tạo mã QR thanh toán và gửi hóa đơn qua mail khách hàng hoặc xuất hóa đơn PDF.

---