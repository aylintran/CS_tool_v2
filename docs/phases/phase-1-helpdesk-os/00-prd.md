---
phase_slug: phase-1-helpdesk-os
features:
  - crisp-plugin-add-ticket-form
  - crisp-plugin-transfer-ticket-form
  - crisp-plugin-store-info-subdomain
  - cs-dashboard-dual-note-cards
  - slack-oauth-thread-creation
  - shift-handover-workflow
  - shop-detail-360
  - app-store-reviews-tracking
  - duty-shift-management
  - cs-analytics-reporting
  - ticket-management-admin-dev
  - cs-workflow-guide-documentation
doc_type: prd
version: 5.7
status: draft
owner: "@po"
reviewers: ["@ba", "@tech-lead"]
created_at: 2026-08-19
last_updated: 2026-08-20
links:
  related_docs: []
depends_on: []
consumed_by:
  - ../../features/crisp-plugin-add-ticket-form/01-srs.md
  - ../../features/crisp-plugin-transfer-ticket-form/01-srs.md
  - ../../features/crisp-plugin-store-info-subdomain/01-srs.md
  - ../../features/cs-dashboard-dual-note-cards/01-srs.md
  - ../../features/slack-oauth-thread-creation/01-srs.md
  - ../../features/shift-handover-workflow/01-srs.md
  - ../../features/shop-detail-360/01-srs.md
  - ../../features/app-store-reviews-tracking/01-srs.md
  - ../../features/duty-shift-management/01-srs.md
  - ../../features/cs-analytics-reporting/01-srs.md
  - ../../features/ticket-management-admin-dev/01-srs.md
  - ../../features/cs-workflow-guide-documentation/01-srs.md
---

# PRD: Helpdesk OS — Comprehensive CS & Admin Suite (Phase 1 - v5.7 Final Spec)

## 1. Context & Architecture

PRD v5.7 bổ sung **Màn hình CS Workflow Guide (`📖 CS Workflow Guide`)** trực tiếp trong hệ thống Internal Tool giúp CS Agent dễ dàng truy cập, tra cứu và nắm vững toàn bộ quy trình vận hành Helpdesk OS:

1. **Sơ đồ luồng 4 bước (4-Step Operational Flow)**:
   - **Bước 1**: Tiếp nhận & check thông tin Store / Visitor Data trên Crisp Plugin.
   - **Bước 2**: Bấm `+ Add ticket` gửi Slack Thread dưới tên CS cá nhân (OAuth User Token).
   - **Bước 3**: Thảo luận **duy nhất trong Slack Thread** (Web App lắng nghe im lặng không phát push alert trùng lặp).
   - **Bước 4**: Bàn giao ca trực (`🔄 Transfer Ticket`) với 2 thẻ note vàng (`Note transfer case` ngắn hạn + `Summary note` auto-fill DB).
2. **Quy tắc 12 Trạng thái Custom**: Phân loại theo nhóm trách nhiệm (Dev / CS / Follow-up).
3. **Quy tắc 2 Khung Vàng Note**: Phân định rạch ròi giữa ghi chú giao ca trực và tóm tắt cốt lõi lưu DB.

---

## 2. Sub-features Summary Table

| Feature Slug | Short Scope | Role Access | SRS Path |
| :--- | :--- | :--- | :--- |
| `cs-workflow-guide-documentation` | Màn hình Hướng dẫn Quy trình CS Agent dạng Accordion (Sơ đồ 4 bước, Manual Follow-up Flow, 12 Status Rules, 2 Note Cards) | CS, Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/cs-workflow-guide-documentation/01-srs.md` |
| `dashboard-operations-cs` | CS Dashboard 3 thẻ đếm, filter App, Client-side Search, Thẻ ticket 2 cột với Edit Modal (LOCKED Request Content, Shopify Tagging Modal) | CS, Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/dashboard-operations-cs/01-srs.md` |
| `crisp-plugin-integration` | Crisp Sidebar Plugin View: Top Bar search & Add Ticket, Store Meta Header, Card actions (Edit, View Slack, Transfer), Footer CS Name, Slack OAuth User Token | CS, Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/crisp-plugin-integration/01-srs.md` |
| `shop-detail-360` | Shop 360° View: Search Domain / Subdomain (Parent Store resolution), Needs Action, Account Meta, CS Pinned Notes, Add Sub-domain | CS, Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/shop-detail-360/01-srs.md` |
| `app-store-reviews-tracking` | Theo dõi Review Shopify App Store, 3-Way Auto Linking (Review ➔ Domain ➔ Crisp Chat ID), Rating Breakdown & Complaint Themes | CS, Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/app-store-reviews-tracking/01-srs.md` |
| `duty-shift-management` | Quản lý Lịch Trực Ca CS & Dev (Mon-Sun), Publish Roster lên Slack, Bảng SLA Routing Rules (Urgent 15m, High 1h, Medium 4h, Low 24h) | CS Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/duty-shift-management/01-srs.md` |
| `cs-analytics-reporting` | Báo cáo Phân tích Vận hành CS: Date Range Filter, Created/Resolved/Resolution Time, Biểu đồ Status Bar, Urgency Donut, Volume, SLA Performance | CS Lead, Admin | `docs/phases/phase-1-helpdesk-os/features/cs-analytics-reporting/01-srs.md` |
| `ticket-management-admin-dev` | Màn hình Admin Settings 6 Tabs (Target Apps, Slack Channels Picker via Slack API, Status Workflow, Urgency Rules, Full CS Guide Editor with Live Preview, Slack Teams Sync) | Admin, Dev Lead | `docs/phases/phase-1-helpdesk-os/features/ticket-management-admin-dev/01-srs.md` |
