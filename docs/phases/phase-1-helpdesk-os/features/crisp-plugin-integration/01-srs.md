---
feature_slug: crisp-plugin-integration
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

# SRS: Crisp Plugin Integration & Ticket / Store Controls (`crisp-plugin-integration`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho Sidebar Plugin tích hợp trực tiếp bên trong giao diện Crisp Chat. Plugin giúp nhân viên CS tiếp nhận chat của merchant, kiểm tra thông tin Store Domain / Sub-domain / Visitor Data, tạo Ticket đẩy lên Slack Thread dưới tên cá nhân CS, và thực hiện chuyển giao ca trực (Transfer Ticket).

---

## 2. Scope

### 2.1 In Scope
- **Tab Tickets**: Hiển thị danh sách ticket của store hiện tại. Nút `+ Add ticket` mở form tạo ticket mới. Nút `View Slack` mở thread Slack tương ứng. Nút `Transfer` mở modal bàn giao ticket.
- **Tab Store Info**: Hiển thị bảng **Visitor Data** (store_url, store_id, store_country, store_plan, store_email, user_agent, add_charge, app_version, app_plan, pricing_ver) và danh sách các sub-domain đã liên kết.
- **Form Add Ticket**: Form tạo ticket gồm các trường: Target App (`APO`, `APB`, `ACS`), Slack Channel (`#apo-paid-task`, `#apo-urgent-case`...), Status (12 trạng thái), Urgency (1/2), Store Domain, Feature Tag, Request Content.

### 2.2 Out of Scope
- **Add Sub-domain**: Tính năng thêm và quản lý sub-domain thuộc phạm vi màn hình **Shop 360° Detail** (Crisp Plugin chỉ hiển thị read-only các sub-domain đã được thêm).
- Chỉnh sửa trực tiếp tin nhắn gốc trên Crisp Chat.
- Quản lý phân quyền tài khoản admin Slack (thuộc về Slack App Configuration).

---

## 3. Key Business Rules

- **3.1. Slack OAuth Integration**: Khi CS bấm `Gửi Ticket lên Slack`, hệ thống sử dụng Slack User OAuth Token của chính CS đó (ví dụ: Aylin Tran) để post bài vào Slack Channel được chọn.
- **3.2. Auto Domain Matching**: Plugin tự động lấy Crisp Website ID / Visitor URL để tra cứu Store Domain và Sub-domains tương ứng trong DB.
- **3.3. Dual Note Structure**:
  - `Handoff / Transfer note`: Ghi chú ngắn hạn cho ca trực (không lưu dài hạn).
  - `Summary note`: Tóm tắt cốt lõi ticket lưu cố định trong DB, tự động fill vào Transfer modal khi mở.
- **3.4. Sub-domain Tracking**: Khi một sub-domain mới được thêm thành công, tất cả ticket liên quan đến sub-domain đó sẽ được tự động gom vào danh sách ticket của Store.

---

## 4. Domain Model

### 4.1 Entities
- `CrispSession`: Lưu thông tin session chat Crisp (website_id, session_id, visitor_email).
- `StoreInfo`: Thông tin cửa hàng (store_domain, sub_domains, store_id, country, app_plan, visitor_data).
- `Ticket`: Vé hỗ trợ (ticket_id, store_domain, status, urgency, assigned_to, request_content, summary_note, handoff_note, tags).

### 4.2 Enums
- `TargetApp`: `APO`, `APB`, `ACS`.
- `UrgencyLevel`: `1` (Khẩn cấp), `2` (Bình thường).
- `TicketStatus`: 12 trạng thái custom (`Chờ collab - CS`, `Chờ check - Dev`, `Đang check - Dev`, `Đã check - Dev`, `Rejected - Dev lead`, `CHỜ KHÁCH - CS`, `Chờ CS`, `Uninstall`, `Done - CS`, `Fl up 1 (12h)`, `Fl up 2 (24h)`, `Fl up 3 (36h)`).

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `store_domain` | string | Yes | None | Phải đúng định dạng domain (e.g. `domain.myshopify.com` hoặc custom domain). |
| `sub_domains` | array<string> | No | `[]` | Mảng các sub-domain liên kết với Store Domain chính. |
| `target_app` | enum | Yes | `APO` | Phải thuộc `TargetApp` enum. |
| `slack_channel` | string | Yes | `#apo-paid-task` | Tên channel Slack hợp lệ có CS và Dev tham gia. |
| `status` | enum | Yes | `Chờ check - Dev` | Phải thuộc 1 trong 12 `TicketStatus` enums. |
| `urgency` | integer | Yes | `1` | Giá trị `1` (Khẩn cấp) hoặc `2` (Bình thường). |
| `request_content` | string | Yes | None | Tối thiểu 5 ký tự. Đồng bộ trực tiếp làm nội dung bài đăng trên Slack Thread. |
| `summary_note` | string | No | `""` | Tóm tắt cốt lõi lưu DB. Được dùng để auto-fill vào form Transfer. |
| `handoff_note` | string | No | `""` | Ghi chú ca trực ngắn hạn giữa các CS. |

