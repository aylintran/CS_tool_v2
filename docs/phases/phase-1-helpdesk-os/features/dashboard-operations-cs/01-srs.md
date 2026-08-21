---
feature_slug: dashboard-operations-cs
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

# SRS: CS Operational Dashboard & Ticket Controls (`dashboard-operations-cs`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **CS Dashboard** trong Internal Web App thuộc hệ thống Helpdesk OS. Đây là trung tâm điều phối và xử lý ticket kỹ thuật hàng ngày của nhân viên Customer Support (CS Agent), cung cấp các công cụ: 3 thẻ thống kê ca trực, bộ lọc theo App/Assignee/Status, ô tìm kiếm thời gian thực, thẻ Ticket 2 cột với Dual Note Cards (`Note transfer case` và `Summary note`), cụm nút thao tác plain không kèm emoji, Modal Edit Ticket với Request Content bị khóa (read-only), Modal Tagging theo phong cách Shopify, và Modal Transfer Ticket có logic tự động đếm số lần Reopen.

---

## 2. Scope

### 2.1 In Scope
- **3 Metric Stat Cards**:
  - `Ticket mở trong ca`: Đếm tổng số ticket mở (`state.tickets.length`).
  - `Ticket gán cho tôi (Aylin)`: Đếm số ticket gán cho CS đang đăng nhập (`assignedTo === 'Aylin Tran'`).
  - `Ticket unread`: Đếm số ticket chưa đọc (`unread === true`).
- **Toolbar & Filter Controls**:
  - App Filter Buttons: `ALL`, `APO`, `APB`, `ACS` được sinh động từ cấu hình `state.config.apps`. Nút active có nền xanh `bg-blue-600 text-white shadow font-bold`, nút inactive có nền nhạt `bg-slate-100 text-slate-700 border border-slate-300`.
  - Quick Filter `Gán cho tôi` toggle button (`#filter-me-btn`).
  - Dropdown Status Filter (`#status-filter-select`) chứa `All Statuses` và toàn bộ 12 custom statuses.
  - Live Search Input (`#search-input`) tìm kiếm không độ trễ theo Request Summary, Store Domain, hoặc Ticket ID.
  - Nút `+ Add New Ticket` mở Modal tạo ticket mới.
- **Thẻ Ticket 2 Cột (2-Column Ticket Card)**:
  - Cột 1 (Thông tin & Hành động):
    - Hàng Header: Badge App ở đầu (VD: `APO`), Status Badge (VD: `Đang check - Dev`), Urgency Badge (VD: `Urgent`, `High`, `Medium`, `Low`), Reopened Badge (nếu `reopenCount > 0`), Ticket ID (`TK-4821`) đặt cùng cụm với Channel (`Channel: #apo-paid-task`).
    - Dòng Store URL: Clickable link điều hướng trực tiếp sang màn hình Shop 360° View.
    - Dòng Phân công: `Assigned to: [CS Name] at [Timestamp]`.
    - Dòng Request Content: Hiển thị dạng text liền mạch không có khung xám bao quanh.
    - Dòng Tags: Danh sách tag pills nền nhạt (`bg-slate-100 text-slate-700 border border-slate-200`).
    - Cụm Action Buttons: `View Crisp`, `View Slack`, `Edit`, `Transfer`, `Store Info` định dạng plain button, không kèm icon emoji, viền nhạt `border border-slate-200`, nền nhạt `bg-slate-50 hover:bg-slate-100`.
  - Cột 2 (Dual Note Cards):
    - Card 1: `Note transfer case` (nền vàng cam nhạt `bg-amber-50/80 border border-amber-200`) chứa ghi chú ngắn hạn giữa các ca trực CS.
    - Card 2: `Summary note` (nền xám nhạt `bg-slate-50 border border-slate-200`) chứa tóm tắt bản chất kỹ thuật lưu cố định trong DB.
