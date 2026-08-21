---
feature_slug: ticket-management-admin-dev
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

# SRS: Admin & System Settings Control Panel (`ticket-management-admin-dev`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **Admin & System Settings** trong Internal Web App thuộc Helpdesk OS dành cho vai trò Product Owner (PO), Dev Lead và Administrator. Module này cung cấp bảng điều khiển trung tâm 6 Tabs để cấu hình danh mục Target Apps, ánh xạ kênh Slack từ Workspace qua Picker Modal, quản lý danh sách Statuses và Urgency SLA, chỉnh sửa trực tiếp toàn bộ tài liệu CS Workflow Guide bằng Markdown với Live Preview hai cột, và đồng bộ danh sách nhân sự từ Slack User Groups (`@cs` và `@dev`).

---

## 2. Scope

### 2.1 In Scope
- **Admin Navigation Toolbar**:
  - Tiêu đề: `Admin & System Settings` kèm nhãn phân quyền `PO & Dev Lead Control`.
  - Nút lưu cấu hình: `Save All Settings`.
  - 6 Tab Switchers (`switchAdminTab(tab)`):
    1. `Target Apps` (`#adm-tab-apps` / `#admin-panel-apps`).
    2. `Slack Channels` (`#adm-tab-channels` / `#admin-panel-channels`).
    3. `Status Workflow` (`#adm-tab-statuses` / `#admin-panel-statuses`).
    4. `Urgency & SLA` (`#adm-tab-urgency` / `#admin-panel-urgency`).
    5. `CS Guide Markdown Editor` (`#adm-tab-guide-edit` / `#admin-panel-guide-edit`).
    6. `Slack Teams Sync` (`#adm-tab-teams` / `#admin-panel-teams`).
- **Chi Tiết Các Tab Quản Trị**:
  - **Tab 1 — Target Apps**:
    - Danh sách các ứng dụng mục tiêu (`#admin-apps-grid`): Tên App, Mã App Code ngắn, nút `Remove` gỡ app.
    - Nút `+ Add App` (`addNewTargetAppPrompt()`): Nhập App Code và Tên App để bổ sung vào `state.config.apps`.
    - Tự động cập nhật tức thì danh sách App tại Sidebar và các dropdown tạo ticket.
  - **Tab 2 — Slack Channels**:
    - Danh sách kênh đang ánh xạ (`#admin-channels-list`): Tên kênh (`#apo-paid-task`), mô tả, badge `Active Sync`.
    - Nút `+ Map Slack Workspace Channel` mở **Modal Slack Channel Picker** (`#modal-slack-channel-picker`):
      - Danh sách kênh từ Workspace: `#apo-paid-task`, `#apo-urgent-case`, `#acs-color-swatch-task`, `#apb-general-support`, `#cs-announcements`.
      - Đánh dấu `Already Mapped` và vô hiệu hóa radio button đối với các kênh đã được ánh xạ.
      - Nút `+ Map Channel` để xác nhận ánh xạ kênh mới vào hệ thống.
  - **Tab 3 — Status Workflow**:
    - Danh sách 12 trạng thái quy chuẩn (`#admin-statuses-list`): Mã trạng thái và nhóm phân loại (`Dev`, `CS`, `Followup`, `Closed`).
    - Nút `+ Add Status` (`addNewStatusPrompt()`): Bổ sung trạng thái mới vào danh mục.
  - **Tab 4 — Urgency & SLA**:
    - Bảng định nghĩa 4 cấp độ: Urgent (15m), High (1h), Medium (4h), Low (24h) với tiêu chí phân loại khách hàng và sự cố.
  - **Tab 5 — CS Guide Markdown Editor**:
    - Giao diện soạn thảo 2 cột (Split-pane Markdown Editor):
      - Cột trái: Source Code Editor (`#admin-guide-editor-text`) với font monospace, sự kiện `oninput="updateGuideLivePreview()"`.
      - Cột phải: Live Preview Result (`#admin-guide-editor-preview`) render HTML thời gian thực qua `marked.parse()`.
    - Nút `Reset Default` (`resetDefaultGuideContent()`): Khôi phục văn bản về bản chuẩn ban đầu.
    - Nút `Publish Updated Guide` (`publishGuideContent()`): Xuất bản bài hướng dẫn mới và đồng bộ ngay sang màn hình CS Workflow Guide.
  - **Tab 6 — Slack Teams Sync**:
    - Khung **CS Team (`#admin-cs-team-list`)**: Danh sách thành viên CS (`Aylin Tran`, `Ngan Pham`, `Ha Tran`, `Thao Vo`) và nút `Sync @cs` (`syncSlackGroup('cs')`).
    - Khung **Dev Team (`#admin-dev-team-list`)**: Danh sách thành viên Dev (`Tuan Nguyen`, `Minh Dao`, `Hung Le`, `Bao Nguyen`) và nút `Sync @dev` (`syncSlackGroup('dev')`).

