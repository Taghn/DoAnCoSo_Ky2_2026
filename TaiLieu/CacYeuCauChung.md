# HƯỚNG DẪN CHI TIẾT ĐỀ TÀI ĐỒ ÁN: POS CHO CỬA HÀNG NỘI THẤT NHỎ

Tài liệu này định hướng cách phân tích, thiết kế, xây dựng, kiểm thử và demo đề tài POS theo yêu cầu đồ án Công nghệ phần mềm. Sinh viên có thể điều chỉnh tên cửa hàng, dữ liệu mẫu và công nghệ triển khai cho phù hợp với nhóm hoặc cá nhân.

---

## 1. Định hướng và mục tiêu đề tài

Đề tài **“POS cho cửa hàng nội thất nhỏ”** tập trung xây dựng một hệ thống POS hỗ trợ bán hàng tại quầy cho cửa hàng nội thất nhỏ. Hệ thống cần giúp cửa hàng giảm thao tác thủ công, quản lý sản phẩm, hóa đơn, khách hàng, tồn kho và báo cáo doanh thu trong cùng một nền tảng.

* **Xây dựng quy trình bán hàng nhanh:** Tìm sản phẩm, thêm vào giỏ, tính tiền, thanh toán, tạo hóa đơn.
* **Quản lý dữ liệu tập trung:** Quản lý sản phẩm, danh mục, khách hàng, nhân viên và tồn kho.
* **Cung cấp dashboard theo dõi:** Doanh thu, số đơn, sản phẩm bán chạy và hàng tồn kho thấp.
* **Bảo đảm tính an toàn hệ thống:** Có đăng nhập, phân quyền, validation dữ liệu và xử lý lỗi cơ bản.
* **Khả năng mở rộng:** Có thể mở rộng thêm mã vạch, QR, khuyến mãi, khách hàng thân thiết, AI/API hoặc nhiều chi nhánh.

---

## 2. Yêu cầu đánh giá cần đáp ứng

* **Phân tích bài toán rõ:** Nêu bối cảnh cửa hàng, vấn đề hiện tại, mục tiêu hệ thống, actor và phạm vi.
* **Thiết kế hệ thống có kiến trúc rõ ràng:** Kiến trúc MVC hoặc client-server; đầy đủ ERD, Activity Diagram, Use Case Diagram, Sequence Diagram và Class Diagram.
* **Chức năng không dừng ở CRUD:** Cần có luồng bán hàng, tạo hóa đơn, xử lý tồn kho, thanh toán và báo cáo.
* **Giao diện:** Dễ dùng, responsive, tối ưu thao tác nhanh cho màn hình POS.
* **Dữ liệu và bảo mật:** Validation, đăng nhập, phân quyền, mã hóa mật khẩu, kiểm soát truy cập.
* **Dashboard:** Có biểu đồ, số liệu doanh thu, đơn hàng, tồn kho và insight quản trị.
* **Triển khai & demo:** Có triển khai demo, dữ liệu mẫu, tài khoản mẫu và kịch bản bảo vệ rõ ràng.
* **Tính năng nâng cao:** Có điểm nâng cao như QR, mã vạch, khuyến mãi, khách hàng thân thiết, AI/API hoặc báo cáo nâng cao.
* **Chất lượng mã nguồn:** Code phân tầng, đặt tên rõ, có xử lý lỗi và tài liệu cài đặt.

---

## 3. Phân tích bài toán

Cửa hàng nội thất nhỏ thường có nhiều sản phẩm, nhiều mức giá, nhiều biến thể và nhu cầu bán nhanh tại quầy. Nếu quản lý bằng sổ sách hoặc Excel, cửa hàng dễ sai lệch tồn kho, khó tra cứu hóa đơn và khó đánh giá hiệu quả kinh doanh.

### 3.1. Vấn đề thực tế
* Sản phẩm có kích thước, màu sắc, chất liệu và có thể cần giao/lắp đặt.
* Tồn kho sản phẩm lớn khó kiểm soát nếu không lưu vị trí và trạng thái.
* Khách thường cần tư vấn theo không gian và ngân sách.

### 3.2. Phạm vi đề tài
* Quản lý nội thất nhỏ, phụ kiện trang trí, tồn kho, hóa đơn và khách hàng.
* Hỗ trợ kích thước, chất liệu, màu sắc, ghi chú vận chuyển.
* Không bắt buộc thiết kế 3D nội thất.

---

## 4. Actor và phân quyền

