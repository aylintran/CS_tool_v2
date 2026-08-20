---
feature_slug: app-store-reviews-tracking
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

# SRS: App Store Customer Reviews & Automated 3-Way Tracking (`app-store-reviews-tracking`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **App Store Customer Reviews** tích hợp tính năng **Tracking 3 Chiều Tự Động (Store Review ➔ Store Domain ➔ Crisp Chat ID)**. Màn hình giúp CS xem biểu đồ phân bố sao, danh mục khiếu nại hàng đầu, danh sách các bài review từ Shopify App Store, và chủ động mở phiên chat Crisp hoặc xem thông tin Shop 360° để hỗ trợ khách hàng ngay lập tức.

---

## 2. Scope

### 2.1 In Scope
- **Tracking 3 Chiều Tự Động**: Tự động khớp tên merchant/review trên Shopify App Store với Store Domain và phiên Crisp Chat ID của merchant đó.
- **Top Stats Grid**: Biểu đồ Rating Breakdown (5★ -> 1★) và danh sách Top Complaint Themes (Scanner freeze, Billing...).
- **Recent Reviews Feed Item**:
  - Rating (sao), Tiêu đề review, Category badge (`Bug signal`, `Positive`...), Timestamp, Author details.
  - **Khung metadata tracking**: Hiển thị `Store Domain` (link clickable) và nút **`Crisp`**.
  - **Bộ nút tác vụ**: `Create ticket` (pre-fill form), `Crisp`, và **`Store Info`** (chuyển thẳng tới Shop 360°).

### 2.2 Out of Scope
- Tự động xóa các review xấu trên Shopify App Store (phải tuân thủ chính sách Shopify).

---

## 3. Key Business Rules

- **3.1. 3-Way Auto Linking**: Mỗi bài review kéo về từ Shopify App Store API được hệ thống quét thuật toán matching (Email / Store Domain / Author) để tìm Crisp Chat ID tương ứng.
- **3.2. One-click Ticket Conversion**: Bấm `Create ticket` từ bài review sẽ tự động mở Form Add Ticket với Store Domain và nội dung review được điền sẵn.
- **3.3. Store Info Navigation**: Bấm nút `Store Info` hoặc click vào Store Domain sẽ điều hướng CS thẳng tới màn hình Shop 360° của cửa hàng đó.

---

## 4. QA Scenarios

1. Kiểm tra hiển thị biểu đồ Rating Breakdown và Top Complaint Themes chính xác.
2. Kiểm tra mỗi thẻ Review hiển thị đúng số sao, tiêu đề, tác giả, và badge loại review.
3. Kiểm tra khung metadata hiển thị Store Domain matched và nút `Crisp`.
4. Kiểm tra click `Cris` mở đúng phiên chat của merchant đó trên Crisp.
5. Kiểm tra click `Store Info` trên thẻ review điều hướng thành công tới màn hình Shop 360°.
6. Kiểm tra click `Create ticket` mở Form Add Ticket pre-fill sẵn thông tin Store Domain và nội dung review.
7. Kiểm tra gửi ticket tạo từ review xuất hiện bài post trên Slack Thread.
8. Kiểm tra hiển thị banner hướng dẫn Tracking 3 chiều tự động ở trên cùng màn hình.

---

## 5. User Stories

### Story: US-04 — CS chuyển Review tiêu cực thành Ticket hỗ trợ và mở Crisp Chat 1-click
**As a** CS Agent  
**I want to** xem bài đánh giá kém sao trên App Store Feed, thấy ngay Store Domain và nút mở Crisp Chat  
**So that** mở chat hỗ trợ merchant tức thì và chuyển review đó thành Ticket xử lý trên Slack.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang ở màn hình App Store Customer Reviews  
  **When** CS thấy bài review 1 sao về lỗi Scanner freeze từ store `kaifit.myapp.io`  
  **Then** Khung metadata hiển thị `Store Domain: kaifit.myapp.io` và nút `Cris (Mykola)`  
  **When** CS bấm `Create ticket`  
  **Then** Form Add Ticket mở ra với Store Domain `kaifit.myapp.io` và nội dung review được điền sẵn.

### Links
- Business rule: [SRS Section 3.1](#31-3-way-auto-linking) & [Section 3.2](#32-one-click-ticket-conversion)
