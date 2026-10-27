# 📊 Tổng Hợp Hệ Thống POS — Cửa Hàng Nội Thất Toàn Thắng
> **Stack:** Next.js 14 (App Router) + TypeScript + Prisma ORM + MySQL + Socket.io + NextAuth

---

## 🗄️ Kiến Trúc Tổng Quan

```
next-pos/src/app/
├── (pos)/                  ← Route Group chính (yêu cầu đăng nhập - NextAuth)
│   ├── POSClient.tsx       ← Giao diện bán hàng (1311 dòng)
│   ├── POSSidebar.tsx      ← Thanh điều hướng (200 dòng)
│   ├── page.tsx            ← Server Component → POS chính (/)
│   ├── orders/             ← Tra cứu Đơn hàng (/orders)
│   ├── customers/          ← Quản lý Khách hàng (/customers)
│   ├── products/           ← Quản lý Sản phẩm (/products)
│   ├── categories/         ← Quản lý Danh mục (/categories)
│   ├── staff/              ← Quản lý Nhân viên (/staff)
│   ├── reports/            ← Báo cáo Thống kê (/reports)
│   ├── revenue/            ← Báo cáo Doanh thu (/revenue)
│   ├── inventory/          ← Quản lý Tồn kho (/inventory)
│   └── logs/               ← Nhật ký Hoạt động (/logs)
├── login/                  ← Trang đăng nhập
└── api/                    ← REST API (Next.js Route Handlers)
```

### 📐 Database Schema (Prisma + MySQL)

| Model | Trường quan trọng | Ghi chú |
|-------|------------------|---------|
| `User` | name, email, password, role, isActive | 3 role: ADMIN/MANAGER/STAFF |
| `Category` | name | Danh mục sản phẩm |
| `Product` | sku, name, price, importPrice, stock, image, colors, brand, description | `colors` dạng text: `"Nâu:10,Trắng:5"` |
| `Customer` | name, phone, email, isVip, spent | `spent` = tổng chi tiêu tích lũy |
| `Order` | id (CUID), customerId, customerName, total, method, note, createdBy | method: "Tiền mặt"/"Chuyển khoản" |
| `OrderItem` | productId, quantity, price, productName | Lưu tên SP tại thời điểm mua |
| `ProductLog` | productName, action, quantity, stockBefore, stockAfter, createdBy | Log nhập/xuất kho |
| `ActivityLog` | user, role, action, target, details | Log toàn bộ hoạt động |
| `ScanSession` | token, isActive, expiresAt | Phiên kết nối máy quét QR |
| `ScannedItem` | sessionId, productSku, color | SP quét qua mobile |

---

## 🔐 Phân Quyền Theo Role

```tsx
// POSSidebar.tsx — điều hướng hiện/ẩn theo role
{role === "ADMIN" && <NavItem label="Nhân viên" href="/staff" />}
{role === "ADMIN" && <NavItem label="Nhật ký Hoạt động" href="/logs" />}
{(role === "ADMIN" || role === "MANAGER") && <NavItem label="Báo cáo Thống kê" href="/reports" />}
{(role === "ADMIN" || role === "MANAGER") && <NavItem label="Báo cáo Doanh thu" href="/revenue" />}
{(role === "ADMIN" || role === "MANAGER") && <NavItem label="Tồn kho" href="/inventory" />}
{(role === "ADMIN" || role === "MANAGER") && <NavItem label="Sản phẩm" href="/products" />}
{(role === "ADMIN" || role === "MANAGER") && <NavItem label="Danh mục" href="/categories" />}
// Còn lại (POS, Đơn hàng, Khách hàng): tất cả role đều xem được
```

| Tab | ADMIN | MANAGER | STAFF |
|-----|:-----:|:-------:|:-----:|
| Bán hàng POS | ✅ | ✅ | ✅ |
| Tra cứu Đơn hàng | ✅ | ✅ | ✅ |
| Khách hàng | ✅ | ✅ | ✅ |
| Sản phẩm | ✅ | ✅ | ❌ |
| Danh mục | ✅ | ✅ | ❌ |
| Báo cáo Thống kê | ✅ | ✅ | ❌ |
| Báo cáo Doanh thu | ✅ | ✅ | ❌ |
| Tồn kho | ✅ | ✅ | ❌ |
| Nhân viên | ✅ | ❌ | ❌ |
| Nhật ký Hoạt động | ✅ | ❌ | ❌ |