| Actor | Quyền và nhiệm vụ chính |
| :--- | :--- |
| **Admin / Chủ cửa hàng** | Quản lý toàn hệ thống, tài khoản, phân quyền, cấu hình, báo cáo tổng hợp. |
| **Quản lý cửa hàng** | Quản lý sản phẩm, tồn kho, hóa đơn, khách hàng, khuyến mãi và báo cáo vận hành. |
| **Nhân viên bán hàng** | Tạo hóa đơn, tìm sản phẩm, nhận thanh toán, xử lý đổi trả cơ bản, xem lịch sử bán hàng trong ca. |
| **Khách hàng** | Cung cấp thông tin khi mua, nhận hóa đơn, tích điểm hoặc đặt hàng online (nếu hệ thống có chức năng này). |

---

## 5. Chức năng hệ thống

### 5.1. Chức năng cốt lõi
* **Tài khoản:** Đăng nhập, đăng xuất, đổi mật khẩu và phân quyền theo vai trò.
* **Quản lý danh mục:** Thêm, sửa, xóa/ẩn, tìm kiếm, thống kê số lượng sản phẩm theo danh mục.
* **Quản lý sản phẩm:** Mã SKU, tên, hình ảnh, giá nhập, giá bán, số lượng tồn, trạng thái kinh doanh.
* **Bán hàng tại quầy:** Tìm kiếm/quét mã, thêm giỏ hàng, tính tiền, áp dụng giảm giá, thanh toán.
* **Quản lý hóa đơn:** Xem chi tiết, lọc theo ngày/nhân viên/khách hàng, hủy hóa đơn theo quyền.
* **Quản lý kho:** Nhập hàng, xuất hàng do bán, điều chỉnh tồn kho, lưu lịch sử giao dịch kho.
* **Quản lý khách hàng:** Thông tin liên hệ, lịch sử mua hàng, điểm tích lũy nếu có.
* **Dashboard và báo cáo:** Doanh thu, số đơn, sản phẩm bán chạy, tồn kho thấp, hiệu suất nhân viên.

### 5.2. Chức năng riêng của đề tài
* Quản lý sản phẩm theo kích thước, chất liệu, màu sắc.
* Tạo đơn bán có ghi chú giao/lắp đặt.
* Theo dõi sản phẩm trưng bày và sản phẩm trong kho.
* Gợi ý sản phẩm theo không gian: Phòng khách, phòng ngủ, góc làm việc.

---

## 6. Thiết kế dữ liệu và ERD

Cơ sở dữ liệu cần chuẩn hóa để tránh trùng lặp, có khóa chính, khóa ngoại và ràng buộc dữ liệu. ERD phải thể hiện quan hệ giữa sản phẩm, danh mục, hóa đơn, chi tiết hóa đơn, khách hàng, nhân viên, thanh toán và kho.

### 6.1. Thuộc tính sản phẩm nên có
| Thuộc tính | Mục đích |
| :--- | :--- |
| **Mã sản phẩm / SKU** | Duy nhất, dùng tìm kiếm, quét mã hoặc in tem. |
| **Tên sản phẩm** | Tên hiển thị trên POS và hóa đơn. |
| **Danh mục** | Phân loại sản phẩm theo nhóm. |
| **Giá nhập, giá bán** | Phục vụ tính lợi nhuận và báo cáo. |
| **Số lượng tồn** | Dùng kiểm soát bán hàng và cảnh báo hết hàng. |
| **Trạng thái** | Đang bán, tạm ngưng, hết hàng. |
| **Kích thước** | Dài, rộng, cao để khách kiểm tra phù hợp không gian. |
| **Chất liệu / Màu sắc** | Gỗ, kim loại, nhựa, vải, màu sắc. |
| **Vị trí kho** | Kệ/kho để lấy hàng nhanh. |
| **Ghi chú vận chuyển** | Cồng kềnh, cần lắp đặt, giao sau. |

### 6.2. Các bảng dữ liệu đề xuất
* `users`: Tài khoản nhân viên, quản lý, admin, khách hàng (nếu có online order).
* `roles`: Vai trò và nhóm quyền.
* `categories`: Danh mục sản phẩm/dịch vụ.
* `products`: Thông tin sản phẩm, giá, tồn kho, trạng thái.
* `customers`: Thông tin khách hàng và lịch sử mua hàng.
* `orders`: Hóa đơn bán tại quầy hoặc đơn online.
* `order_details`: Chi tiết sản phẩm trong từng hóa đơn.
* `payments`: Phương thức và trạng thái thanh toán.
* `stock_transactions`: Lịch sử nhập, xuất, điều chỉnh tồn kho.
* `promotions`: Chương trình khuyến mãi, mã giảm giá nếu có.
* `audit_logs`: Nhật ký thao tác quan trọng (nếu làm nâng cao).