---

## 6. State Transitions

- **Tạo ticket mới**: `(None) -> Chờ check - Dev` hoặc `Chờ collab - CS`.
- **Bàn giao ca**: `Chuyển assigned_to từ CS ca cũ sang CS ca mới` (giữ nguyên hoặc cập nhật Status).

### 6.1 Disallowed Transitions
- Không cho phép chuyển trạng thái về `None` hoặc trạng thái không thuộc 12 trạng thái quy chuẩn.

---

## 7. Runtime / Behavior Contract

- **Input**: User action click nút trên Crisp Plugin Sidebar.
- **Output**: Tạo Ticket mới trong DB, phát API Slack `chat.postMessage` qua User Token, tự động re-render danh sách ticket trên Crisp Plugin.
- **Fallback**: Nếu Slack API lỗi ngắt kết nối, ticket vẫn được ghi nhận trong DB với nhãn `Slack Pending Sync` và không crash giao diện Crisp Plugin.Đồng thời có cảnh báo "System slack connection failed" trong phần Tab Tickets và thêm nút 'resend' khi click vào sẽ gửi lại message lên slack.

---

## 8. QA Scenarios

1. Kiểm tra hiển thị Tab Tickets với đúng Store Domain tương ứng phiên Crisp Chat.
2. Kiểm tra click `+ Add ticket` mở modal tạo ticket đúng với dữ liệu domain pre-fill.
3. Kiểm tra chọn đúng 1 trong 12 trạng thái custom trên Add Ticket Modal.
4. Kiểm tra gửi ticket thành công xuất hiện bài post trên Slack dưới tên CS cá nhân.
5. Kiểm tra tin nhắn đếm counter ticket trên Crisp Plugin tăng lên ngay sau khi tạo.
6. Kiểm tra chuyển sang Tab Store Info hiển thị đầy đủ thông tin Visitor Data (store_id, user_agent...).
7. Kiểm tra danh sách sub-domain của Store hiển thị đầy đủ và chính xác trên Tab Store Info.
8. Kiểm tra khi một sub-domain được thêm từ màn hình Shop 360°, thông tin sub-domain và các ticket liên quan tự động hiển thị trên Crisp Plugin.
9. Kiểm tra click `Transfer` trên ticket mở modal bàn giao ca pre-fill Summary Note.
10. Kiểm tra nhập Handoff Note mới và submit Transfer thành công.
11. Kiểm tra click `View Slack` mở tab trang Slack thread tương ứng.
12. Kiểm tra khi chuyển cửa sổ chat Crisp khác, thông tin Crisp Plugin tự động reload tương ứng.
13. Kiểm tra tính năng lọc ticket theo app trên Crisp Plugin.
14. Kiểm tra hiển thị thông báo Toast khi thao tác thành công.
15. Kiểm tra trường hợp mất kết nối mạng hiển thị lỗi thân thiện.

---

## 9. Implementation Notes for AI Code Generation

- Sử dụng Crisp Widget SDK (`$crisp.get()` & `$crisp.push()`) để tương tác với chat session.
- Đảm bảo token Slack OAuth được lưu an toàn trong kho Secrets/Env và chỉ gọi qua backend API trung gian.
- Response time hiển thị Plugin phải < 300ms.

---

## 10. Final Implementation Assumptions to Review

- Giả định tất cả CS Agent đã đăng nhập thành công tài khoản Slack cá nhân trên ứng dụng Helpdesk OS.

---

## 11. User Stories

### Story: US-01 — CS tạo Ticket từ Crisp Plugin đẩy lên Slack Thread
**As a** CS Agent  
**I want to** tạo ticket trực tiếp từ Crisp Plugin bên cạnh cửa sổ chat với merchant  
**So that** tóm tắt yêu cầu của khách và gửi lên Slack Thread cho Dev hỗ trợ xử lý mà không cần mở nhiều tab.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang mở cuộc chat với merchant trên Crisp  
  **When** CS bấm nút `+ Add ticket` trên Crisp Plugin, chọn Channel `#apo-paid-task` và nhập nội dung `Request: Button style mismatch`  
  **Then** Hệ thống tạo Ticket mới trong DB và gửi tin nhắn đăng bài lên kênh Slack `#apo-paid-task` dưới tên tài khoản Slack của CS  
  **And** Thẻ Ticket mới xuất hiện ngay lập tức trên Crisp Plugin và CS Dashboard.

- **Given** CS nhập nội dung request rỗng (dưới 5 ký tự)  
  **When** CS bấm `Gửi Ticket lên Slack`  
  **Then** Hệ thống báo lỗi: `"Request Content phải chứa ít nhất 5 ký tự"`.

### Links
- Business rule: [SRS Section 3.1](#31-slack-oauth-integration)

### Definition of Done
- [x] Code merged
- [x] Unit tests pass
- [x] AC tests pass (QA)
- [x] Docs updated