---

## 📌 TAB 1 — Bán Hàng POS (`/`) • `POSClient.tsx` (1311 dòng)

### 🎯 Nhiệm vụ
Giao diện bán hàng chính tại quầy — hiển thị sản phẩm, quản lý giỏ hàng, thanh toán, in hóa đơn, gửi email, quét mã QR bằng điện thoại, tư vấn AI.

### ⚙️ Cách hoạt động
1. Load sản phẩm/khách hàng từ server (SSR `page.tsx`) → truyền qua props
2. Nhân viên lọc theo danh mục, thương hiệu, tìm kiếm (hỗ trợ không dấu)
3. Click sản phẩm → **Modal chọn màu** → `addToCart()`
4. Mở **Giỏ hàng** (slide-in drawer) → tìm/tạo khách hàng, áp voucher, thêm ghi chú
5. Bấm **Thanh toán** → chọn Tiền mặt/Chuyển khoản (QR VietQR) → `handleCheckoutConfirm()`
6. Kết quả: hiển thị **Hóa đơn** → In (`window.print()`) hoặc gửi Email

### 📊 State Management
| State | Kiểu | Mô tả |
|-------|------|-------|
| `products` | `Product[]` | Danh sách sản phẩm (đã lọc isActive) |
| `cart` | `CartItem[]` | Giỏ hàng `{ product, quantity, selectedColor }` |
| `cartPhone` | `string` | SĐT khách hàng đang chọn |
| `appliedVoucher` | `{code, percent}` | Voucher đang áp dụng |
| `paymentMethod` | `"cash" \| "bank"` | Phương thức thanh toán |
| `receiptData` | `any` | Dữ liệu hóa đơn sau khi thanh toán thành công |
| `scannerSession` | `string` | Token phiên quét QR (ví dụ: `"POS_abc123"`) |
| `printMode` | `"receipt" \| "single" \| "bulk"` | Chế độ in |

### 💻 Vibecode chính

#### 1. Tìm kiếm không dấu tiếng Việt
```tsx
// POSClient.tsx:22-35
const removeVietnameseTones = (str: string): string => {
  return str.normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
    .replace(/đ/g, "d").replace(/Đ/g, "d").toLowerCase();
};

// Lọc theo từng token (tìm theo tên, SKU, danh mục, thương hiệu)
const filteredProducts = products.filter((p) => {
  if (!p.isActive) return false;
  const tokens = removeVietnameseTones(searchQuery).split(/\s+/).filter(Boolean);
  return tokens.every((t) =>
    removeVietnameseTones(p.name).includes(t) ||
    removeVietnameseTones(p.sku || "").includes(t)
  );
});
```

#### 2. Tính tiền (VIP → Voucher)
```tsx
// POSClient.tsx:310-313
const totalAmount    = cart.reduce((s, i) => s + i.product.price * i.quantity, 0);
const vipDiscount    = isCartVip ? Math.round(totalAmount * 0.1) : 0;  // -10% VIP
const voucherDiscount = appliedVoucher
  ? Math.round((totalAmount - vipDiscount) * appliedVoucher.percent / 100) : 0;
const finalAmount    = totalAmount - vipDiscount - voucherDiscount;
// ⚠️ Voucher tính trên giá SAU khi giảm VIP
```

#### 3. Thanh toán — gọi API + cập nhật stock ngay
```tsx
// POSClient.tsx:388-437
const handleCheckoutConfirm = async () => {
  const payload = {
    customerPhone, customerName, total: finalAmount, method,
    items: cart.map(i => ({ productId: i.product.id, quantity: i.quantity,
                            price: i.product.price, color: i.selectedColor })),
    note: orderNote.trim() || null
  };
  const res  = await fetch("/api/orders", { method: "POST", body: JSON.stringify(payload) });
  const data = await res.json();
  // Server trả về updatedProducts → cập nhật state tức thì
  if (Array.isArray(data.updatedProducts))
    setProducts(data.updatedProducts.filter(p => p.isActive));
  setReceiptData({ id, customer, date, method, items, subtotal, vipDiscount, ... });
};
```

