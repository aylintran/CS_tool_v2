---
phase_slug: phase-1-helpdesk-os
features:
  - dashboard-operations-cs
  - crisp-plugin-integration
  - shop-detail-360
  - app-store-reviews-tracking
  - duty-shift-management
  - cs-analytics-reporting
  - cs-workflow-guide-documentation
  - ticket-management-admin-dev
doc_type: prd
version: 6.0
status: approved
owner: "@po"
reviewers: ["@ba", "@tech-lead"]
created_at: 2026-08-19
last_updated: 2026-08-21
links:
  related_docs: []
depends_on: []
consumed_by:
  - ../../features/dashboard-operations-cs/01-srs.md
  - ../../features/crisp-plugin-integration/01-srs.md
  - ../../features/shop-detail-360/01-srs.md
  - ../../features/app-store-reviews-tracking/01-srs.md
  - ../../features/duty-shift-management/01-srs.md
  - ../../features/cs-analytics-reporting/01-srs.md
  - ../../features/cs-workflow-guide-documentation/01-srs.md
  - ../../features/ticket-management-admin-dev/01-srs.md
---

# PRD: Helpdesk OS — Comprehensive CS & Admin Suite (Phase 1)

## 1. Context & problem

- **Bối cảnh**: Hệ sinh thái Shopify Apps (Avis Product Options, App Bundles, Aris Color Swatch) xử lý khối lượng lớn yêu cầu hỗ trợ kỹ thuật từ merchant qua Crisp Chat và điều phối xử lý với Dev team qua Slack Workspace.
- **Vấn đề cốt lõi**:
  1. **Đứt gãy thông tin giao ca trực (Shift Handover Gap)**: CS hết ca không thể bàn giao ngữ cảnh kỹ thuật ngắn hạn mà không làm bẩn dữ liệu tóm tắt lưu trữ dài hạn trong cơ sở dữ liệu.
  2. **Nhiễu kênh Slack & Phân tán công cụ**: CS phải nhảy qua lại giữa Crisp Chat, Slack channels và Google Sheets để tra cứu thông tin cửa hàng, lịch trực và tiến độ xử lý ticket.
  3. **Thiếu cơ chế liên kết 3 chiều (3-Way Auto Linking)**: Khi merchant khiếu nại review tiêu cực 1-2 sao trên Shopify App Store, đội CS mất nhiều thời gian tra cứu domain và không mở được đúng hội thoại Crisp Chat của khách hàng.
  4. **Quản trị cấu hình phân tán**: PO / Dev Lead không thể tự cấu hình danh sách App, đồng bộ nhóm Slack (`@cs`, `@dev`), chọn kênh Slack trực tiếp từ workspace Slack API, hoặc chỉnh sửa trực tiếp tài liệu Hướng dẫn CS Workflow Guide dạng Markdown.
- **Lý do giải quyết ngay**: Cần một hệ thống vận hành tập trung (Helpdesk OS) đồng bộ 3 nền tảng (Crisp Plugin, Slack Thread, Internal Web App Suite) với thiết kế Light Mode chuyên nghiệp, loại bỏ icon rác, đảm bảo hiệu suất phản hồi dưới 15 phút cho ticket Urgent.

---

## 2. Goals

### Business goals
- Giảm First Response Time (FRT) của các ticket Urgent (High risk) xuống dưới 15 phút.
- Tăng tỷ lệ giải quyết đúng hạn (SLA Resolution Rate) lên trên 95% cho toàn bộ 4 cấp độ Urgency.
- Tự động hóa 100% việc tạo Slack Thread dưới danh nghĩa cá nhân của CS Agent (thông qua Slack OAuth User Token).
- Cắt giảm 100% thời gian tạo báo cáo giao ca thủ công nhờ cơ chế Dual Note Cards (Note transfer case + Summary note).