- **Modal Edit Ticket (`#modal-edit-ticket`)**:
  - Request Content hoàn toàn bị KHÓA (read-only/disabled) để đảm bảo toàn vẹn dữ liệu Slack Thread gốc.
  - Cho phép sửa Status (dropdown 12 trạng thái), Urgency (`Urgent`, `High`, `Medium`, `Low`), Phân công (`Assigned To`), Summary Note.
  - Quản lý danh sách tag hiện tại với nút xóa `✕` trên từng tag pill và nút `+ Edit Tags` mở Modal Tagging.
- **Modal Tagging (`#modal-tagging`)**:
  - Quản lý danh mục Master Tags (`availableTagsMaster`).
  - Danh sách checkbox chọn nhanh tag có sẵn.
  - Ô nhập thêm tag mới (`#new-tag-input`) và nút `Add tags`.
- **Modal Transfer Ticket (`#modal-transfer-ticket`)**:
  - Bàn giao ca trực gồm: `Transfer to` (chọn CS từ danh sách `state.config.csTeam`), `Remind` (mốc thời gian nhắc nhở), `Status`, `Urgency`, `Note transfer case` (ghi chú ngắn hạn), và `Summary note` (tự động nạp sẵn từ DB).
  - Tự động phát hiện chuyển từ `Done - Need CS Check` về `Đang check - Dev` để tăng `reopenCount` và gán badge `Reopened`.

### 2.2 Out of Scope
- Chỉnh sửa nội dung Request gốc trực tiếp trên Web App (phải sửa trên bài đăng gốc Slack Thread).
- Xóa vĩnh viễn ticket khỏi database từ giao diện CS Agent (chỉ Admin mới có quyền lưu trữ/archive).

---

## 3. Key Business Rules

- **3.1. Lock Request Content Policy**: Trong Modal Edit Ticket, ô `Request Content` bắt buộc ở chế độ `readonly / disabled`. CS Agent không được phép chỉnh sửa trực tiếp nội dung này nhằm bảo toàn tính toàn vẹn và đồng bộ với nội dung yêu cầu gốc của khách hàng trên Slack Thread.
- **3.2. Shopify-style Dynamic Tagging**:
  - Mỗi ticket có thể gán 0 đến nhiều tags kỹ thuật (ví dụ: `Live preview`, `CSS Issue`, `Theme Conflict`, `Option Set`).
  - Khi xóa tag trên modal Edit hoặc Tagging Modal, mảng `tags` của ticket được cập nhật tức thì và render lại giao diện.
- **3.3. Dual Note Separation Principle**:
  - `Note transfer case`: Ghi chú nội bộ tạm thời phục vụ giao ca, được ghi đè sau mỗi lần Transfer thành công.
  - `Summary note`: Tóm tắt kỹ thuật cốt lõi lưu DB dài hạn. Khi mở Modal Transfer, hệ thống bắt buộc tự động nạp `ticket.summaryNote` vào ô textarea mà không bắt CS phải copy-paste thủ công.
- **3.4. Multi-Factor Real-time Search**: Ô tìm kiếm `#search-input` thực hiện lọc đồng thời trên 3 trường: `requestContent`, `storeUrl`, và `ticket.id` theo cơ chế không phân biệt hoa thường (`toLowerCase()`), kích hoạt ngay trên sự kiện `oninput`.
- **3.5. Automated Reopen Detection & Invariance**: Khi ticket có trạng thái trước đó là `Done - Need CS Check` (hoặc nhóm Resolved) và được cập nhật trạng thái mới là `Đang check - Dev` (hoặc nhóm Dev), hệ thống tự động:
  1. Tăng biến đếm `reopenCount = (ticket.reopenCount || 0) + 1`.
  2. Bật cờ hiển thị badge `Reopened` màu vàng trên thẻ ticket.
  3. Bắn Toast thông báo: `Ticket [ID] từ [Old] ➔ [New] (Reopened lần thứ N)! Badge Reopened đã tự động được gán.`
- **3.6. Plain Button & No-Emoji UI Rule**:
  - Toàn bộ các nút bấm hành động (`View Crisp`, `View Slack`, `Edit`, `Transfer`, `Store Info`) tuyệt đối không dùng icon emoji (`💬`, `✏️`, `🔄`, `🏪`).
  - Định dạng chuẩn: `px-3 py-1.5 rounded-lg text-xs font-medium bg-slate-50 hover:bg-slate-100 text-slate-700 border border-slate-200 hover:border-slate-300 transition-colors`.