#### 4. Gửi hóa đơn qua email
```tsx
// POSClient.tsx:444-473
const handleSendEmail = async () => {
  const res = await fetch("/api/orders/email", {
    method: "POST",
    body: JSON.stringify({ email, orderId, customerName, total, items, ... })
  });
};
```

#### 5. Voucher mã cứng
```tsx
// POSClient.tsx:39
const VALID_VOUCHERS: Record<string, number> = {
  "SALE10": 10, "SALE20": 20, "TEST0102": 99.9
};
```

#### 6. QR Thanh toán VietQR (không cần API ngân hàng)
```tsx
// POSClient.tsx:314
const qrUrl = `https://img.vietqr.io/image/sacombank-0352254951-compact2.png
  ?amount=${finalAmount}
  &addInfo=Thanh%20toan%20noi%20that%20Toan%20Thang
  &accountName=Nguyen%20Huu%20Thang`;
// → Khách hàng quét → ngân hàng tự điền thông tin
```

#### 7. In hóa đơn với CSS Media Print (3 chế độ)
```tsx
// POSClient.tsx:1307 — CSS @media print
// 3 vùng ẩn trong DOM: #receipt-print-wrapper, #single-qr-print-area, #bulk-qr-print-area
// Khi gọi window.print(), chỉ vùng theo printMode được visible
const triggerPrint = (mode: "receipt" | "single" | "bulk") => {
  setPrintMode(mode);
  setTimeout(() => window.print(), 150);
};
```

#### 8. Socket.io — Kết nối máy quét QR bằng điện thoại
```tsx
// POSClient.tsx:182-284
// Tạo session token
const currentSession = "POS_" + Math.random().toString(36).substr(2, 9);
setScannerUrl(`${tunnelUrl}?session=${currentSession}`);

// Kết nối Socket.io (qua Cloudflare Tunnel nếu không phải localhost)
socketInstance = io(socketUrl, { reconnectionAttempts: 10, reconnectionDelay: 2000 });
socketInstance.emit("pos-join", scannerSession); // Tham gia phòng

// Nghe sự kiện
socketInstance.on("barcode-scanned", (barcode) => {
  // Yêu cầu điện thoại chọn màu
  socketRef.current?.emit('require-color-selection', {
    sessionToken, product: { name, colors }
  });
});
socketInstance.on("barcode-scanned-with-color", ({ barcode, color }) => {
  addToCart(product, color); // Thêm vào giỏ với màu đã chọn
});
```

---

## 📌 TAB 2 — Tra Cứu Đơn Hàng (`/orders`) • `OrdersClient.tsx` (322 dòng)

### 🎯 Nhiệm vụ
Tra cứu, xem chi tiết và in lại hóa đơn các đơn hàng đã tạo.

### ⚙️ Cách hoạt động
- Dữ liệu load từ SSR (không fetch ở client)
- Lọc theo: thời gian (Hôm nay / Chọn ngày), phương thức thanh toán, từ khóa
- Click đơn hàng → Modal chi tiết → In lại hóa đơn
- Hỗ trợ URL param `?id=xxx` → tự mở modal đúng đơn hàng

### 📊 State chính
| State | Kiểu | Mô tả |
|-------|------|-------|
| `orders` | `Order[]` | Danh sách đơn hàng |
| `orderTimeFilter` | `"all" \| "today" \| "custom_date"` | Mặc định `"today"` |
| `paymentFilter` | `"all" \| "Tiền mặt" \| "Chuyển khoản"` | |
| `viewingOrder` | `Order \| null` | Đơn hàng đang xem |

### 💻 Vibecode chính

#### Lọc đơn hàng
```tsx
// OrdersClient.tsx:75-102
const filteredOrders = orders.filter((o) => {
  const matchSearch = o.id.toLowerCase().includes(orderSearchQuery.toLowerCase())
    || o.customer.toLowerCase().includes(orderSearchQuery.toLowerCase());
  if (paymentFilter !== "all" && o.method !== paymentFilter) return false;
  if (orderTimeFilter === "today") {
    const diffDays = Math.floor((today - parseDateVi(o.date)) / (1000*60*60*24));
    return diffDays === 0;
  }
  return true;
});
```

#### Deep-link: mở đơn hàng qua URL
```tsx
// OrdersClient.tsx:59-73
useEffect(() => {
  const idParam = new URLSearchParams(window.location.search).get("id");
  if (idParam) {
    setOrderSearchQuery(idParam);
    setOrderTimeFilter("all"); // Bỏ filter thời gian để tìm được
    const targetOrder = initialOrders.find((o) => o.id === idParam);
    if (targetOrder) setViewingOrder(targetOrder); // Mở modal ngay
  }
}, [initialOrders]);
```