### User goals
- **CS Agent (Aylin Tran, Ngan Pham, ...)**: Thao tác tạo ticket, tra cứu metadata cửa hàng (Shop 360°), gắn tag kỹ thuật, và giao ca chỉ trong vài cú click ngay trên Crisp Sidebar Plugin hoặc Web App.
- **Dev Team**: Nhận yêu cầu kỹ thuật chuẩn hóa trên đúng Slack Channel của từng App (`#apo-paid-task`, `#apo-urgent-case`, `#acs-color-swatch-task`, `#apb-general-support`) kèm đầy đủ domain, mô tả, tag và link đối soát.
- **PO / Dev Lead**: Tự chủ quản trị Target Apps, thêm/sửa Statuses, chọn ánh xạ Slack Workspace Channels, chỉnh sửa Markdown CS Workflow Guide với Live Preview tức thì, và đồng bộ nhân sự từ Slack User Groups (`@cs`, `@dev`).

---

## 3. User personas & use cases

### Primary persona
- **Aylin Tran — Customer Support Specialist**: Người trực tiếp trả lời livechat merchant trên Crisp, tạo ticket kỹ thuật gửi Dev, theo dõi trạng thái và bàn giao ca trực khi kết thúc giờ làm.
- **Minh Dao / Tuan Nguyen — Dev Specialist**: Kỹ sư tiếp nhận ticket trên Slack Thread, xử lý lỗi CSS / Option Set / Theme Conflict và cập nhật trạng thái về `Done - Need CS Check`.
- **Product Owner / Dev Lead**: Người quản trị toàn hệ thống, cấu hình SLA, chỉnh sửa tài liệu quy trình vận hành và kiểm tra báo cáo CS Analytics.

### Top use cases
1. **Tiếp nhận & Tạo ticket từ Crisp Plugin**: Khách chat trên Crisp ➔ CS xem Visitor Data ➔ Bấm `+ Add ticket` ➔ Chọn App, Channel, Status, Urgency, Tag ➔ Tự động bắn Slack Thread.
2. **Bàn giao ca trực không gián đoạn (Transfer Ticket)**: CS hết ca ➔ Bấm `Transfer` ➔ Chọn CS nhận ca ➔ Nhập `Note transfer case` ngắn hạn + hệ thống tự load `Summary note` cốt lõi từ DB ➔ Chuyển ca mượt mà.
3. **Tra cứu toàn diện Shop 360° View**: Nhập domain cửa hàng ➔ Xem trạng thái cảnh báo (Needs Attention, No Feedback Loop), lịch sử ticket, thông tin gói cước và Pinned Notes.
4. **Xử lý khiếu nại từ App Store Review**: Xem review tiêu cực ➔ Hệ thống liên kết tự động Review ➔ Domain ➔ Crisp Chat ID ➔ Bấm `+ Create Ticket` để mở ticket xử lý ngay.
5. **Chỉnh sửa tài liệu CS Workflow Guide bằng Markdown**: PO mở Admin Settings ➔ Soạn thảo nội dung Markdown bên trái ➔ Xem Live Preview bên phải ➔ Bấm `Publish Updated Guide` để cập nhật toàn hệ thống.

---

## 4. Scope

### In scope
- **Kiến trúc mô phỏng 3 nền tảng (3-Platform Switcher)**:
  1. `1. Crisp Chat + Plugin`: Mô phỏng chat merchant + Helpdesk OS Sidebar Plugin (Tab Tickets, Tab Store info, Sub-domain management, Visitor Data).
  2. `2. Slack Thread Sync`: Mô phỏng bài đăng Slack Thread tự động tạo qua Slack OAuth User Token.
  3. `3. Internal Web App`: Bộ công cụ quản trị nội bộ đầy đủ với 7 views chuyên biệt.