---

## 4. Domain Model

### 4.1 Entities
- `Ticket`:
  - `id` (string): Mã định danh ticket duy nhất (e.g. `TK-4821`).
  - `app` (string): Mã App áp dụng (`APO`, `APB`, `ACS`).
  - `channel` (string): Tên kênh Slack ánh xạ (e.g. `#apo-paid-task`).
  - `storeUrl` (string): Domain cửa hàng Shopify (e.g. `woodesign.ie.myshopify.com`).
  - `status` (string): Trạng thái hiện tại trong 12 trạng thái custom.
  - `urgency` (string/number): Cấp độ ưu tiên (`Urgent`, `High`, `Medium`, `Low` hoặc 1, 2, 3, 4).
  - `feature` (string): Tên module/tính năng liên quan (e.g. `Live preview`).
  - `assignedTo` (string): Tên CS Agent phụ trách (e.g. `Aylin Tran`).
  - `timestamp` (string): Mốc thời gian tạo/giao ca.
  - `requestContent` (string): Nội dung yêu cầu kỹ thuật gốc.
  - `tags` (array<string>): Danh sách các nhãn phân loại kỹ thuật.
  - `handoffNote` (string): Ghi chú giao ca ngắn hạn (`Note transfer case`).
  - `summaryNote` (string): Ghi chú tóm tắt kỹ thuật cốt lõi (`Summary note`).
  - `reopenCount` (integer): Số lần ticket bị mở lại.
  - `unread` (boolean): Trạng thái chưa đọc đối với CS hiện tại.

### 4.2 Enums
- `UrgencyLevel`: `Urgent` (SLA 15m), `High` (SLA 1h), `Medium` (SLA 4h), `Low` (SLA 24h).
- `TicketStatusGroup`: `Dev Group`, `CS Group`, `Followup Group`, `Closed Group`.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `id` | string | Yes | None | Khóa chính dạng `TK-XXXX`, không được trùng lặp. |
| `app` | string | Yes | None | Phải trùng với 1 mã code trong `state.config.apps`. |
| `channel` | string | Yes | `#apo-paid-task` | Phải là tên kênh Slack hợp lệ bắt đầu bằng `#`. |
| `storeUrl` | string | Yes | None | Phải chứa đuôi `.myshopify.com` hoặc domain merchant hợp lệ. |
| `status` | string | Yes | `Đang check - Dev` | Phải thuộc danh sách trạng thái quy chuẩn trong `state.config.statuses`. |
| `urgency` | string | Yes | `Medium` | Thuộc một trong 4 giá trị: `Urgent`, `High`, `Medium`, `Low`. |
| `assignedTo` | string | Yes | `Aylin Tran` | Tên nhân sự thuộc `state.config.csTeam`. |
| `requestContent` | string | Yes | None | Chuỗi mô tả yêu cầu, tối thiểu 5 ký tự, bị khóa read-only trong modal Edit. |
| `tags` | array<string> | No | `[]` | Mảng chứa các chuỗi nhãn không trùng nhau. |
| `handoffNote` | string | No | `""` | Ghi chú ca trực, tối đa 500 ký tự. |
| `summaryNote` | string | No | `""` | Tóm tắt kỹ thuật DB, tối đa 1000 ký tự. |
| `reopenCount` | integer | No | `0` | Số nguyên không âm, tự động tăng khi chuyển từ Done sang Dev. |
| `unread` | boolean | No | `false` | Giá trị boolean biểu thị trạng thái chưa đọc. |

---

## 6. State Transitions

### 6.1 Allowed Transitions
- `Chờ tiếp nhận` ➔ `Đang check - Dev` ➔ `Dev đang fix` ➔ `Done - Need CS Check`
- `Done - Need CS Check` ➔ `Đang check - Dev` (Kích hoạt Tự động Reopen: `reopenCount++`)
- `Done - Need CS Check` ➔ `Khách đã xác nhận - Resolved` ➔ `Closed`
- `Chờ khách reply (Followup)` ➔ `Đang check - Dev` / `Khách đã xác nhận - Resolved`