---

## 📌 TAB 3 — Khách Hàng (`/customers`) • `CustomersClient.tsx` (257 dòng)

### 🎯 Nhiệm vụ
Quản lý danh sách thành viên, xem lịch sử chi tiêu, phân loại VIP.

### 📊 State chính
| State | Kiểu | Mô tả |
|-------|------|-------|
| `customerFilter` | `"all" \| "vip" \| "regular"` | Lọc loại khách |
| `editingCustomer` | `Customer \| null` | Khách đang chỉnh sửa |

### 💻 Vibecode chính — CRUD qua API
```tsx
// POST: Thêm khách mới
fetch("/api/customers", { method: "POST", body: JSON.stringify({ name, phone, email }) });

// PUT: Sửa thông tin
fetch("/api/customers", { method: "PUT", body: JSON.stringify(editingCustomer) });

// DELETE: Xóa
fetch(`/api/customers?id=${customerId}`, { method: "DELETE" });
```

#### Lọc VIP / Thường
```tsx
const filteredCustomers = customers.filter((c) => {
  const matchSearch = c.name.toLowerCase().includes(query) || c.phone.includes(query);
  if (customerFilter === "vip") return matchSearch && c.isVip;
  if (customerFilter === "regular") return matchSearch && !c.isVip;
  return matchSearch;
});
```

**JSX chính:** Bảng 6 cột — Tên+Email, SĐT, badge VIP/Thành viên, Tổng chi tiêu (`spent`), nút Sửa/Xóa.

---

## 📌 TAB 4 — Báo Cáo Thống Kê (`/reports`) • `ReportsClient.tsx` (579 dòng)

### 🎯 Nhiệm vụ
Phân tích kinh doanh: doanh thu, số đơn, biểu đồ xu hướng, sản phẩm bán chạy, cảnh báo tồn kho thấp.

### ⚙️ Cách hoạt động
- **Không có fetch API** — toàn bộ tính toán từ dữ liệu SSR
- Lọc theo khoảng thời gian (7 ngày, tháng này, tháng trước, năm, ngày cụ thể)
- Biểu đồ SVG tự vẽ bằng CSS/JS (không dùng thư viện chart)
- Click card "Tồn kho" → `router.push("/inventory?stock=low")` (deep-link)

### 📊 KPI tính toán
```tsx
// ReportsClient.tsx:89-93
const totalRevenue  = filteredReportOrders.reduce((s, o) => s + o.total, 0);
const totalOrders   = filteredReportOrders.length;
const aov           = totalOrders > 0 ? Math.round(totalRevenue / totalOrders) : 0;
const grossProfit   = Math.round(totalRevenue * 0.45); // Ước tính margin 45%
const lowStockCount = products.filter((p) => p.stock < 5).length;
```

### 💻 Vibecode chính
```tsx
// Lọc theo thời gian
const filteredReportOrders = orders.filter((o) => {
  if (reportTimeFilter === "7days") {
    const diffDays = Math.floor((today - parseDateVi(o.date)) / (1000*60*60*24));
    return diffDays >= 0 && diffDays < 7;
  }
  if (reportTimeFilter === "this_month")
    return oDate.getMonth() === now.getMonth() && oDate.getFullYear() === now.getFullYear();
  // last_month, this_year, custom_date...
});

// Navigate với deep-link
router.push("/inventory?stock=low");   // → Tab Tồn kho tự filter
router.push("/orders?id=${o.id}");     // → Tab Đơn hàng mở đúng đơn
```

**JSX chính:** KPI cards → SVG line chart xu hướng (tooltip hover) → SVG donut chart theo danh mục → Top 5 sản phẩm bán chạy → Bảng đơn hàng trong kỳ.

---

## 📌 TAB 5 — Báo Cáo Doanh Thu (`/revenue`) • `RevenueClient.tsx` (495 dòng)

### 🎯 Nhiệm vụ
Phân tích doanh thu sâu hơn: so sánh tăng trưởng kỳ trước, phân tích theo phương thức, loại khách, xếp hạng nhân viên.

