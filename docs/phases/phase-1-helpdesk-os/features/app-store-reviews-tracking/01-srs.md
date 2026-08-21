---
feature_slug: app-store-reviews-tracking
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

# SRS: App Store Customer Reviews & Automated 3-Way Tracking (`app-store-reviews-tracking`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **App Store Reviews** trong Internal Web App thuộc hệ thống Helpdesk OS. Tính năng này giải quyết bài toán tiếp nhận và xử lý khiếu nại của merchant từ Shopify App Store thông qua cơ chế **Tracking 3 Chiều Tự Động (Store Review ➔ Store Domain ➔ Crisp Chat ID)**, hỗ trợ CS Agent phân tích các chủ đề khiếu nại phổ biến, mở nhanh cuộc trò chuyện Crisp với merchant, điều hướng sang Shop 360°, và chuyển đổi 1-click từ bài review thành ticket kỹ thuật trên Slack.

---

## 2. Scope

### 2.1 In Scope
- **3-Way Auto Linking Banner**:
  - Banner giới thiệu cơ chế liên kết 3 chiều thông minh (`Store Review ➔ Store Domain ➔ Crisp Chat ID`).
- **Rating & Complaint Themes Summary**:
  - Khung thống kê điểm đánh giá trung bình: `4.8 / 5.0 (96 reviews)`, số lượng review tiêu cực `reviews < 3 stars (12)`.
  - Khung phân tích chủ đề khiếu nại hàng đầu (Top Complaint Themes): `CSS layout issue`, `Option loading slow`, `Pricing confusion`.
- **Reviews Feed Container (`#reviews-list-container`)**:
  - Danh sách bài đánh giá render động từ `state.reviews` qua hàm `renderReviews()`.
  - Cấu trúc mỗi thẻ review:
    - Dòng tiêu đề: Điểm số sao (`5/5`, `1/5`), Tiêu đề đánh giá, Badge phân loại (VD: `Positive Feedback` nền xanh, `Negative - Needs Outreach` nền đỏ), Thời gian đánh giá.
    - Dòng nội dung: Văn bản phản hồi chi tiết từ merchant.
    - Dòng Footer: Tên tác giả (`Author`), Domain cửa hàng (link gọi `navigateToShopInfo(domain)`), Tên người dùng Crisp (`Crisp user`), và nút tác vụ `+ Create Ticket`.
- **1-Click Ticket Creation Workflow (`createTicketFromReview(domain, title)`)**:
  - Tự động nạp `domain` vào ô `#add-domain-input`.
  - Tự động nạp chuỗi `Review khiếu nại: [Title]` vào ô `#add-request-input`.
  - Mở Modal Add Ticket để CS chọn App, Channel, và bắn Slack Thread.

### 2.2 Out of Scope
- Tự động phản hồi lên trang công khai Shopify App Store mà không qua tài khoản Shopify Partners của Admin.

---

## 3. Key Business Rules

- **3.1. 3-Way Auto Resolution Algorithm**: Khi có review mới từ Shopify App Store, hệ thống tự động đối chiếu thông tin tác giả, email và tên cửa hàng để tìm ra:
  1. Store Domain chính xác trên hệ thống.
  2. Crisp Chat Session ID tương ứng của merchant đó.
- **3.2. 1-Click Ticket Conversion**: Khi CS click vào nút `+ Create Ticket` trên bất kỳ thẻ review nào:
  1. Gọi hàm `createTicketFromReview(domain, title)`.
  2. Điền tự động `domain` vào form tạo ticket.
  3. Điền tự động mô tả `Review khiếu nại: [title]` vào ô yêu cầu.
  4. Mở Modal Add Ticket `#modal-add-ticket`.
- **3.3. Shop 360° Seamless Navigation**: Click vào tên domain cửa hàng tại dòng footer của thẻ review sẽ kích hoạt hàm `navigateToShopInfo(domain)` để chuyển ngay sang view Shop 360° của cửa hàng đó.
- **3.4. No-Emoji Plain Styling**: Toàn bộ các nút bấm và nhãn trạng thái trong màn hình Reviews đều áp dụng phong cách plain button, không chứa icon emoji.

---

## 4. Domain Model

