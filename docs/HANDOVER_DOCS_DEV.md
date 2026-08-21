# HELPDESK OS — HỒ SƠ TÀI LIỆU KỸ THUẬT & BÀN GIAO TOÀN DIỆN DÀNH CHO DEV TEAM (PHASE 1)

> **Document Type**: Technical Specification & System Architecture Handover  
> **Target Audience**: Backend Engineers, Frontend Engineers, Fullstack Leads, QA Engineers  
> **System Name**: Helpdesk OS (Customer Support & Admin Operations Suite)  
> **Version**: 3.0 (Production Blueprint)  
> **Last Updated**: 21/08/2026  

---

## MỤC LỤC

1. [TỔNG QUAN REPOSITORY (REPO CÓ GÌ?)](#1-tổng-quan-repository-repo-có-gì)
2. [MỤC ĐÍCH HỆ THỐNG (TOOL NÀY ĐỂ LÀM GÌ?)](#2-mục-đích-hệ-thống-tool-này-để-làm-gì)
3. [CRISP PLUGIN LÀM GÌ? (KIẾN TRÚC & TÍNH NĂNG PLUGIN)](#3-crisp-plugin-làm-gì-kiến-trúc--tính-năng-plugin)
4. [QUY TẮC NGHIỆP VỤ CỐT LÕI (CORE BUSINESS RULES)](#4-quy-tắc-nghiệp-vụ-cốt-lõi-core-business-rules)
5. [TECH DESIGN: HỆ THỐNG API ENDPOINTS CẦN XÂY DỰNG](#5-tech-design-hệ-thống-api-endpoints-cần-xây-dựng)
6. [DATABASE SCHEMA (PRISMA / POSTGRESQL DESIGN)](#6-database-schema-prisma--postgresql-design)
7. [CHECKLIST TRIỂN KHAI CHO DEV & QA](#7-checklist-triển-khai-cho-dev--qa)

---

## 1. TỔNG QUAN REPOSITORY (REPO CÓ GÌ?)

### 1.1 Cấu Trúc File & Thư Mục

```
CS_tool_v2/
├── .agents/
│   └── AGENTS.md                                # Quy chuẩn tối thượng: SDS Tokens, DOM safety, Light mode
├── docs/
│   ├── HANDOVER_DOCS_DEV.md                     # Tài liệu bàn giao tổng thể cho Dev (File này)
│   └── phases/
│       └── phase-1-helpdesk-os/
│           ├── 00-prd.md                        # Phase PRD (Product Requirements Document)
│           └── features/                        # 8 Feature SRS Documents chi tiết
│               ├── crisp-plugin-integration/    # 01-srs.md (Crisp Plugin + Auto Segment)
│               ├── dashboard-operations-cs/     # 01-srs.md (CS Dashboard, Filters, Search)
│               ├── shop-detail-360/             # 01-srs.md (Store 360°, Live Meta, Notes)
│               ├── app-store-reviews-tracking/  # 01-srs.md (3-Way Linking, 1-Click Ticket)
│               ├── duty-shift-management/       # 01-srs.md (Roster, SLA Rules, Shift Rotation)
│               ├── cs-analytics-reporting/      # 01-srs.md (KPIs, Status Chart, Performance)
│               ├── cs-workflow-guide-documentation/ # 01-srs.md (Markdown Guide dynamic)
│               └── ticket-management-admin-dev/ # 01-srs.md (Admin 6 Tabs, Modals, CS online)
├── Helpdesk_demo 3.html                         # Live Interactive Demo 3.0 hoàn chỉnh nhất (Single Page App)
├── helpdesk_demo_3.html                         # Bản backup demo 3.0
├── helpdesk_cs_v2.html                          # Bản đồng bộ demo 3.0
└── README.md                                    # Hướng dẫn khởi chạy và tổng quan nhanh
```

### 1.2 Các Phiên Bản Demo Trong Repo
- **`Helpdesk_demo 3.html`**: Bản mô phỏng tương tác 100% logic của toàn bộ hệ thống gồm 3 Platform Switcher:
  1. *Platform 1*: Crisp Chat Simulator + Helpdesk OS Plugin Sidebar.
  2. *Platform 2*: Slack Thread Sync Simulator (bài đăng cá nhân qua OAuth User Token).
  3. *Platform 3*: Internal Web App Dashboard với 7 màn hình làm việc (Dashboard, Shop 360°, Reviews Feed, Duty Shift, Analytics, CS Guide Markdown, Admin Settings).

---

## 2. MỤC ĐÍCH HỆ THỐNG (TOOL NÀY ĐỂ LÀM GÌ?)

### 2.1 Bối Cảnh & Vấn Đề Cần Giải Quyết
Đội ngũ vận hành CS hỗ trợ nhiều Shopify Apps (Avis Product Options, App Bundles, Aris Color Swatch...). Hiện tại quy trình đang gặp các điểm nghẽn:
- **Phân mảnh kênh trao đổi**: Khách chat trên Crisp, nội bộ bàn giao qua Slack, quản lý case trên bảng tính rời rạc khiến trôi sót ticket.
- **Thiếu ngữ cảnh kỹ thuật khi chat**: CS phải mở tab Shopify Admin riêng để tra cứu Store ID, cước phí, plan, app version.
- **Spam thông báo Slack**: Việc bắn thông báo tự động mỗi khi đổi trạng thái làm loãng các kênh trao đổi kỹ thuật của Dev.
- **Bàn giao ca trực đứt đoạn**: Ghi chú ngắn hạn của ca trước bị lẫn với mô tả lỗi kỹ thuật dài hạn.

### 2.2 Mục Đích Cốt Lõi Của Helpdesk OS
Helpdesk OS là hệ điều hành vận hành tập trung (Unified Operations OS) nhằm:
1. **Đồng bộ luồng công việc 3 bên liên tục**: Crisp Livechat ➔ Slack Thread (trao đổi kỹ thuật duy nhất) ➔ Helpdesk OS Web App.
2. **Không spam notification**: Mọi trao đổi diễn ra duy nhất trong **Slack Thread**. Web App lắng nghe webhook để cập nhật status im lặng.
3. **Phân biệt rạch ròi ghi chú**: Tách biệt thẻ màu vàng **`Note transfer case`** (giao ca ngắn hạn giữa CS) và **`Summary note`** (lưu DB dài hạn).
4. **Tự động hóa thông minh**: Tự động phân loại Crisp Segment theo trạng thái ticket, tự động phát hiện Reopen ticket khi khách quay lại, và tự động điều phối cho CS trực ca (`CS online`).

---

## 3. CRISP PLUGIN LÀM GÌ? (KIẾN TRÚC & TÍNH NĂNG PLUGIN)

Plugin Helpdesk OS được nhúng trực tiếp vào Sidebar bên phải của giao diện Crisp Chat Dashboard khi CS chat với merchant:

### 3.1 Các Chức Năng Chính Của Crisp Plugin
1. **Trích Xuất Metadata Thời Gian Thực (Live Visitor Data)**:
   - Tự động nhận diện Store URL từ session của khách (`store_url`, `store_id`, `store_country`, `store_plan`, `store_email`, `user_agent`, `add_charge`, `app_version`, `app_plan`, `pricing_ver`).
2. **Quản Lý Multi-Domain & Sub-Domains**:
   - Cho phép CS gán nhiều sub-domain hoặc custom domain vào cùng 1 store profile.
3. **Tạo Ticket Nhanh 1-Click (`+ Add ticket`)**:
   - Tự động điền Store Domain hiện tại, cho phép CS chọn Target App, Slack Channel, Urgency, Tag ➔ Bắn bài đăng mở **Slack Thread** dưới tên cá nhân của CS (Slack OAuth User Token).
4. **Xem Mini Ticket Cards & Điều Phối**:
   - Hiển thị danh sách ticket đang mở của chính store đó ngay trên sidebar.
   - Nút thao tác nhanh: `Edit`, `View Slack`, `Transfer`.
5. **Cơ Chế Tự Động Gán Crisp Segment (Auto Segment Engine)**:
   - Tự động gắn tag Segment viết tắt lên phiên chat Crisp khi status ticket thay đổi:
     - **`NW`** (*No one waiting*): Áp dụng khi ticket hoàn thành (`Done - CS`, `Done - Need CS Check`, `Uninstall`, `Rejected - Dev lead`, `Done`).
     - **`WR`** (*Waiting reply*): Áp dụng khi Dev/CS đã phản hồi và đang theo dõi merchant trả lời (`Đã check - Dev`, `Fl up 1/2/3`).
     - **`POC`** (*Pending on customer*): Áp dụng khi chờ thông tin bổ sung từ merchant (`CHỜ KHÁCH - CS`, `Chờ CS`, `Chờ collab - CS`).
     - **`POD`** (*Pending on dev*): Áp dụng khi yêu cầu đang nằm trong hàng đợi hoặc Dev đang xử lý (`Chờ check - Dev`, `Đang check - Dev`).

---

## 4. QUY TẮC NGHIỆP VỤ CỐT LÕI (CORE BUSINESS RULES)

### 4.1 Quy Tắc 4 Cấp Độ Urgency & SLA Routing
- **Urgent (Đỏ)**: High risk tickets (Lỗi nghiêm trọng, hỏng checkout, lỗi thanh toán order) — **SLA Target: 15 phút**.
- **High (Vàng cam)**: Khách hàng gói **Premium, Advance plan** — **SLA Target: 1 giờ**.
- **Medium (Xanh dương)**: Khách hàng gói **Pro, Basic plan** — **SLA Target: 4 giờ**.
- **Low (Xám)**: Khách hàng gói **Free, Old, Dev plan** — **SLA Target: 24 giờ**.

### 4.2 Quy Tắc 12 Trạng Thái Custom (Status Workflow)
- **Nhóm Dev**: `Chờ check - Dev`, `Đang check - Dev`, `Đã check - Dev`, `Done - Need CS Check`, `Rejected - Dev lead`.
- **Nhóm CS**: `Chờ collab - CS`, `CHỜ KHÁCH - CS`, `Chờ CS`, `Uninstall`, `Done - CS`.
- **Nhóm Follow-up**: `Fl up 1 (12h)`, `Fl up 2 (24h)`, `Fl up 3 (36h)`.

### 4.3 Quy Tắc Auto Reopened Tracking
- Khi một ticket đang ở trạng thái kết thúc (`Done - CS`, `Done`, `Done - Need CS Check`, `Resolved`) mà được chuyển về bất kỳ trạng thái mở nào ➔ Hệ thống tự động tăng `reopenCount += 1` và gán nhãn `Reopened` màu vàng nổi bật.

### 4.4 Quy Tắc Dual Note Cards
- **Card 1 - `Note transfer case`**: Ghi chú nội bộ ngắn hạn phục vụ giao ca (bị ghi đè ở mỗi lần transfer).
- **Card 2 - `Summary note`**: Tóm tắt bản chất kỹ thuật cốt lõi lưu trữ DB vĩnh viễn (tự động pre-fill vào các modal).

### 4.5 Quy Tắc Transfer Recipient & Điều Phối "CS online"
- Trong Modal Transfer Ticket, trường `Transfer to` cho phép:
  - Chọn đích danh CS: `@aylin (Aylin Tran)`, `@ngan (Ngan Pham)`, `@ha (Ha Tran)`, `@thao (Thao Vo)`.
  - Chọn **`CS online`**: Hệ thống sẽ truy vấn bảng phân ca `Duty Shift Roster` tại thời điểm `remind_at` để tự động gán ticket cho CS đang trực ca tương ứng.

---

## 5. TECH DESIGN: HỆ THỐNG API ENDPOINTS CẦN XÂY DỰNG

Backend API được thiết kế theo chuẩn RESTful JSON API (`/api/v1/`), stateless JWT auth cho Web App và OAuth 2.0 cho Slack & Crisp.

### 5.1 Bảng Tổng Hợp Endpoint API Contract

| HTTP Method | Endpoint Path | Chức Năng Chính | Request Payload / Params | Response Data |
| :--- | :--- | :--- | :--- | :--- |
| **AUTH & SLACK OAUTH** | | | | |
| `POST` | `/api/v1/auth/slack/oauth/callback` | Trao đổi OAuth code lấy Slack User Token của cá nhân CS | `{ code: string }` | `{ token: string, user: CSUser }` |
| `GET` | `/api/v1/auth/me` | Lấy thông tin CS Agent hiện tại & ca trực hôm nay | Headers: `Bearer <token>` | `{ id, name, username, slackTag, role }` |
| **CRISP PLUGIN & INTEGRATION** | | | | |
| `GET` | `/api/v1/crisp/session-meta` | Trích xuất Live Visitor Data của phiên chat Crisp | `?session_id=xxx&domain=xxx` | `{ store_url, store_id, plan, country, app_plan, meta }` |
| `POST` | `/api/v1/crisp/segments/auto-tag` | Gán segment viết tắt (`NW/WR/POC/POD`) lên Crisp Chat | `{ session_id: string, segment: "NW"\|"WR"\|"POC"\|"POD" }` | `{ success: true, updated_segment }` |
| `POST` | `/api/v1/crisp/subdomains` | Gắn thêm sub-domain mới vào tracking của store | `{ store_url: string, subdomain: string }` | `{ store_url, subdomains: string[] }` |
| **TICKET MANAGEMENT** | | | | |
| `GET` | `/api/v1/tickets` | Danh sách ticket kèm filter (App, Assignee, Status, Search) | `?app=APO&status=ALL&assignee=me&search=button&page=1` | `{ tickets: Ticket[], total, unread_count }` |
| `POST` | `/api/v1/tickets` | Tạo ticket mới & Bắn bài đăng mở Slack Thread dưới token CS | `{ app, channel, store_url, status, urgency, request_content, tags }` | `{ id: "TK-4822", slack_thread_ts: "1724...", ticket: Ticket }` |
| `GET` | `/api/v1/tickets/:id` | Chi tiết ticket (request locked, dual notes, logs) | `id: string` (e.g. `TK-4821`) | `{ ticket: TicketDetail, thread_messages }` |
| `PATCH` | `/api/v1/tickets/:id` | Cập nhật Status, Urgency, Tags, Summary Note (Tự động Reopen) | `{ status?, urgency?, tags?, summary_note? }` | `{ ticket: Ticket, reopened_trigger: boolean }` |
| `POST` | `/api/v1/tickets/:id/transfer` | Bàn giao ca trực (Hỗ trợ đích danh CS hoặc `CS online`) | `{ transfer_to: string, remind_at: string, status, urgency, handoff_note, summary_note }` | `{ ticket: Ticket, assigned_agent: string, auto_segment: string }` |
| **SLACK THREAD & WEBHOOK SYNC** | | | | |
| `POST` | `/api/v1/webhooks/slack/events` | Webhook nhận phản hồi trong Thread từ Dev để sync status | Slack Event Payload (url_verification / message) | `{ ok: true }` |
| `GET` | `/api/v1/slack/workspace-channels` | Lấy danh sách channels trong Slack Workspace cho Admin | None | `channels: [{ code: "#apo-paid-task", desc }]` |
| `POST` | `/api/v1/slack/groups/sync` | Đồng bộ danh sách `@cs` và `@dev` từ Slack User Groups | `{ group_type: "cs" \| "dev" }` | `{ members: TeamMember[] }` |
| **STORE 360° & HEALTH** | | | | |
| `GET` | `/api/v1/shops/:domain/360` | Lấy toàn bộ dữ liệu 360° của Store (Meta, Tickets, Health) | `domain: string` (e.g. `kaifit.myapp.io`) | `{ store_info, urgent_count, review_count, no_feedback_loop, notes, tickets }` |
| `POST` | `/api/v1/shops/:domain/notes` | Thêm CS Pinned note cho store | `{ note: string }` | `{ notes: string[] }` |
| `POST` | `/api/v1/shops/:domain/check-in` | Gửi email check-in chăm sóc khách hàng tự động | `{ template: "no_feedback_loop" }` | `{ success: true, sent_at }` |
| **APP STORE REVIEWS FEED** | | | | |
| `GET` | `/api/v1/reviews` | Lấy danh sách review 3-Way Link (Review ➔ Domain ➔ Crisp) | `?rating=negative&page=1` | `{ reviews: Review[], avg_rating: 4.8, complaint_themes }` |
| `POST` | `/api/v1/reviews/:id/convert-ticket` | Chuyển đổi review khiếu nại thành Ticket 1-click | `{ id: string }` | `{ ticket_id: "TK-4823", ticket: Ticket }` |
| **DUTY SHIFT ROSTER & ROUTING** | | | | |
| `GET` | `/api/v1/roster` | Lấy bảng phân ca làm việc tuần hiện tại của CS & Dev | `?week=2026-W34` | `{ roster: MemberRoster[], shifts: string[] }` |
| `PUT` | `/api/v1/roster/shift` | Cập nhật ca trực của 1 nhân sự | `{ member_id: string, day_index: number, shift_code: string }` | `{ success: true, updated_roster }` |
| `POST` | `/api/v1/roster/publish-slack` | Xuất bản lịch trực tuần lên `#cs-announcements` | `{ week: string }` | `{ success: true, slack_message_ts }` |
| `GET` | `/api/v1/roster/on-duty-cs` | Tra cứu CS Agent đang online tại thời điểm xác định | `?timestamp=2026-08-21T14:00:00Z` | `{ on_duty_cs: ["Aylin Tran", "Ngan Pham"] }` |
| **CS ANALYTICS & REPORTING** | | | | |
| `GET` | `/api/v1/analytics/kpis` | Lấy 4 chỉ số KPI lớn theo kỳ | `?range=7d \| 14d \| 30d` | `{ total_volume, resolved, avg_first_response, avg_resolution_time }` |
| `GET` | `/api/v1/analytics/breakdown` | Tỷ lệ % phân bố Status và Urgency | `?range=14d` | `{ status_distribution, urgency_breakdown }` |
| `GET` | `/api/v1/analytics/agent-performance` | Bảng đánh giá năng suất từng CS Agent | `?range=30d` | `agents: [{ name, tickets, avg_frt, csat, sla_compliance }]` |
| **CS WORKFLOW GUIDE & ADMIN CONFIG** | | | | |
| `GET` | `/api/v1/guide` | Lấy nội dung Markdown của bài CS Workflow Guide | None | `{ markdown_content: string, last_updated }` |
| `PUT` | `/api/v1/guide` | Xuất bản nội dung Markdown mới cho CS Workflow Guide | `{ markdown_content: string }` | `{ success: true, published_at }` |
| `GET` | `/api/v1/admin/config` | Lấy toàn bộ cấu hình hệ thống (Apps, Channels, Statuses) | None | `{ apps, slack_channels, statuses, urgency_rules }` |
| `PUT` | `/api/v1/admin/config` | Cập nhật cấu hình hệ thống | `{ apps?, slack_channels?, statuses?, urgency_rules? }` | `{ success: true, config }` |

---

## 6. DATABASE SCHEMA (PRISMA / POSTGRESQL DESIGN)

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

generator client {
  provider = "prisma-client-js"
}

enum UrgencyLevel {
  URGENT  // SLA 15m
  HIGH    // SLA 1h
  MEDIUM  // SLA 4h
  LOW     // SLA 24h
}

enum ShiftType {
  MORNING_08_16
  AFTERNOON_14_22
  NIGHT_20_04
  ON_CALL
  DAY_OFF
}

enum CrispSegment {
  NW   // no one waiting
  WR   // waiting reply
  POC  // pending on customer
  POD  // pending on dev
}

model User {
  id           String       @id @default(uuid())
  email        String       @unique
  name         String
  username     String       @unique // e.g. @aylin
  slackTag     String       // e.g. @cs-aylin
  slackToken   String?      // Encrypted Slack OAuth User Token
  role         String       // CS_AGENT | DEV_LEAD | ADMIN
  assignedTickets Ticket[]  @relation("AssignedTickets")
  rosters      RosterShift[]
  createdAt    DateTime     @default(now())
  updatedAt    DateTime     @updatedAt
}

model StoreProfile {
  id           String       @id @default(uuid())
  domain       String       @unique // e.g. kaifit.myapp.io
  subdomains   String[]     // Multi-domain array
  storeId      String?
  storeEmail   String?
  ownerName    String?
  shopifyPlan  String?
  country      String?
  appVersion   String?
  appPlan      String?
  pricingVer   String?
  pinnedNotes  String[]
  tickets      Ticket[]
  reviews      Review[]
  createdAt    DateTime     @default(now())
  updatedAt    DateTime     @updatedAt
}

model Ticket {
  id             String        @id // e.g. "TK-4821"
  appCode        String        // e.g. "APO", "APB", "ACS"
  slackChannel   String        // e.g. "#apo-paid-task"
  slackThreadTs  String?       // Slack Thread Timestamp ID
  storeUrl       String
  store          StoreProfile  @relation(fields: [storeUrl], references: [domain])
  status         String        // e.g. "Đang check - Dev"
  crispSegment   CrispSegment  @default(POD)
  urgency        UrgencyLevel  @default(MEDIUM)
  reopenCount    Int           @default(0)
  assignedToId   String?
  assignedToUser User?         @relation("AssignedTickets", fields: [assignedToId], references: [id])
  assignedText   String        // e.g. "@aylin (Aylin Tran)" or "CS online"
  requestContent String        // Locked Read-only after creation
  featureTag     String        @default("Live preview")
  tags           String[]
  handoffNote    String?       // Short-term shift transfer note (overwritten)
  summaryNote    String?       // Persistent technical core summary
  isUnread       Boolean       @default(true)
  remindAt       DateTime?
  createdAt      DateTime      @default(now())
  updatedAt      DateTime      @updatedAt

  @@index([storeUrl])
  @@index([status])
  @@index([assignedToId])
}

model Review {
  id           String        @id // e.g. "REV-01"
  stars        Int           // 1 to 5
  title        String
  content      String
  authorName   String
  appCode      String
  domain       String
  store        StoreProfile  @relation(fields: [domain], references: [domain])
  crispUser    String
  badgeText    String
  isNegative   Boolean       @default(false)
  createdAt    DateTime      @default(now())
}

model RosterShift {
  id           String     @id @default(uuid())
  userId       String
  user         User       @relation(fields: [userId], references: [id])
  weekCode     String     // e.g. "2026-W34"
  dayOfWeek    Int        // 0 = Monday, 6 = Sunday
  shift        ShiftType  @default(MORNING_08_16)
  
  @@unique([userId, weekCode, dayOfWeek])
}

model SystemConfig {
  id             String    @id @default("SYSTEM_CONFIG_SINGLETON")
  appsJson       Json      // List of target apps
  channelsJson   Json      // List of mapped Slack channels
  statusesJson   Json      // List of custom statuses
  urgencyJson    Json      // SLA rules
  guideMarkdown  String    @db.Text
  updatedAt      DateTime  @updatedAt
}
```

---

## 7. CHECKLIST TRIỂN KHAI CHO DEV & QA

### 7.1 Backend Checklist
- [ ] Cấu hình Prisma schema và chạy database migration trên PostgreSQL.
- [ ] Tích hợp Slack Web API (`chat.postMessage` dưới token của user CS tạo ticket để mở Slack Thread).
- [ ] Xây dựng Webhook Receiver xử lý events từ Slack (`message.groups`, `message.channels`) để sync trạng thái ticket.
- [ ] Tích hợp Crisp REST API:
  - Gọi API add segment conversation (`NW`, `WR`, `POC`, `POD`) khi ticket update status.
  - Lấy session profile meta khi mở plugin.
- [ ] Xây dựng logic tự động phát hiện Reopen (`reopenCount++`) khi status đổi từ `Done` sang trạng thái mở.
- [ ] Xây dựng logic điều phối `CS online` dựa vào bảng `RosterShift`.

### 7.2 Frontend Checklist
- [ ] Triển khai giao diện dựa trên Design System SDS (Light Mode, bảng màu Slate neutral, không hardcode HEX).
- [ ] Áp dụng triệt để Defensive Programming (`safeSetHTML`, guard checks `if (element)`) để ngăn chặn lỗi runtime.
- [ ] Tích hợp `marked.js` cho CS Workflow Guide và Split-pane Admin Editor (độ trễ preview < 50ms).
- [ ] Đảm bảo Modal Transfer Ticket hiển thị đầy đủ danh sách CS cá nhân và tùy chọn `CS online`.
- [ ] Hiển thị đầy đủ các Badge: Status, Urgency, Crisp Auto Segment (`NW/WR/POC/POD`), Reopened.

### 7.3 QA & Testing Scenarios
1. **Scenario 1 - Luồng tạo ticket & gán Crisp Segment**: Tạo ticket status `Chờ check - Dev` ➔ Kiểm tra Slack xuất hiện thread dưới tên CS ➔ Kiểm tra Crisp session được gán segment `POD`.
2. **Scenario 2 - Luồng Reopen**: Ticket đang `Done - CS` được sửa thành `Chờ check - Dev` ➔ Kiểm tra `reopenCount` tăng lên 1 và xuất hiện badge vàng `Reopened`.
3. **Scenario 3 - Luồng Transfer sang CS Online**: Bàn giao ticket với `Transfer to: CS online`, Remind lúc 14:00 ➔ Kiểm tra ticket tự động gán cho CS Agent có ca `Afternoon 14-22` trong ngày.
4. **Scenario 4 - Luồng Review sang Ticket**: Bấm `+ Create Ticket` từ bảng Review khiếu nại ➔ Mở modal tạo ticket với Domain và Request summary được pre-fill chính xác.

---
*Tài liệu được biên soạn và bảo chứng bởi Tech Lead & BA Team — Helpdesk OS Phase 1.*