- **7 Internal Views trên Web App**:
  1. `CS Dashboard`: 3 thẻ đếm (Ticket mở trong ca, Ticket gán cho tôi, Ticket unread), Bộ lọc App (`ALL`, `APO`, `APB`, `ACS`), Toggle filter `Gán cho tôi`, Dropdown Status, Live Search, Ticket Card 2 cột với Dual Note Cards (`Note transfer case` & `Summary note`), Cụm plain buttons (`View Crisp`, `View Slack`, `Edit`, `Transfer`, `Store Info`).
  2. `Shop 360° View`: Tra cứu domain/sub-domain, Thẻ cảnh báo tài khoản (Needs Attention / No Feedback Loop), Account Meta (Shop ID, Email, Owner, Plan, Country, App Version), CS Pinned Notes, Danh sách ticket riêng của store.
  3. `App Store Reviews Tracking`: 3-Way Auto Linking (Review ➔ Domain ➔ Crisp Chat ID), Thống kê rating & complaint themes, Nút `Create Ticket` auto-populate domain và nội dung review.
  4. `Duty Shift Roster`: Lịch trực tuần Mon-Sun cho CS & Dev, nút xuất bản lên `#cs-announcements`, Bảng quy tắc SLA Routing Rules (Urgent 15m, High 1h, Medium 4h, Low 24h).
  5. `CS Analytics & Reporting`: KPI Cards (Total Volume, Resolved, First Response Time, Resolution Time), Phân bố trạng thái, Phân bố Urgency, Bảng hiệu suất CS Agent.
  6. `CS Workflow Guide`: Trang đọc tài liệu hướng dẫn quy trình CS 4 bước, 4 cấp Urgency, 12 Custom Statuses, Quy tắc Dual Note Cards dạng Markdown rendered.
  7. `Admin & System Settings`: 6 Tabs quản trị (Target Apps, Slack Channels Picker, Status Workflow, Urgency & SLA, CS Guide Markdown Editor with Live Preview, Slack Teams Sync).
- **Hệ thống Modals & Dynamic Tagging**:
  - Modal Add Ticket, Modal Transfer Ticket (với logic Auto Reopened), Modal Edit Ticket (Khóa locked Request Content), Modal Tagging (Quản lý Master Tags và gán tag vào ticket), Modal Slack Channel Picker.
- **Nguyên tắc giao diện & An toàn Code**:
  - Giao diện Light Mode nền sáng (`#f8fafc`), thẻ nền trắng (`#ffffff`), chữ đậm rõ nét (`#0f172a`), viền nhạt (`#e2e8f0`).
  - Toàn bộ button định dạng plain button không kèm icon emoji.
  - Toàn bộ badge trạng thái/urgency không chứa emoji, chỉ giữ màu nền/viền chuẩn semantic.
  - Bảo toàn 100% 105 DOM IDs, áp dụng helper an toàn `safeSetHTML` ngăn ngừa unhandled null pointer crashes.

### Out of scope
- Tích hợp thanh toán trực tiếp Shopify Billing API (chỉ quản lý hiển thị Plan Meta).
- Nhận diện giọng nói hoặc gọi thoại VoIP trên Crisp (chỉ hỗ trợ Livechat text).
- AI Chatbot tự động trả lời khách hàng mà không qua CS kiểm duyệt (Phase 2).

---

## 5. Constraints & assumptions

### Constraints
- **Client-Side Vanilla Architecture**: Toàn bộ hệ thống demo vận hành độc lập, tải mượt mà trên single HTML file mà không phụ thuộc vào heavy node backend phức tạp.
- **DOM & JS Safety (P0 Critical)**: Bất biến cấu trúc DOM IDs (`admin-apps-grid`, `dashboard-app-filter-buttons`, `shop-tickets-container`,...) để tránh crash chuỗi `window.onload`.
- **Light Mode Pure Aesthetics**: Tuân thủ bảng màu Slate/Blue/Purple tinh tế, không dùng màu chói hoặc hardcode mã màu phá vỡ Semantic Design System.

### Assumptions to verify
- Tích hợp Slack Webhook / Slack OAuth User Token cho phép đăng thread dưới danh tính cá nhân của CS Agent đã phân công.
- Crisp Plugin SDK hỗ trợ nạp iframe và đọc live Visitor Meta từ session của merchant đang chat.