### 6.2 Disallowed Transitions
- Không cho phép chuyển thẳng từ `Closed` sang `Dev đang fix` mà không qua bước `Reopen / Đang check - Dev`.
- Không cho phép cập nhật trạng thái nếu trường `assignedTo` bị để trống.

---

## 7. Runtime/behavior contract

- **Input**: Danh sách `state.tickets`, bộ lọc `activeAppFilter`, `activeAssigneeFilter`, giá trị `#search-input`, giá trị `#status-filter-select`.
- **Output**: Render danh sách thẻ ticket tương ứng vào `#tickets-container` thông qua hàm an toàn `safeSetHTML`.
- **Ordering**:
  1. Lọc theo App (`activeAppFilter === 'ALL'` hoặc `t.app === activeAppFilter`).
  2. Lọc theo Phân công (`activeAssigneeFilter === 'ALL'` hoặc `t.assignedTo === 'Aylin Tran'`).
  3. Lọc theo Status (`statusFilter === 'ALL'` hoặc `t.status === statusFilter`).
  4. Lọc theo Search Query (`matchSearch === true`).
- **Fallback**: Nếu kết quả lọc trả về rỗng (0 tickets), hiển thị thông báo rỗng an toàn mà không làm crash layout.

---

## 8. QA Scenarios

1. Kiểm tra 3 thẻ stat cards trên Dashboard hiển thị chính xác tổng số ticket mở, số ticket của Aylin và số ticket unread.
2. Kiểm tra click các nút lọc App (`ALL`, `APO`, `APB`, `ACS`) cập nhật class active màu xanh và hiển thị đúng tickets của từng app.
3. Kiểm tra click nút toggle `Gán cho tôi` chỉ hiển thị các ticket có `assignedTo === 'Aylin Tran'`.
4. Kiểm tra chọn từng trạng thái trong dropdown Status Filter lọc chính xác danh sách ticket.
5. Kiểm tra nhập từ khóa vào `#search-input` lọc real-time theo nội dung request, domain store và mã ticket.
6. Kiểm tra cấu trúc thẻ ticket hiển thị App badge ở đầu, tiếp đến Status badge, Urgency badge (không có chữ "High risk"), và Reopened badge nếu có.
7. Kiểm tra Ticket ID (`TK-XXXX`) hiển thị bên cạnh tên channel ở góc trên bên phải thẻ ticket.
8. Kiểm tra dòng Request Content hiển thị dạng text thuần không có khung viền xám bao quanh.
9. Kiểm tra danh sách action buttons (`View Crisp`, `View Slack`, `Edit`, `Transfer`, `Store Info`) là plain button không có emoji.
10. Kiểm tra thẻ ticket hiển thị đầy đủ 2 khung ghi chú vàng (`Note transfer case` và `Summary note`).
11. Kiểm tra mở Modal Edit Ticket: trường Request Content ở trạng thái disabled/read-only không cho sửa.
12. Kiểm tra thay đổi Status hoặc Urgency trong Modal Edit và bấm Lưu cập nhật thẻ ticket ngay lập tức.
13. Kiểm tra bấm nút `✕` trên tag pill trong Modal Edit gỡ tag ra khỏi ticket.
14. Kiểm tra mở Modal Tagging, tích chọn tag mới hoặc nhập tag mới và bấm `Add tags` cập nhật danh sách tag của ticket.
15. Kiểm tra mở Modal Transfer: trường `Summary note` tự động nạp sẵn dữ liệu tóm tắt từ DB.
16. Kiểm tra nhập `Note transfer case` mới và thực hiện Transfer: khung ghi chú ca trực được cập nhật nội dung mới.
17. Kiểm tra chuyển trạng thái ticket từ `Done - Need CS Check` về `Đang check - Dev` qua Modal Transfer: `reopenCount` tự động tăng lên 1 và badge `Reopened` xuất hiện.
18. Kiểm tra click nút `Store Info` trên thẻ ticket điều hướng sang view Shop 360° với domain tương ứng được nạp sẵn.

---