### 2.2 Out of Scope
- Chỉnh sửa cấu hình hạ tầng server hosting database.

---

## 3. Key Business Rules

- **3.1. Synchronous App Propagation**: Khi một Target App mới được thêm qua Tab 1 hoặc gỡ bỏ:
  1. Mảng `state.config.apps` được cập nhật.
  2. Tự động kích hoạt `renderAdminPanels()` và `populateSelectDropdowns()`.
  3. Thanh lọc App Filter trên Dashboard và danh sách App ở Sidebar tự động cập nhật ngay lập tức.
- **3.2. Duplicate Channel Prevention**: Trong Modal Slack Channel Picker, các kênh đã tồn tại trong `state.config.slackChannels` bắt buộc ở trạng thái `disabled`, hiển thị nhãn `Already Mapped` để tránh ánh xạ trùng lặp.
- **3.3. Real-time Live Markdown Preview**: Mọi ký tự gõ vào `#admin-guide-editor-text` lập tức kích hoạt `updateGuideLivePreview()` để parse và hiển thị trên `#admin-guide-editor-preview` với độ trễ < 50ms.
- **3.4. Slack User Group Sync & Transfer Recipient Routing**: Nút `Sync @cs` và `Sync @dev` cập nhật mảng nhân sự tương ứng. Danh sách `transfer-to-select` trong Modal Transfer Ticket hỗ trợ:
  - Chọn tài khoản CS theo `@username` kèm tên đầy đủ và tag Slack cá nhân (VD: `@aylin — Aylin Tran (@cs-aylin)`).
  - Tùy chọn đặc biệt **`CS online`**: Tự động gán ticket cho nhân sự CS trực ca vào đúng khung giờ nhắc nhở (`Remind time`).
- **3.5. Plain UI & No-Emoji Policy**: Toàn bộ các nút bấm và nhãn trong Admin Settings tuân thủ thiết kế plain text, không dùng icon emoji.

---

## 4. Domain Model

### 4.1 Entities
- `AppConfig`:
  - `code` (string): Mã ngắn (e.g. `APO`, `APB`, `ACS`).
  - `name` (string): Tên đầy đủ của ứng dụng.
- `SlackChannelMapping`:
  - `code` (string): Tên kênh (e.g. `#apo-paid-task`).
  - `desc` (string): Mục đích sử dụng của kênh.
- `CustomStatus`:
  - `code` (string): Tên trạng thái (e.g. `Đang check - Dev`).
  - `group` (string): Nhóm phân loại (`Dev`, `CS`, `Followup`, `Closed`).

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `app_code` | string | Yes | None | 2 - 6 ký tự in hoa, không trùng lặp. |
| `app_name` | string | Yes | None | Tên đầy đủ của app, tối thiểu 3 ký tự. |
| `channel_code` | string | Yes | None | Bắt đầu bằng ký tự `#`, không chứa khoảng trắng. |
| `status_code` | string | Yes | None | Tên trạng thái duy nhất. |

---

## 6. State Transitions

- **Chuyển Tab Admin**:
  - `switchAdminTab(tab)`: Ẩn các panel khác, hiển thị panel tương ứng, cập nhật class active màu tím cho tab button.
- **Thêm App / Kênh / Status**:
  - `Tạo mới ➔ Cập nhật state.config ➔ Re-render panels và dropdowns ➔ Toast thông báo`.

---

## 7. Runtime/behavior contract

- **Input**: User actions trên 6 tab admin settings.
- **Output**: Cập nhật `state.config` và gọi `renderAdminPanels()` thông qua `safeSetHTML`.
- **Fallback**: Luôn kiểm tra tồn tại của các element DOM trước khi gán nội dung.

---

## 8. QA Scenarios

