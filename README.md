# CS Tool v2 — Helpdesk OS Suite

Hệ thống quản lý vận hành Hỗ trợ khách hàng (CS Operations & Helpdesk OS) tập trung dành cho đội ngũ CS, Dev & Lead. Hệ thống tích hợp trực tiếp giữa **Crisp Chat**, **Slack Thread OAuth**, **Shopify App Store Reviews** và **Shop 360° Comprehensive View**.

---

## 📌 Tổng Quan Kiến Trúc & Quy Chuẩn (Architecture & SDS Rules)

Dự án tuân thủ nghiêm ngặt mô hình Semantic Design System (SDS) chuẩn:

- **Theme (Primitives)**: Chứa các mã màu cơ bản (e.g. `Brand-100`). Bộ default là `Value`, các custom themes gồm `Mode`, `Morning Dew`, `Ash Blue`, `Emerald Waltz`... Kích hoạt qua attribute `[data-theme="..."]`.
- **Mode (Semantics)**: Chứa logic giao diện (e.g. `Background-Primary`). Gồm 2 mode `SDS Light` và `SDS Dark`, kích hoạt qua attribute `[data-mode="light|dark"]`.
- **Nguyên Tắc Tối Thượng**:
  - **Không Hardcode mã HEX** (`#FFFFFF`, `#2C2C2C`) vào giao diện hay CSS.
  - **Bắt buộc dùng biến SDS**: Sử dụng hệ thống biến `var(--sds-color-...)` (ví dụ `var(--sds-color-background-default-default)`).

---

## 🚀 Danh Sách Features — Phase 1: Helpdesk OS

Hệ thống tài liệu thiết kế phần mềm (SRS) thuộc Phase 1 được tổ chức cấu trúc chuẩn tại `docs/phases/phase-1-helpdesk-os/`:

