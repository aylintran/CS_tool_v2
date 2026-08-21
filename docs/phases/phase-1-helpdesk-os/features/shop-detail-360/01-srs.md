---
feature_slug: shop-detail-360
phase_slug: phase-1-helpdesk-os
doc_type: srs
version: 2.1
status: approved
owner: "@ba"
reviewers: ["@po", "@tech-lead", "@qa-lead"]
created_at: 2026-08-20
last_updated: 2026-08-21
links:
  related_docs:
    - ../../00-prd.md
depends_on:
  - ../../00-prd.md
consumed_by:
  - ./03-tech-design.md
---

# SRS: Shop 360° Comprehensive View (`shop-detail-360`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **Shop 360° View** trong Internal Web App thuộc Helpdesk OS. Tính năng này cung cấp một trung tâm tra cứu dữ liệu 360 độ về một merchant cụ thể: tìm kiếm theo domain, hiển thị tình trạng sức khỏe tài khoản (Needs Attention / No Feedback Loop), thông tin tài khoản và cấu hình kỹ thuật (Account Info & Visitor Meta), quản lý ghi chú CS Pinned Notes, và danh sách toàn bộ ticket thuộc store đó với cấu trúc thẻ 2 cột và Dual Note Cards tương tự CS Dashboard.

---

## 2. Scope

### 2.1 In Scope
- **Domain Search Bar (`#store-search-domain-input`)**:
  - Ô nhập domain cửa hàng (VD: `kaifit.myapp.io`, `woodesign.ie.myshopify.com`).
  - Nút bấm `Search` thực thi hàm `handleStoreSearchByDomain()`.
- **Shop Header Banner**:
  - Tiêu đề tên Shop dạng chữ in hoa (`#shop-name-title`, VD: `KAIFIT STORE`).
  - Dòng mô tả trạng thái gói cước (`#shop-subdomain-desc`, VD: `kaifit.myapp.io • Growth plan • GMT+7`).
  - Nút tác vụ `Open in Crisp` cho phép mở ngay phiên chat của store trên Crisp Chat.
- **Left Column: Needs Attention & Health Alerts**:
  - Thẻ `NEEDS ATTENTION` (cảnh báo viền đỏ `bg-red-50 border border-red-200`) thông báo số lượng ticket Urgent đang chờ xử lý và nút điều hướng tới ticket.
  - Thẻ `NO FEEDBACK LOOP` (cảnh báo viền xanh/xám `bg-slate-50 border border-slate-200`) thông báo các ticket đóng không có CSAT và nút `Send check-in`.
- **Right Column: Account Info & CS Notes**:
  - Bảng **Account Info & Visitor Meta**:
    - `Shop ID`: Mã số ID cửa hàng (VD: `55302118874`).
    - `Email`: Email liên hệ quản trị (`anh@kaifit.vn`).
    - `Owner`: Tên chủ sở hữu (`Kai Fitness Co.`).
    - `Plan`: Gói cước Shopify (`Growth`).
    - `Country`: Quốc gia (`Vietnam`).
    - `App Version`: Phiên bản ứng dụng (`v2`).
    *(Lưu ý: Dòng Rating đã được lược bỏ theo quy chuẩn giao diện mới).*
  - Khung **CS Pinned Notes (`#shop-notes-list`)**:
    - Danh sách các ghi chú nội bộ được ghim cố định cho store.
    - Nút `+ Add note` gọi hàm `addShopNote()` mở hộp thoại nhập ghi chú mới và lưu tức thì.
- **Bottom Section: All Tickets for This Store (`#shop-tickets-container`)**:
  - Tiêu đề hiển thị tổng số ticket của store: `TẤT CẢ TICKET CỦA STORE NÀY ([Count])`.
  - Danh sách ticket cards đầy đủ 2 cột: Cột 1 gồm badges (App, Status, Urgency, Reopened), Ticket ID + Channel, Store link, Phân công, Request content dạng text không khung, Tag pills, và Plain Action Buttons (`View Crisp`, `View Slack`, `Edit`, `Transfer`); Cột 2 gồm Dual Note Cards (`Note transfer case` & `Summary note`).