### 📊 Các chỉ số đặc biệt
```tsx
// RevenueClient.tsx
const revenueGrowth = ((currentRevenue - prevRevenue) / prevRevenue) * 100; // % tăng trưởng
const cashPercent   = Math.round((cashRevenue / totalRevenue) * 100);       // % tiền mặt
const retailRevenue = filteredOrders                                          // Khách vãng lai
  .filter(o => !o.isVip && !o.customerId).reduce(...);
```

**JSX chính:** 4 KPI cards (Tổng DT + % tăng trưởng, AOV, Lợi nhuận gộp, KH) → SVG bar chart doanh thu với gradient → SVG donut chart phương thức TT → Bảng xếp hạng nhân viên (badge vàng/bạc/đồng).

---

## 📌 TAB 6 — Tồn Kho (`/inventory`) • `InventoryClient.tsx` (394 dòng)

### 🎯 Nhiệm vụ
Theo dõi và nhập hàng, xem lịch sử biến động tồn kho.

### ⚙️ Cách hoạt động
1. Hiển thị danh sách SP với badge trạng thái: 🔴 Hết hàng / 🟡 Cần nhập (< 5) / 🟢 Còn hàng
2. Lọc theo danh mục, thương hiệu, trạng thái tồn kho
3. Nút **Nhập kho** → Modal chọn màu + số lượng → gọi `PUT /api/products`
4. Xem **Lịch sử** nhập/xuất (`GET /api/products/history`)
5. URL param `?stock=low` → auto bật filter "Cần nhập"

### 💻 Vibecode chính

#### Nhập kho
```tsx
// InventoryClient.tsx:74-onwards
const handleImportStock = async () => {
  const newStock = importingProduct.stock + qty;
  await fetch("/api/products", {
    method: "PUT",
    body: JSON.stringify({
      id: importingProduct.id,
      stock: newStock,
      importColor: importColor || null,
      importQty: qty,
      action: "Nhập hàng"  // → server ghi vào ProductLog
    })
  });
};
```

#### Auto-filter từ URL
```tsx
const searchParams = useSearchParams();
useEffect(() => {
  if (searchParams.get("stock") === "low") setStockFilter("Cần nhập");
}, [searchParams]);
```

**JSX chính:** Bảng 8 cột — ảnh, tên+SKU+danh mục, thương hiệu, tổng tồn, badge trạng thái, giá nhập, tổng giá tồn, nút Nhập kho.

---

## 📌 TAB 7 — Sản Phẩm (`/products`) • `ProductsClient.tsx` (424 dòng)

### 🎯 Nhiệm vụ
CRUD sản phẩm, upload ảnh, quản lý màu sắc theo từng màu, xem lịch sử thao tác.

### 📊 State chính
| State | Mô tả |
|-------|-------|
| `editingProduct` | Sản phẩm đang sửa (`null` = đang thêm mới) |
| `productForm` | Form data: sku, name, price, importPrice, stock, category, image, colors, brand, description |
| `isUploadingImage` | Loading khi upload ảnh |
| `viewingStockProduct` | Xem chi tiết tồn kho theo màu |
| `highlightId` | Highlight row khi đến từ deep-link |

### 💻 Vibecode chính
```tsx
// Upload ảnh
const handleImageUpload = async (e) => {
  const formData = new FormData();
  formData.append("file", e.target.files[0]);
  const res = await fetch("/api/upload", { method: "POST", body: formData });
  const data = await res.json();
  setProductForm(f => ({ ...f, image: data.url }));
};

// Highlight từ deep-link (?id=xxx)
useEffect(() => {
  const idParam = new URLSearchParams(window.location.search).get("id");
  if (idParam) {
    setHighlightId(Number(idParam));
    // Scroll đến row + ẩn highlight sau 3.5 giây
    setTimeout(() => setHighlightId(null), 3500);
  }
}, []);
```

---

## 📌 TAB 8 — Danh Mục (`/categories`) • `CategoriesClient.tsx` (242 dòng)

### 🎯 Nhiệm vụ
CRUD danh mục sản phẩm, xem sản phẩm trong từng danh mục.

### 💻 Vibecode chính
```tsx
// Xóa danh mục → sản phẩm chuyển về "Chưa có danh mục"
fetch(`/api/categories?name=${catName}`, { method: "DELETE" });

// Sửa tên danh mục → phải gửi cả tên cũ
fetch("/api/categories", {
  method: "PUT",
  body: JSON.stringify({ oldName: editingCategory, name: newName })
});
```

