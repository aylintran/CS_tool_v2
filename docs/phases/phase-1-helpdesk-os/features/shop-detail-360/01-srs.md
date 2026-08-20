---
feature_slug: shop-detail-360
phase_slug: phase-1-helpdesk-os
doc_type: srs
version: 1.0
status: draft
owner: "@ba"
reviewers: ["@po", "@tech-lead", "@qa-lead"]
created_at: 2026-08-20
last_updated: 2026-08-20
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
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **Shop 360° View** trong Internal Web App. Màn hình này cung cấp góc nhìn toàn cảnh về 1 cửa hàng merchant: tra cứu bằng Store Domain, xem Account Info + Visitor Data, quản lý CS Pinned Notes, và **hiển thị toàn bộ danh sách Ticket thuộc Store đó** dưới dạng định dạng thẻ 2 cột đầy đủ.

---

## 2. Scope

### 2.1 In Scope
- **Search by Store Domain**: Ô tìm kiếm `🔍 Enter store domain...` kèm nút `Search` ở góc trên bên phải cho phép CS nhập domain bất kỳ để tải toàn bộ dữ liệu 360° của store.
- **Needs Action Section**: 2 thẻ chỉ số `URGENT` (tickets cần xử lý ngay) và `REVIEWS < 3★`.
- **Account Info & Visitor Meta**: Bảng thông tin tài khoản và thông tin Visitor Data (Shop ID, Email, Owner, Plan, Country, App Version, Rating).
- **Add Sub-domain**: Cho phép CS nhập sub-domain bất kỳ và bấm `+` để thêm sub-domain vào danh sách tracking của Store.
- **CS Pinned Notes**: Khung ghi chú ghim nội bộ riêng cho Shop. Nút `+ Add note` mở prompt thêm ghi chú mới.
- **Tất Cả Ticket Của Store Này**: Danh sách toàn bộ ticket thuộc Store (và các Sub-domains của Store), hiển thị theo **đúng 100% định dạng thẻ 2 cột đầy đủ** (kèm 2 khung vàng `Note transfer case` & `Summary note`, nút `View Slack`, `Edit`, `Transfer`).

### 2.2 Out of Scope
- Thay đổi trực tiếp gói cước Billing của Merchant trên Shopify.
- Xóa lịch sử log giao dịch thanh toán.

---

## 3. Key Business Rules

- **3.1. Store Domain Resolution**: Khi CS nhập Store Domain hoặc Sub-domain bất kỳ vào ô Search, hệ thống tự động quy đổi về Store ID gốc và hiển thị dữ liệu tập trung.
- **3.2. Full Dual-Column Ticket Render**: Tất cả ticket hiển thị ở phần "TẤT CẢ TICKET CỦA STORE NÀY" phải sử dụng chung thành phần UI Ticket Card 2 cột đầy đủ giống hệt CS Dashboard, không được cắt giảm thông tin.
- **3.3. Persistent CS Notes**: Ghi chú ghim CS Pinned Notes được lưu vĩnh viễn theo Shop ID và hiển thị cho tất cả nhân viên CS khi truy cập Shop đó.
- **3.4. Sub-domain Tracking & Management**: CS có thể thêm sub-domain liên kết với Store. Khi một sub-domain mới được thêm thành công, tất cả ticket liên quan đến sub-domain đó sẽ được tự động gom về danh sách ticket của Store.

---

## 4. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `store_domain` | string | Yes | None | Khóa tra cứu chính của Shop. |
| `sub_domains` | array<string> | No | `[]` | Mảng các sub-domain liên kết với Store Domain chính. |
| `shop_name` | string | Yes | None | Tên cửa hàng merchant. |
| `account_status` | string | Yes | `ACTIVE` | `ACTIVE` hoặc `UNINSTALL`. |
| `visitor_data` | object | Yes | `{}` | Chứa `store_id`, `country`, `user_agent`, `app_version`... |
| `pinned_notes` | array<string> | No | `[]` | Danh sách chuỗi ghi chú ghim nội bộ. |

---

## 5. QA Scenarios

1. Kiểm tra gõ domain `kaifit.myapp.io` vào ô Search và bấm Search tải đúng dữ liệu Shop Kai Fitness.
2. Kiểm tra thông tin Shop ID, Email, Owner, Country, App Version hiển thị chính xác.
3. Kiểm tra thẻ URGENT hiển thị đúng số lượng ticket khẩn cấp của shop.
4. Kiểm tra phần "TẤT CẢ TICKET CỦA STORE NÀY" hiển thị đầy đủ các ticket thuộc store đó.
5. Kiểm tra các thẻ ticket hiển thị đúng cấu trúc 2 cột kèm 2 khung vàng `Note transfer case` & `Summary note`.
6. Kiểm tra bấm `+ Add note` nhập ghi chú mới hiển thị ngay trong danh sách CS Pinned Notes.
7. Kiểm tra tìm kiếm domain chưa tồn tại hiển thị thông báo rỗng thân thiện.
8. Kiểm tra bấm nút `Edit` trên ticket trong Shop 360 mở Edit Modal hoạt động bình thường.
9. Kiểm tra bấm nút `Transfer` trên ticket trong Shop 360 mở Transfer Modal hoạt động bình thường.
10. Kiểm tra nhập sub-domain mới và bấm `+` tại Shop 360 bổ sung sub-domain mới vào danh sách tracking và tự động gom các ticket liên quan về Store.

---

## 6. User Stories

### Story: US-03 — CS tra cứu Shop 360° bằng Store Domain và xem toàn bộ lịch sử Ticket
**As a** CS Agent  
**I want to** gõ Store Domain vào ô tìm kiếm trên màn hình Shop 360°  
**So that** xem toàn bộ thông tin tài khoản, Visitor Data và lịch sử các ticket của store đó dưới dạng thẻ 2 cột đầy đủ.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang ở màn hình Shop 360° View  
  **When** CS nhập `woodesign.ie.myshopify.com` vào ô Search và bấm `Search`  
  **Then** Màn hình tải thông tin cửa hàng WooDesign cùng toàn bộ ticket thuộc store đó  
  **And** Các ticket hiển thị đầy đủ 2 cột thông tin kèm 2 khung vàng ghi chú.

### Links
- Business rule: [SRS Section 3.1](#31-store-domain-resolution) & [Section 3.2](#32-full-dual-column-ticket-render)