### 2.2 Out of Scope
- Thay đổi trực tiếp phương thức thanh toán thẻ tín dụng của merchant trên cổng thanh toán Shopify Billing.

---

## 3. Key Business Rules

- **3.1. Dynamic Store Domain Resolution**: Khi CS nhập domain vào `#store-search-domain-input` và bấm Search, hàm `handleStoreSearchByDomain()` thực hiện:
  1. Cập nhật `state.currentStoreDomain = domainVal`.
  2. Cập nhật tiêu đề Shop Title và Subdomain Description.
  3. Lọc danh sách tickets trong `state.tickets` theo điều kiện `t.storeUrl.toLowerCase().includes(domainVal.toLowerCase())`.
  4. Cập nhật biến đếm `#shop-ticket-count-badge` và `#shop-urgent-count`.
  5. Render danh sách thẻ ticket vào `#shop-tickets-container`.
  6. Bắn Toast: `Đã tải dữ liệu Store 360° cho domain: [Domain]`.
- **3.2. Direct Navigation from Dashboard**: Khi CS click vào liên kết Store URL hoặc nút `Store Info` trên bất kỳ thẻ ticket nào ở CS Dashboard hoặc Reviews, hệ thống gọi hàm `navigateToShopInfo(domain)` tự động chuyển sang view Shop 360° và nạp sẵn dữ liệu của store đó.
- **3.3. Persistent CS Pinned Notes**: CS Pinned Notes cho phép nhân viên ghi lại các đặc thù của khách hàng (VD: kênh liên lạc ưa thích, giờ liên hệ). Ghi chú mới được tạo thông qua `addShopNote()` sẽ được append vào danh sách `#shop-notes-list` và hiển thị chữ nghiêng màu slate.
- **3.4. Ticket UI Consistency**: Mọi thẻ ticket hiển thị tại Shop 360° View bắt buộc tuân thủ 100% quy chuẩn UI của CS Dashboard: không icon emoji, Request content không khung viền xám, badge App ở đầu, mã ticket bên cạnh channel.

---

## 4. Domain Model

### 4.1 Entities
- `ShopDetail`:
  - `storeDomain` (string): Domain chính.
  - `storeName` (string): Tên thương hiệu cửa hàng.
  - `shopId` (string): Shopify Shop ID.
  - `ownerEmail` (string): Email chủ cửa hàng.
  - `ownerName` (string): Tên chủ cửa hàng.
  - `plan` (string): Gói cước Shopify.
  - `country` (string): Quốc gia đăng ký.
  - `appVersion` (string): Phiên bản app đang chạy.
  - `pinnedNotes` (array<string>): Danh sách ghi chú nội bộ.
  - `urgentTicketCount` (integer): Số ticket khẩn cấp đang mở.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `store_domain` | string | Yes | None | Phải là chuỗi domain hợp lệ, không chứa khoảng trắng. |
| `shop_id` | string | Yes | None | Chuỗi số nguyên dương đại diện ID cửa hàng. |
| `owner_email` | string | Yes | None | Chuỗi định dạng email hợp lệ. |
| `pinned_notes` | array<string> | No | `[]` | Mảng chứa các chuỗi văn bản ghi chú CS. |

---

## 6. State Transitions

- **Tìm kiếm Store**:
  - `Nhập Domain ➔ Tìm kiếm thành công ➔ Render dữ liệu Shop 360°`.
- **Thêm CS Note**:
  - `Nhập text ➔ Bấm lưu ➔ Append vào #shop-notes-list ➔ Toast thông báo`.

---

## 7. Runtime/behavior contract