---

## 6. Open questions

- [x] Bỏ chữ "(High risk)" ở badge Urgent: **Đã hoàn thành** (chỉ còn `Urgent`).
- [x] Đưa badge App lên đầu thẻ ticket và chuyển mã ticket sang cạnh channel: **Đã hoàn thành**.
- [x] Bỏ khung xám bao quanh Request Content và hiển thị dạng text liền mạch: **Đã hoàn thành**.
- [x] Xóa toàn bộ emoji icons trên toàn bộ giao diện, chuyển sang Plain Button tinh tế: **Đã hoàn thành**.
- [x] Bỏ dòng Rating trong Account Info của Shop 360°: **Đã hoàn thành**.

---

## 7. Sub-features

| Feature Slug | Short Scope | SRS Path | Tech Design Path |
| :--- | :--- | :--- | :--- |
| `dashboard-operations-cs` | CS Dashboard 3 thẻ đếm, filter App/Assignee, Search, Ticket card 2 cột với Dual Note Cards, Plain action buttons, Edit Modal (Locked Request), Dynamic Tagging Modal | `docs/phases/phase-1-helpdesk-os/features/dashboard-operations-cs/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/dashboard-operations-cs/03-tech-design.md` |
| `crisp-plugin-integration` | Crisp Livechat Simulator + Sidebar Plugin View (Tickets Tab, Store info Tab, Sub-domain management, Visitor Data Meta Table, Add Ticket modal launcher) | `docs/phases/phase-1-helpdesk-os/features/crisp-plugin-integration/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/crisp-plugin-integration/03-tech-design.md` |
| `shop-detail-360` | Shop 360° View: Domain/Subdomain search, Needs Attention alerts, Account Meta (No Rating), CS Pinned Notes, Store-specific Ticket List | `docs/phases/phase-1-helpdesk-os/features/shop-detail-360/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/shop-detail-360/03-tech-design.md` |
| `app-store-reviews-tracking` | Theo dõi Shopify App Store Reviews, 3-Way Auto Linking (Review ➔ Domain ➔ Crisp Chat ID), Rating & Complaint Theme breakdown, Create Ticket from Review | `docs/phases/phase-1-helpdesk-os/features/app-store-reviews-tracking/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/app-store-reviews-tracking/03-tech-design.md` |
| `duty-shift-management` | Quản lý Lịch trực CS & Dev Team (Mon-Sun), Nút Publish lên Slack `#cs-announcements`, Bảng quy tắc SLA Routing Rules (Urgent 15m, High 1h, Medium 4h, Low 24h) | `docs/phases/phase-1-helpdesk-os/features/duty-shift-management/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/duty-shift-management/03-tech-design.md` |
| `cs-analytics-reporting` | Báo cáo Phân tích Vận hành CS: Date Range, Volume, Resolved, First Response Time, Resolution Time, Biểu đồ Status/Urgency, Bảng hiệu suất CS Agent | `docs/phases/phase-1-helpdesk-os/features/cs-analytics-reporting/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/cs-analytics-reporting/03-tech-design.md` |
| `cs-workflow-guide-documentation` | Màn hình Hướng dẫn Quy trình CS Agent (Sơ đồ 4 bước, 4 cấp Urgency, 12 Trạng thái Custom, Quy tắc Dual Note Cards) render động từ Markdown | `docs/phases/phase-1-helpdesk-os/features/cs-workflow-guide-documentation/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/cs-workflow-guide-documentation/03-tech-design.md` |
| `ticket-management-admin-dev` | Admin Settings 6 Tabs (Target Apps, Slack Channels Picker Modal, Status Workflow, Urgency & SLA, Markdown Guide Editor with Live Preview, Slack Teams Sync) | `docs/phases/phase-1-helpdesk-os/features/ticket-management-admin-dev/01-srs.md` | `docs/phases/phase-1-helpdesk-os/features/ticket-management-admin-dev/03-tech-design.md` |