## 9. Implementation notes for AI code generation

- **DOM Identifier Preservation (P0 Critical)**: Bắt buộc giữ nguyên vẹn các ID cốt lõi: `tickets-container`, `stat-total-open`, `stat-assigned-me`, `stat-unread`, `dashboard-app-filter-buttons`, `status-filter-select`, `search-input`, `filter-me-btn`, `modal-edit-ticket`, `modal-transfer-ticket`, `modal-tagging`.
- **Defensive DOM Operations**: Toàn bộ thao tác gán `innerHTML` phải thông qua helper `safeSetHTML(elementId, htmlContent)` để ngăn ngừa runtime exceptions.
- **Tailwind Palette**: Sử dụng bảng màu Slate/Blue cho light mode: nền trắng `#ffffff`, viền nhạt `border-slate-200`, text chính `text-slate-800`, text phụ `text-slate-500`, accent `bg-blue-600`.

---

## 10. Final implementation assumptions to review

- Người dùng CS đang đăng nhập mặc định trong session mô phỏng là `Aylin Tran`.
- Phím tắt tìm kiếm thực hiện trên client-side in-memory state mà không cần gọi API debounce về backend.

---

## 11. User Stories

### Story: US-01 — CS lọc và tìm kiếm Ticket trên Dashboard
**As a** CS Agent (Aylin Tran)  
**I want to** lọc ticket theo App, theo trạng thái, theo phân công cá nhân và tìm kiếm real-time theo từ khóa  
**So that** tôi nhanh chóng định vị được các ticket cần xử lý trong ca trực của mình.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang ở giao diện CS Dashboard  
  **When** CS click vào nút `APO` trên thanh App Filter  
  **Then** Danh sách chỉ hiển thị các ticket có `app === 'APO'`  
  **And** Nút `APO` được kích hoạt giao diện active màu xanh đậm.

- **Given** CS đang xem danh sách tickets  
  **When** CS gõ `woodesign` vào ô `#search-input`  
  **Then** Danh sách tickets được lọc tức thì chỉ còn lại các ticket có chứa chuỗi `woodesign` trong domain hoặc nội dung request.

### Story: US-02 — CS chỉnh sửa Ticket và gán Tagging chuẩn Shopify
**As a** CS Agent  
**I want to** mở Modal Edit Ticket để cập nhật trạng thái, mức độ ưu tiên và gán tag kỹ thuật  
**So that** thông tin xử lý được phân loại chính xác mà không làm sai lệch nội dung request gốc của merchant.

#### Acceptance Criteria (Gherkin)
- **Given** CS click vào nút `Edit` trên ticket `TK-4821`  
  **When** Modal Edit Ticket hiển thị  
  **Then** Ô `Request Content` ở trạng thái disabled/read-only không thể chỉnh sửa text  
  **And** CS có thể thay đổi dropdown Status sang `Đang check - Dev` và click `+ Edit Tags` để mở Modal Tagging.

- **Given** CS đang mở Modal Tagging  
  **When** CS tích chọn tag `CSS Issue` và bấm `Add tags`  
  **Then** Tag `CSS Issue` được bổ sung vào danh sách tag của ticket và hiển thị trên thẻ ticket sau khi lưu.

### Story: US-03 — CS bàn giao ca trực và tự động kích hoạt Reopen
**As a** CS Agent sắp hết ca trực  
**I want to** mở Modal Transfer Ticket để chuyển ticket cho đồng nghiệp ca sau kèm ghi chú giao ca  
**So that** đồng nghiệp nắm rõ ngữ cảnh xử lý mà dữ liệu tóm tắt DB không bị đè mất.

#### Acceptance Criteria (Gherkin)
- **Given** Ticket `TK-4821` có trạng thái hiện tại là `Done - Need CS Check`  
  **When** CS bấm `Transfer`, chọn chuyển cho `Ngan Pham`, chọn trạng thái mới là `Đang check - Dev` và nhập `Note transfer case`  
  **Then** Hệ thống tăng `reopenCount` lên 1, gán badge `Reopened` cho ticket, và lưu `Note transfer case` mới vào khung ghi chú màu vàng.