- **Input**: Chuỗi domain từ `#store-search-domain-input` hoặc tham số gọi từ `navigateToShopInfo(domain)`.
- **Output**: Cập nhật các element `#shop-name-title`, `#shop-subdomain-desc`, `#shop-ticket-count-badge`, `#shop-urgent-count`, `#shop-tickets-container`.
- **Fallback**: Nếu domain không có ticket nào, render thông báo an toàn: `Chưa có ticket nào được tạo cho store domain này.`

---

## 8. QA Scenarios

1. Kiểm tra nhập `woodesign.ie.myshopify.com` vào ô Search và bấm Search nạp đúng tiêu đề `WOODESIGN STORE`.
2. Kiểm tra thông tin Shop ID, Email, Owner, Plan, Country, App Version hiển thị đầy đủ và chính xác.
3. Xác nhận dòng Rating không còn xuất hiện trong bảng Account Info.
4. Kiểm tra thẻ cảnh báo `NEEDS ATTENTION` hiển thị đúng số lượng ticket Urgent của store.
5. Kiểm tra danh sách "TẤT CẢ TICKET CỦA STORE NÀY" hiển thị đúng các ticket của domain đó.
6. Kiểm tra các thẻ ticket tại Shop 360° có App badge ở đầu, Request content không khung, và action buttons là plain button không có emoji.
7. Kiểm tra click `Edit` hoặc `Transfer` trên ticket tại Shop 360° mở đúng các Modal tương ứng.
8. Kiểm tra bấm `+ Add note`, nhập nội dung ghi chú và bấm OK: ghi chú mới xuất hiện ngay trong `#shop-notes-list`.
9. Kiểm tra click vào Store URL từ CS Dashboard tự động điều hướng sang Shop 360° và nạp đúng domain.
10. Kiểm tra click `Open in Crisp` chuyển sang nền tảng mô phỏng Crisp Chat.
11. Kiểm tra tìm kiếm domain chưa có ticket nào hiển thị thông báo rỗng dạng italic thân thiện.
12. Kiểm tra ô tìm kiếm domain tự động cắt bỏ khoảng trắng thừa đầu cuối (`trim()`).
13. Kiểm tra biến đếm `#shop-ticket-count-badge` khớp với số lượng thẻ ticket được render.
14. Kiểm tra Toast thông báo xuất hiện sau mỗi lần tìm kiếm thành công.
15. Kiểm tra bố cục 2 cột (Needs Attention bên trái, Account Info bên phải) hiển thị cân đối và đáp ứng tốt trên các kích thước màn hình.

---

## 9. Implementation notes for AI code generation

- **Preserved DOM Identifiers**: `store-search-domain-input`, `shop-name-title`, `shop-subdomain-desc`, `shop-id-val`, `shop-email-val`, `shop-owner-val`, `shop-plan-val`, `shop-notes-list`, `shop-ticket-count-badge`, `shop-urgent-count`, `shop-tickets-container`.
- **Defensive Rendering**: Luôn kiểm tra `document.getElementById` trước khi gán text/HTML.

---

## 10. Final implementation assumptions to review

- Dữ liệu demo gán mặc định metadata ban đầu của store là Kai Fitness và WooDesign.

---

## 11. User Stories

### Story: US-01 — CS tra cứu toàn diện Shop 360° và thêm ghi chú Pinned Note
**As a** CS Agent  
**I want to** tra cứu một domain cửa hàng trên Shop 360° View và thêm ghi chú đặc thù khách hàng  
**So that** tôi và các đồng nghiệp ca sau nắm bắt đầy đủ lịch sử hỗ trợ và lưu ý quan trọng về merchant này.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang ở màn hình Shop 360° View  
  **When** CS nhập `woodesign.ie.myshopify.com` và bấm Search  
  **Then** Giao diện hiển thị thông tin cửa hàng Woodesign, các cảnh báo chú ý, và danh sách toàn bộ ticket của store  
  **And** CS bấm `+ Add note`, nhập `Khách hay chat lúc 15h chiều`  
  **Then** Ghi chú được thêm vào danh sách CS Pinned Notes và bắn Toast thông báo.
