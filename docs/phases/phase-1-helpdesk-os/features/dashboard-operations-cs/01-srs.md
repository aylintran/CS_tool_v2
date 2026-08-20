---
feature_slug: dashboard-operations-cs
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

# SRS: CS Operational Dashboard & Ticket Controls (`dashboard-operations-cs`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **CS Dashboard** trong Internal Web App. Đây là trung tâm điều hành làm việc của nhân viên CS để theo dõi chỉ số ticket trong ca, lọc ticket theo App/Status/Assignee, tìm kiếm real-time, chỉnh sửa ticket (Edit Ticket với Request Content bị khóa + Shopify Tagging Modal), và bàn giao ca trực (Transfer Ticket).

---

## 2. Scope

### 2.1 In Scope
- **3 Stat Cards**: Đếm tổng ticket mở trong ca, ticket gán cho CS hiện tại (Aylin Tran), và ticket unread.
- **Filter Toolbar**: Thanh lọc theo App (`ALL`, `APO`, `APB`, `ACS`), bộ lọc Dropdown chứa đầy đủ **12 Trạng Thái Custom**, và nút `Assigned to me`.
- **Search Bar**: Ô tìm kiếm real-time theo Request content, Store domain, hoặc Ticket ID.
- **Thẻ Ticket 2 Cột**:
  - Cột trái: Badges Status, Urgency, Ticket ID, App tag, Channel, Store URL, Assigned CS, Request content, Tags, Nút tác vụ (`View Slack`, `Edit`, `Transfer`, `Store Info`).
  - Cột phải: **Dual Yellow Note Cards** (`📝 Note transfer case` & `📄 Summary note`).
- **Modal Edit Ticket**:
  - Ô Request Content bị **KHÓA (Read-only)** vì đồng bộ trực tiếp với Slack Thread gốc.
  - Cho phép sửa Status (12 trạng thái), Urgency, Summary Note.
  - Tích hợp **Modal Tagging như Shopify** (chọn checkbox, gõ tag mới, gỡ tag pills).
- **Modal Transfer Ticket**: Bàn giao ca trực kèm Handoff Note và Summary Note auto-fill từ DB.

### 2.2 Out of Scope
- Chỉnh sửa trực tiếp bài đăng trên Slack Thread từ Dashboard (phải sửa trên Slack).
- Quản lý phân quyền tài khoản Admin/Dev (thuộc màn hình Kanban Board).

---

## 3. Key Business Rules

- **3.1. Lock Request Content Policy**: Trong Modal Edit Ticket, ô `Request Content` hoàn toàn **Read-only / Locked** nhằm bảo toàn tính đồng bộ với bài đăng Slack Thread gốc.
- **3.2. Shopify-style Tagging**: Gán nhiều tags trên Ticket qua giao diện Modal Tagging chuẩn Shopify. Các tag được render dạng pills xanh có nút `✕` gỡ trực quan.
- **3.3. Dual Yellow Note Storage**:
  - `Note transfer case`: Ghi chú ca trực ngắn hạn (chỉ lưu cho ca giao tiếp theo).
  - `Summary note`: Tóm tắt cốt lõi lưu DB cố định, tự động load vào Form Transfer mà CS không cần copy-paste.
- **3.4. Real-time Search**: Tìm kiếm tức thì khi CS gõ từ khóa vào ô Search mà không cần reload trang.

---

## 4. Domain Model

### 4.1 Entities
- `DashboardStats`: `total_open_tickets`, `assigned_to_me_tickets`, `unread_tickets`.
- `Ticket`: `id`, `app`, `channel`, `store_url`, `status`, `urgency`, `assigned_to`, `timestamp`, `request_content`, `tags`, `handoff_note`, `summary_note`, `unread`.
- `Tag`: `name`, `color`.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `ticket_id` | string | Yes | None | Mã duy nhất (e.g. `TK-4821`). |
| `status` | enum | Yes | `Chờ check - Dev` | Phải thuộc 1 trong 12 `TicketStatus` enums. |
| `urgency` | integer | Yes | `1` | `1` (Khẩn cấp) hoặc `2` (Bình thường). |
| `request_content` | string | Yes | Read-only | Đọc từ DB/Slack, không được sửa trong Edit Modal. |
| `tags` | array<string> | No | `[]` | Danh sách tên tag gán vào ticket. |
| `summary_note` | string | No | `""` | Tóm tắt cốt lõi lưu DB. |
| `handoff_note` | string | No | `""` | Ghi chú ca trực ngắn hạn. |

---

## 6. QA Scenarios

1. Kiểm tra 3 thẻ đếm chỉ số hiển thị đúng số lượng ticket mở trong ca.
2. Kiểm tra lọc ticket theo App (`APO`, `APB`, `ACS`) trả về kết quả chính xác.
3. Kiểm tra dropdown Status chứa đủ 12 trạng thái custom quy chuẩn.
4. Kiểm tra gõ từ khóa tìm kiếm real-time lọc thẻ ticket tức thì.
5. Kiểm tra thẻ Ticket Card hiển thị đầy đủ 2 cột và 2 khung ghi chú vàng.
6. Kiểm tra mở Edit Ticket Modal thấy ô Request Content bị khóa (disabled/read-only).
7. Kiểm tra mở Shopify Tagging Modal, chọn/bỏ chọn checkbox tag hoạt động đúng.
8. Kiểm tra gõ tag mới trong Tagging Modal và bấm Add tags hiển thị badge pills mới.
9. Kiểm tra bấm nút `✕` trên tag pill gỡ tag ra khỏi ticket.
10. Kiểm tra mở Transfer Modal thấy Summary Note được tự động load từ DB.
11. Kiểm tra nhập Handoff Note mới và lưu Transfer cập nhật khung vàng `Note transfer case`.
12. Kiểm tra click `🏪 Store Info` chuyển tới màn hình Shop 360° của store đó.
13. Kiểm tra click `View Slack` mở link Slack Thread tương ứng.
14. Kiểm tra nút `Assigned to me` chỉ lọc ticket của CS hiện tại.
15. Kiểm tra giao diện Toast notification hiển thị sau mỗi thao tác lưu.

---

## 7. User Stories

### Story: US-02 — CS chỉnh sửa Ticket và gán Tagging chuẩn Shopify
**As a** CS Agent  
**I want to** mở Modal Edit Ticket để thay đổi trạng thái, tóm tắt cốt lõi và gán nhiều Tags cho ticket  
**So that** phân loại vấn đề kỹ thuật rõ ràng mà không làm sai lệch bài đăng Slack Thread gốc.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang ở màn hình CS Dashboard và chọn bấm nút `✏️ Edit` trên một Ticket  
  **When** Modal Edit Ticket hiện ra  
  **Then** Ô `Request Content` bị khóa không cho chỉnh sửa (disabled/read-only)  
  **And** CS có thể chọn đổi Trạng thái trong 12 trạng thái custom, mở Shopify Tagging Modal để tích chọn thêm tag `Theme Conflict` và bấm `Lưu Thay Đổi`  
  **Then** Thẻ Ticket Card được cập nhật ngay lập tức với tag mới và trạng thái mới.

### Links
- Business rule: [SRS Section 3.1](#31-lock-request-content-policy) & [Section 3.2](#32-shopify-style-tagging)