### 6.3. Quan hệ chính trong ERD
* Một danh mục có nhiều sản phẩm; một sản phẩm thuộc một danh mục.
* Một hóa đơn có nhiều chi tiết hóa đơn; một chi tiết hóa đơn tham chiếu một sản phẩm.
* Một khách hàng có nhiều hóa đơn; một nhân viên có thể lập nhiều hóa đơn.
* Một sản phẩm có nhiều giao dịch kho để theo dõi nhập, xuất, điều chỉnh.
* Một hóa đơn có một hoặc nhiều bản ghi thanh toán nếu hỗ trợ thanh toán kết hợp.

---

## 7. UML và thiết kế giao diện

### 7.1. UML cần có
* **Use Case Diagram:** Đăng nhập, quản lý sản phẩm, bán hàng, thanh toán, quản lý kho, báo cáo.
* **Activity Diagram:** Quy trình bán hàng tại quầy, nhập hàng, xử lý thanh toán, cập nhật tồn kho.
* **Sequence Diagram:** Nhân viên tạo hóa đơn, hệ thống kiểm tra tồn kho, thanh toán và lưu hóa đơn.
* **Class Diagram:** User, Role, Product, Category, Customer, Order, OrderDetail, Payment, StockTransaction.

### 7.2. Màn hình giao diện nên có
* Màn hình đăng nhập và trang tổng quan sau đăng nhập.
* Màn hình POS bán hàng: Danh sách sản phẩm, tìm kiếm, giỏ hàng, tổng tiền, thanh toán.
* Màn hình quản lý sản phẩm, danh mục, khách hàng, nhân viên và tồn kho.
* Màn hình quản lý hóa đơn và chi tiết hóa đơn.
* Màn hình dashboard báo cáo có biểu đồ, bộ lọc ngày và bảng thống kê.
* Giao diện responsive, dễ thao tác bằng chuột hoặc máy quét mã.

### 7.3. Dashboard đề xuất
* Doanh thu theo nhóm nội thất.
* Sản phẩm trưng bày bán chạy.
* Đơn cần giao/lắp đặt.
* Sản phẩm tồn kho cồng kềnh.

---

## 8. Bảo mật, kiểm thử và demo

### 8.1. Bảo mật và kiểm soát dữ liệu
* Mật khẩu được băm bằng `bcrypt`/`argon2` hoặc cơ chế tương đương.
* Kiểm tra quyền truy cập theo vai trò; không cho người dùng truy cập URL ngoài quyền.
* Validation phía client và server cho giá, số lượng, email, số điện thoại, mã sản phẩm.
* Không cho bán vượt tồn kho; không cho nhập giá âm hoặc số lượng âm.
* Sử dụng ORM/prepared statement để hạn chế lỗi SQL Injection.
* Ghi log các thao tác quan trọng: Tạo hóa đơn, hủy hóa đơn, điều chỉnh tồn kho.

### 8.2. Test case cần có
* Tạo sản phẩm có kích thước và chất liệu.
* Tạo hóa đơn có ghi chú giao hàng.
* Lọc nội thất theo không gian sử dụng.
* Kiểm tra tồn kho sau khi bán.

### 8.3. Kịch bản demo bảo vệ
* Thêm bàn nhỏ, ghế, kệ trang trí.
* Bán sản phẩm có ghi chú vận chuyển.
* Xem đơn cần giao/lắp.
* Xem báo cáo doanh thu theo nhóm nội thất.

---

## 9. Chức năng nâng cao và hướng phát triển

* AI gợi ý nội thất theo không gian và ngân sách.
* QR xem ảnh phối cảnh sản phẩm.
* API gửi báo giá PDF cho khách.

---

## 10. Cấu trúc báo cáo đề xuất

* **Chương 1:** Tổng quan đề tài, lý do chọn đề tài, mục tiêu, phạm vi và ý nghĩa thực tiễn.
* **Chương 2:** Cơ sở lý thuyết, tổng quan POS, công nghệ sử dụng, mô hình kiến trúc.
* **Chương 3:** Phân tích hệ thống, actor, yêu cầu chức năng/phi chức năng, Use Case chi tiết.
* **Chương 4:** Thiết kế hệ thống, ERD, UML, thiết kế giao diện, thiết kế API nếu có.
* **Chương 5:** Xây dựng và triển khai, mô tả chức năng, hình ảnh màn hình, kiểm thấu.
* **Chương 6:** Kết quả đạt được, hạn chế và hướng phát triển.
* **Phụ lục:** Hướng dẫn cài đặt, tài khoản demo, script database, dữ liệu mẫu.