**JSX chính:** Grid card mỗi danh mục (Folder icon, tên, số lượng SP, nút Sửa/Xóa). Card đặc biệt "Chưa có danh mục" (dashed border). Click card → Modal xem danh sách SP → Click SP → `router.push("/products?id=${p.id}")`.

---

## 📌 TAB 9 — Nhân Viên (`/staff`) • `StaffClient.tsx` (227 dòng) [ADMIN only]

### 🎯 Nhiệm vụ
CRUD tài khoản nhân viên, phân quyền role, kích hoạt/tạm dừng tài khoản.

### 💻 Vibecode chính
```tsx
// Toggle active/inactive
fetch("/api/staff", {
  method: "PUT",
  body: JSON.stringify({ id: staff.id, isActive: !staff.isActive })
});

// Validate: không cho xóa/thay đổi role ADMIN duy nhất
if (staffForm.role === "ADMIN" && editingStaff?.role === "ADMIN") {
  alert("Không thể thay đổi role của ADMIN!");
}
```

**JSX chính:** Bảng 5 cột — Tên, Email, badge Role (ADMIN=tím/MANAGER=xanh/STAFF=xám), badge Trạng thái, nút Tạm dừng/Kích hoạt/Sửa/Xóa.

---

## 📌 TAB 10 — Nhật Ký Hoạt Động (`/logs`) • `LogsClient.tsx` (179 dòng) [ADMIN only]

### 🎯 Nhiệm vụ
Xem toàn bộ lịch sử thao tác của nhân viên trong hệ thống.

### ⚙️ Cách hoạt động
- Load từ SSR, có nút **Tải lại** để fetch mới
- Lọc theo đối tượng (Sản phẩm, Khách hàng, Đơn hàng...) và loại thao tác
- Format thời gian thông minh: "Hôm nay lúc HH:mm" / "Hôm qua lúc HH:mm"

### 💻 Vibecode chính
```tsx
// fetchSystemLogs — useCallback để tránh re-render
const fetchSystemLogs = useCallback(async () => {
  setIsLoading(true);
  const res = await fetch("/api/logs");
  const data = await res.json();
  setSystemLogs(Array.isArray(data) ? data : []);
  setIsLoading(false);
}, []);

// Format thời gian thân thiện
const formatLogDate = (dateStr: string) => {
  const diffDays = Math.floor((now - date) / (1000*60*60*24));
  if (diffDays === 0) return `Hôm nay lúc ${hours}:${mins}`;
  if (diffDays === 1) return `Hôm qua lúc ${hours}:${mins}`;
  return `DD/MM/YYYY lúc HH:mm`;
};
```

**JSX chính:** Bảng 6 cột — Thời gian, Tài khoản, badge Vai trò, badge Thao tác, badge Đối tượng, Chi tiết.

---

## 🤖 Tính Năng AI Tư Vấn (Popup trong POS) • `AIConsultation/index.tsx` (333 dòng)

### 🎯 Nhiệm vụ
Nhân viên nhập yêu cầu của khách → AI gợi ý sản phẩm phù hợp kèm lý do.

### ⚙️ Cách hoạt động
1. Nhân viên nhập mô tả yêu cầu khách (ví dụ: "bàn trà gỗ sồi cho căn hộ nhỏ")
2. Gửi `POST /api/ai` với `{ requirements, products }` (toàn bộ catalog)
3. AI trả về `[{ id: number, reason: string, selectedColor?: string }]`
4. Ghép với sản phẩm thực → hiển thị card với lý do, chọn màu, thêm giỏ

### 💻 Vibecode chính
```tsx
// handleConsult
const response = await fetch("/api/ai", {
  method: "POST",
  body: JSON.stringify({ requirements, products }) // Gửi cả catalog
});
const data = await response.json();
// data = [{ id, reason, selectedColor }]
setRecommendations(data);

// Ghép kết quả AI với dữ liệu thực
const recommendedProducts = recommendations
  .map(rec => ({ ...products.find(p => p.id === rec.id), reason: rec.reason }))
  .filter(Boolean);
```

---

## 📡 Kết Nối Máy Quét QR Bằng Điện Thoại (Socket.io + Cloudflare Tunnel)