### 4.1 Entities
- `AppReview`:
  - `id` (string): ID bài review.
  - `stars` (string): Số sao đánh giá (e.g. `5/5`, `1/5`).
  - `title` (string): Tiêu đề bài đánh giá.
  - `content` (string): Nội dung chi tiết.
  - `badge` (string): Nhãn phân loại (e.g. `Negative - Needs Outreach`).
  - `badgeColor` (string): CSS classes cho màu badge.
  - `time` (string): Thời gian đăng.
  - `author` (string): Tên tác giả.
  - `domain` (string): Domain cửa hàng được ánh xạ.
  - `crispUser` (string): Tên profile trên Crisp Chat.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `title` | string | Yes | None | Tiêu đề review từ Shopify App Store. |
| `content` | string | Yes | None | Nội dung bài đánh giá. |
| `domain` | string | Yes | None | Domain merchant được hệ thống tự động nhận diện. |
| `crisp_user` | string | No | `""` | Tên tài khoản chat trên Crisp nếu khớp được session. |

---

## 6. State Transitions

- **Chuyển đổi Review sang Ticket**:
  - `Review độc lập ➔ Click Create Ticket ➔ Pre-fill Modal ➔ Tạo Ticket thành công ➔ Đẩy Slack Thread`.

---

## 7. Runtime/behavior contract

- **Input**: Mảng `state.reviews`.
- **Output**: Render danh sách thẻ review vào `#reviews-list-container` qua `safeSetHTML`.
- **Fallback**: Nếu chưa có bài review nào, hiển thị thông báo an toàn `Chưa có bài đánh giá nào.`

---

## 8. QA Scenarios

1. Kiểm tra màn hình Reviews hiển thị banner 3-Way Auto Linking ở trên cùng.
2. Kiểm tra khối thống kê hiển thị điểm số trung bình `4.8 / 5.0 (96 reviews)` và số review tiêu cực `reviews < 3 stars (12)`.
3. Kiểm tra danh sách Top Complaint Themes hiển thị đủ các nhóm vấn đề (CSS layout, Option loading, Pricing).
4. Kiểm tra danh sách thẻ review render đúng số sao, tiêu đề, nội dung và tên tác giả.
5. Kiểm tra bài review 1 sao có badge màu đỏ `Negative - Needs Outreach`.
6. Kiểm tra bài review 5 sao có badge màu xanh lá `Positive Feedback`.
7. Kiểm tra click vào Store Domain trên thẻ review điều hướng thành công sang màn hình Shop 360° với domain tương ứng.
8. Kiểm tra click `+ Create Ticket` trên bài review mở Modal Add Ticket với trường Store Domain và Request Content đã được nạp sẵn.
9. Kiểm tra lưu tạo ticket từ review: ticket mới xuất hiện trên CS Dashboard và có đầy đủ thông tin domain.
10. Kiểm tra các nút trong màn hình Reviews không chứa icon emoji.
11. Kiểm tra màu sắc và độ tương phản chuẩn Light Mode.
12. Kiểm tra hàm `createTicketFromReview` hoạt động an toàn kể cả khi ô input form chưa được render.
13. Kiểm tra thông tin `Crisp user` hiển thị đúng tên merchant tương ứng.
14. Kiểm tra Toast thông báo hiển thị sau khi tạo ticket từ review thành công.
15. Kiểm tra danh sách review cuộn mượt mà trên custom scrollbar.

---

## 9. Implementation notes for AI code generation

- **Preserved DOM Identifiers**: `reviews-list-container`, `modal-add-ticket`, `add-domain-input`, `add-request-input`.
- **Defensive Rendering**: Sử dụng `safeSetHTML('reviews-list-container', ...)` để chống crash DOM.

---

## 10. Final implementation assumptions to review

- Cơ chế 3-way matching hoạt động dựa trên thuật toán so khớp domain và email đã định danh trong cơ sở dữ liệu.

---

## 11. User Stories

### Story: US-01 — CS chuyển đổi Review tiêu cực thành Ticket hỗ trợ 1-click
**As a** CS Agent  
**I want to** bấm nút `+ Create Ticket` trực tiếp từ bài review 1 sao trên App Store  
**So that** hệ thống tự động điền sẵn domain và nội dung khiếu nại vào ticket để tôi gửi ngay cho Dev team hỗ trợ.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang xem màn hình App Store Reviews và thấy bài review 1 sao của `kaifit.myapp.io`  
  **When** CS click vào nút `+ Create Ticket`  
  **Then** Modal Add Ticket tự động mở ra  
  **And** Ô Store Domain được điền sẵn `kaifit.myapp.io`  
  **And** Ô Request Content được điền sẵn `Review khiếu nại: [Title của review]`.