| Feature Slug | Mô Tả Phạm Vi | SRS Path |
| :--- | :--- | :--- |
| **`dashboard-operations-cs`** | CS Dashboard với 3 thẻ đếm (`Cần xử lý`, `Đang check`, `Follow-up`), Bộ lọc App (`APO`, `APB`, `ACS`), Thẻ Ticket 2 cột, Modal Edit (khóa Request Content, Tagging kiểu Shopify), Modal Transfer Ticket. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/dashboard-operations-cs/01-srs.md) |
| **`crisp-plugin-integration`** | Sidebar Plugin tích hợp Crisp Chat: Top Bar search, Store Meta Header, Card Actions (`Edit`, `View Slack`, `Transfer`), Cảnh báo lỗi kết nối Slack kèm nút `Resend`. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/crisp-plugin-integration/01-srs.md) |
| **`shop-detail-360`** | Màn hình Shop 360° View: Ô Search Domain (quy đổi Subdomain ➔ Parent Store), Account Info, Needs Action (`URGENT`, `REVIEWS < 3★`), CS Pinned Notes, **Add Sub-domain**, Thẻ ticket 2 cột đầy đủ. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/shop-detail-360/01-srs.md) |
| **`cs-workflow-guide-documentation`** | Màn hình Hướng dẫn Quy trình CS (`📖 CS Workflow Guide`) dạng Accordion: Sơ đồ luồng 4 bước, Quy trình Follow-up thủ công, 12 Trạng thái Custom, 2 Thẻ ghi chú vàng. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/cs-workflow-guide-documentation/01-srs.md) |
| **`app-store-reviews-tracking`** | Màn hình Theo dõi Review Shopify App Store: Rating breakdown, Top complaint themes, 3-way auto linking (Review ➔ Domain ➔ Crisp Chat ID), Nút `Create ticket`, `Crisp`, `Store Info`. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/app-store-reviews-tracking/01-srs.md) |
| **`duty-shift-management`** | Màn hình Quản lý Lịch Ca Trực CS & Dev (Mon-Sun), Nút **Publish Roster to Slack**, Bảng SLA Routing Rules (🔴 Urgent 15m, 🟠 High 1h, 🔵 Medium 4h, ⚪ Low 24h). | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/duty-shift-management/01-srs.md) |
| **`cs-analytics-reporting`** | Báo cáo Phân tích Vận hành CS: Date Range Filter (7d, 14d, 30d), Created/Resolved/Resolution Time, Escalation Rate, Biểu đồ Tickets by Status, SLA Performance. | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/cs-analytics-reporting/01-srs.md) |
| **`ticket-management-admin-dev`** | Màn hình Admin Settings (`⚙️ Admin Settings`) 6 Tabs: Target Apps, Slack Channels Picker via Slack Web API `conversations.list`, Status Workflow, Urgency Rules, Full CS Guide Editor (Live Preview), Slack Teams Sync (`@cs` & `@dev`). | [`01-srs.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/docs/phases/phase-1-helpdesk-os/features/ticket-management-admin-dev/01-srs.md) |

---

## 🔄 Quy Trình Vận Hành CS (CS Operational Workflow)

### 1. Sơ Đồ Luồng 4 Bước (4-Step Operational Flow)
1. **Bước 1**: Tiếp nhận Crisp Chat ➔ Tra cứu thông tin Store / Visitor Data trên Crisp Plugin.
2. **Bước 2**: Tạo Ticket (`+ Add ticket`) ➔ Đẩy bài lên Slack Thread đúng channel dưới tên cá nhân CS (Slack OAuth User Token).
3. **Bước 3**: Thảo luận **duy nhất trong Slack Thread** (Web App lắng nghe im lặng, không gửi push alert gây nhiễu).
4. **Bước 4**: Bàn giao ca trực (`🔄 Transfer Ticket`) với 2 thẻ note vàng (`Note transfer case` ngắn hạn + `Summary note` lưu DB).

### 2. Quy Trình Follow-up Thủ Công (Manual Follow-up Flow)
1. Dev xử lý xong và báo **Request Done** trên Slack Thread.
2. CS tiến hành **Resolve cuộc chat** trên Crisp với Merchant.
3. CS chuyển trạng thái Ticket sang **`Fl up 1 (12h)`** thủ công (`manually`).
4. Theo dõi và lần lượt chuyển sang **`Fl up 2 (24h)`** ➔ **`Fl up 3 (36h)`** thủ công (`manually`).
5. Sau khi kết thúc `Fl up 3 (36h)` (hoặc khách xác nhận đã OK), CS chuyển ticket sang trạng thái **`Done - CS`** (Resolve ticket).

### 3. Quy Tắc 12 Trạng Thái Custom (12 Custom Statuses)
- **Nhóm Dev**: `Chờ check - Dev`, `Đang check - Dev`, `Đã check - Dev`, `Rejected - Dev lead`.
- **Nhóm CS**: `Chờ collab - CS`, `CHỜ KHÁCH - CS`, `Chờ CS`, `Uninstall`, `Done - CS`.
- **Nhóm Follow-up**: `Fl up 1 (12h)`, `Fl up 2 (24h)`, `Fl up 3 (36h)`.

### 4. Quy Tắc 2 Thẻ Ghi Chú Vàng (Dual Yellow Note Cards)
- **Card 1 — `Note transfer case`**: Ghi chú ngắn hạn phục vụ riêng cho ca trực tiếp theo (không lưu vĩnh viễn).
- **Card 2 — `Summary note`**: Tóm tắt cốt lõi vấn đề ticket lưu cố định trong DB, tự động fill sẵn vào form Transfer khi mở modal.

---

## 🤖 Bộ AI Workflow Skills & Customizations (`.agents` & `.claude`)

Repo tích hợp hệ thống quy tắc & kỹ năng tự động hóa SDLC tiêu chuẩn cho AI Assistant:

- **`.agents/AGENTS.md`**: Quy định kiến trúc Theme & UI Development Rules.
- **`.agents/skills.json`**: Điểm kết nối bộ Skills trong `.claude/skills/`.
- **Bộ 17 Skills SDLC (`.claude/skills/`)**:
  - `feature-prd-author`: Soạn PRD Phase (`00-prd.md`).
  - `feature-srs-author`: Soạn SRS Chi tiết Feature (`01-srs.md`).
  - `feature-design-spec-author`: Viết Spec Thiết kế (`02-design-spec.md`).
  - `feature-tech-design-author`: Thiết kế Kiến trúc & Prisma Schema (`03-tech-design.md`).
  - `feature-frontend-impl` & `feature-backend-impl`: Code FE / BE & Migration.
  - `feature-test-plan-author`: Sinh Test Cases (`04-test-plan.md`) & CSV Jira Xray.
  - `feature-shopify-webhooks` / `billing` / `extension-impl` / `appstore-checklist`: Quy chuẩn tích hợp nền tảng Shopify.

---

## 📁 Cấu Trúc Thư Mục (Directory Structure)

```
CS_tool_v2/
├── .agents/                        # AI Custom Rules & Skills Config
│   ├── AGENTS.md                   # SDS Theme Architecture & UI Rules
│   └── skills.json                 # Mapping tới bộ .claude/skills
├── .claude/
│   └── skills/                     # 17 SDLC & Shopify Platform Skills
│       ├── feature-prd-author/
│       ├── feature-srs-author/
│       ├── feature-frontend-impl/
│       └── ...
├── docs/
│   └── phases/
│       └── phase-1-helpdesk-os/    # Tài liệu Phase 1 Helpdesk OS
│           ├── 00-prd.md           # Master PRD (v5.7 Spec)
│           └── features/
│               ├── dashboard-operations-cs/01-srs.md
│               ├── crisp-plugin-integration/01-srs.md
│               ├── shop-detail-360/01-srs.md
│               ├── cs-workflow-guide-documentation/01-srs.md
│               └── app-store-reviews-tracking/01-srs.md
├── helpdesk_cs_demo.html           # Demo Giao diện Helpdesk CS 360° tương tác
├── workflow_demo.md                # Demo Mô phỏng Luồng Vận hành CS
└── README.md                       # Tài liệu tổng quan dự án
```

---

## 🛠️ Hướng Dẫn Sử Dụng & Demo

- Xem demo trực quan giao diện CS Helpdesk: Mở file [`helpdesk_cs_demo.html`](file:///E:/ABC%20soft%20files/CS_tool_v2/helpdesk_cs_demo.html) trên trình duyệt.
- Xem mô phỏng quy trình vận hành: Đọc file [`workflow_demo.md`](file:///E:/ABC%20soft%20files/CS_tool_v2/workflow_demo.md).