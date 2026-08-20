---
feature_slug: ticket-management-admin-dev
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

# SRS: Admin & System Settings Control Panel (`ticket-management-admin-dev`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **Admin & System Settings (`⚙️ Admin Settings`)** dành riêng cho vai trò `admin` và `dev_lead` (PO / Lead). Màn hình bao gồm 6 Tab quản trị toàn diện hệ thống Helpdesk OS: Target Apps, Slack Channels Picker (kết nối Slack Web API), Status Workflow (12 custom statuses), Urgency Rules, Full CS Guide Markdown Editor với Live Preview, và Slack Teams Sync (`@cs` & `@dev`).

---

## 2. Scope

### 2.1 In Scope
- **Gate Phân Quyền (Role Access Control)**:
  - Chỉ cho phép tài khoản có role `admin` hoặc `dev_lead` truy cập.
  - Từ chối tài khoản role `cs_agent` với thông báo `"Access Denied — Requires Admin/Dev Lead privileges"`.
- **6 Tab Quản Trị Cấu Hình**:
  - **Tab 1 — Target Apps**: Danh sách Target Apps (`APO`, `APB`, `ACS`) + Form `+ Add Target App` (App Code, Full Name, Icon emoji). Tự động đồng bộ lên Sidebar và Dropdown tạo ticket.
  - **Tab 2 — Slack Channels Picker**: Chọn channel từ workspace Slack thực tế qua Slack API `GET /conversations.list`. Đánh dấu `Already Mapped` cho các kênh đã dùng.
  - **Tab 3 — Status Workflow**: Quản lý danh sách 12 trạng thái custom + Nút `+ Add Custom Status` phân loại theo nhóm Dev / CS / Follow-up.
  - **Tab 4 — Urgency Rules**: Hiển thị quy tắc 4 mức độ khẩn cấp (`🔴 Urgent` 15m, `🟠 High` 1h, `🔵 Medium` 4h, `⚪ Low` 24h).
  - **Tab 5 — Full CS Guide Editor**: Trình soạn thảo Markdown 2 cột:
    - Cột trái: Source Editor (`<textarea>` font monospace).
    - Cột phải: Live Preview Panel cập nhật tức thì.
    - Nút `Publish Updated Guide` lưu thay đổi tới DB (`PATCH /api/config/guide-content`).
  - **Tab 6 — Slack Teams Sync**: Nút `🔄 Sync @cs` và `🔄 Sync @dev` gọi Slack API `usergroups.users.list` để đồng bộ danh sách thành viên nhóm CS & Dev.

### 2.2 Out of Scope
- Quản lý hạ tầng máy chủ database PostgreSQL / Redis.

---

## 3. Key Business Rules

- **3.1. Role Gate**: Màn hình Admin Settings kiểm tra strict role. Nếu không phải `admin` hoặc `dev_lead`, toàn bộ giao diện bị khóa.
- **3.2. Live Guide Synchronization**: Khi Admin/PO chỉnh sửa văn bản hướng dẫn ở Tab 5 và bấm `Publish Updated Guide`, màn hình `📖 CS Workflow Guide` của toàn bộ CS Agent lập tức cập nhật nội dung mới.
- **3.3. Slack Integration API**: Tab 2 gọi trực tiếp `GET /conversations.list` của Slack API; Tab 6 gọi `usergroups.users.list` để tự động load danh sách kênh và nhân sự mà không cần gõ tay.

---

## 4. QA Scenarios

1. Kiểm tra tài khoản CS Agent bình thường bị từ chối truy cập Admin Settings với màn hình Access Denied.
2. Kiểm tra tài khoản Admin / Dev Lead truy cập thành công thấy 6 Tab cấu hình.
3. Kiểm tra Tab 1 thêm Target App mới xuất hiện ngay ở Sidebar và Add Ticket Modal.
4. Kiểm tra Tab 2 hiển thị danh sách Slack channels từ Slack Web API.
5. Kiểm tra Tab 5 nhập Markdown ở ô bên trái lập tức render định dạng ở Live Preview bên phải.
6. Kiểm tra bấm `Publish Updated Guide` cập nhật nội dung mới sang màn hình CS Workflow Guide.
7. Kiểm tra Tab 6 bấm Sync `@cs` đồng bộ danh sách thành viên nhóm CS từ Slack.