1. Kiểm tra chuyển đổi qua lại giữa 6 Tabs (Apps, Channels, Statuses, Urgency, Guide Edit, Teams) hiển thị đúng panel tương ứng.
2. Kiểm tra Tab Target Apps: bấm `+ Add App`, nhập mã `APX` và tên `Avis Pixels`: App mới xuất hiện trong danh sách admin, ở Sidebar và trên thanh App Filter.
3. Kiểm tra bấm nút `Remove` trên một Target App: App đó biến mất khỏi danh sách và các dropdown.
4. Kiểm tra Tab Slack Channels: bấm `+ Map Slack Workspace Channel` mở Modal Slack Channel Picker.
5. Kiểm tra các kênh đã được ánh xạ hiển thị nhãn `Already Mapped` và không cho chọn lại.
6. Kiểm tra chọn 1 kênh có sẵn và bấm `+ Map Channel`: kênh mới xuất hiện trong danh sách và dropdown tạo ticket.
7. Kiểm tra Tab Status Workflow: bấm `+ Add Status` thêm trạng thái mới thành công.
8. Kiểm tra Tab Urgency hiển thị đầy đủ thông tin 4 mức độ SLA (15m, 1h, 4h, 24h).
9. Kiểm tra Tab Guide Edit: mở tab tự động nạp mã nguồn Markdown vào ô bên trái và render Live Preview bên phải.
10. Kiểm tra gõ Markdown mới vào ô soạn thảo: Live Preview bên phải cập nhật tức thì.
11. Kiểm tra bấm `Publish Updated Guide` cập nhật nội dung sang màn hình CS Workflow Guide và bắn Toast thông báo.
12. Kiểm tra bấm `Reset Default` khôi phục nội dung bài hướng dẫn về bản chuẩn ban đầu.
13. Kiểm tra Tab Slack Teams Sync: bấm `Sync @cs` và `Sync @dev` hiển thị Toast thông báo đồng bộ thành công.
14. Kiểm tra toàn bộ các nút trong Admin Settings không chứa icon emoji.
15. Kiểm tra giao diện Light Mode hiển thị sắc nét, viền nhạt tinh tế.

---

## 9. Implementation notes for AI code generation

- **Preserved DOM Identifiers (P0 Critical)**: `admin-apps-grid`, `admin-channels-list`, `admin-statuses-list`, `admin-cs-team-list`, `admin-dev-team-list`, `admin-guide-editor-text`, `admin-guide-editor-preview`, `modal-slack-channel-picker`, `slack-workspace-channels-list`.
- **Decoupled Architecture**: Tuyệt đối không thay đổi ID hoặc cấu trúc thẻ HTML trong quá trình style giao diện.

---

## 10. Final implementation assumptions to review

- Chỉ tài khoản có vai trò PO / Dev Lead / Admin mới được cấp quyền truy cập Admin Settings.

---

## 11. User Stories

### Story: US-01 — PO / Dev Lead cấu hình Target Apps và chọn kênh Slack từ Workspace
**As a** Product Owner  
**I want to** thêm Target App mới và ánh xạ Slack Channel từ danh sách kênh của workspace  
**So that** hệ thống tự động cập nhật bộ lọc và điều hướng ticket đến đúng kênh Slack của đội ngũ phát triển.

#### Acceptance Criteria (Gherkin)
- **Given** PO đang ở Tab Target Apps trong Admin Settings  
  **When** PO thêm App mới với mã `APX` và tên `Avis Pixels`  
  **Then** App `APX` tự động xuất hiện tại Sidebar, trên thanh lọc App Dashboard và trong dropdown tạo ticket.

- **Given** PO đang ở Tab Slack Channels  
  **When** PO bấm `+ Map Slack Workspace Channel` và chọn kênh `#apb-general-support`  
  **Then** Kênh `#apb-general-support` được thêm vào danh sách quản lý với nhãn `Active Sync`.

### Story: US-02 — PO soạn thảo và xuất bản CS Workflow Guide bằng Markdown
**As a** Product Owner  
**I want to** soạn thảo nội dung quy trình CS bằng Markdown có Live Preview và bấm xuất bản  
**So that** toàn bộ tài liệu hướng dẫn trên màn hình CS Workflow Guide của CS Agent được cập nhật ngay lập tức.

#### Acceptance Criteria (Gherkin)
- **Given** PO đang ở Tab CS Guide Edit trong Admin Settings  
  **When** PO chỉnh sửa nội dung văn bản Markdown ở khung soạn thảo bên trái  
  **Then** Khung Live Preview bên phải cập nhật định dạng hiển thị tức thì  
  **When** PO bấm nút `Publish Updated Guide`  
  **Then** Hệ thống lưu tài liệu mới, đồng bộ sang màn hình CS Workflow Guide và hiển thị Toast thông báo.