### Luồng hoạt động
```
POS tạo session token "POS_abc123"
  → Lấy Cloudflare Tunnel URL từ /api/tunnel
  → Sinh QR Code chứa {tunnelUrl}?session=POS_abc123
  → Nhân viên dùng điện thoại quét QR
  → Điện thoại mở trình duyệt → camera quét mã sản phẩm
  → Socket.io: điện thoại emit "barcode-scanned" → POS nhận
  → POS emit "require-color-selection" → điện thoại hiện modal chọn màu
  → Điện thoại chọn màu → emit "barcode-scanned-with-color"
  → POS addToCart(product, color)
```

### Lý do dùng Cloudflare Tunnel
- Điện thoại có thể ở mạng 4G khác → không kết nối được `localhost:3001`
- Tunnel tạo URL public `https://xxx.trycloudflare.com` → điện thoại kết nối được
- Tránh lỗi **Mixed Content** (HTTPS page → HTTP socket)

---

## 🌐 API Routes Chính

| Route | Methods | Chức năng |
|-------|---------|-----------|
| `/api/orders` | POST | Tạo đơn, trừ stock, ghi ActivityLog |
| `/api/orders/email` | POST | Gửi hóa đơn qua Nodemailer |
| `/api/customers` | GET/POST/PUT/DELETE | CRUD khách hàng |
| `/api/products` | GET/POST/PUT/DELETE | CRUD sản phẩm + nhập kho → ghi ProductLog |
| `/api/products/history` | GET | Lịch sử biến động tồn kho |
| `/api/categories` | GET/POST/PUT/DELETE | CRUD danh mục |
| `/api/staff` | GET/POST/PUT/DELETE | CRUD nhân viên |
| `/api/ai` | POST | Tư vấn sản phẩm bằng Gemini AI |
| `/api/tunnel` | GET | Lấy Cloudflare Tunnel URL |
| `/api/auth/[...nextauth]` | — | NextAuth đăng nhập/phiên |
| `/api/logs` | GET | Lấy ActivityLog |
| `/api/upload` | POST | Upload ảnh sản phẩm |

---

## 💡 Điểm Quan Trọng Cần Nhớ Cho Vấn Đáp

### 1. Kiến trúc SSR + Client
```
page.tsx (Server Component)    ← Không có "use client"
  → fetch từ Prisma (DB)       ← Chạy trên server
  → truyền props
  → POSClient.tsx (Client)     ← Có "use client", dùng useState/useEffect
```
**Lý do:** SSR giảm loading time, dữ liệu đã có sẵn khi page render lần đầu.

### 2. Format màu sắc dạng text
```
"Nâu:10,Trắng:5,Đen:0"
→ parseProductColors() → [{ name: "Nâu", qty: 10 }, { name: "Trắng", qty: 5 }, ...]
```
→ `qty <= 0` → nút màu bị disabled (`cursor-not-allowed`)

### 3. Thứ tự giảm giá
```
finalAmount = totalAmount - vipDiscount - voucherDiscount
            (tạm tính)  (-10% nếu VIP) (% voucher áp vào giá sau VIP)
```

### 4. Deep-link giữa các trang
| Từ trang | Deep-link sang | Tham số |
|----------|----------------|---------|
| Reports | Inventory | `?stock=low` → auto filter "Cần nhập" |
| Reports / Inventory | Orders | `?id=xxx` → auto mở modal đơn hàng |
| Categories | Products | `?id=xxx` → scroll + highlight 3.5 giây |

### 5. OrderItem lưu tên sản phẩm tại thời điểm mua
```prisma
model OrderItem {
  productName String? // Tên SP lưu tại thời điểm mua
  price       Int     // Giá SP tại thời điểm mua
}
```
→ Tránh mất dữ liệu khi SP bị sửa tên/giá hoặc xóa sau này.

### 6. Animation UX
- Thêm vào giỏ: icon badge "Đã thêm" animate-bounce 1 giây
- Xóa khỏi giỏ: `animate-slide-out-right` 300ms
- Giỏ hàng icon: `scale-110 rotate-3` khi thêm SP
- AI đề xuất: "Đã thêm" checkmark 1.5 giây

### 7. Không cần cài app để quét mã
- Camera API của trình duyệt điện thoại
- Không cần app riêng, chỉ cần quét QR bằng camera

