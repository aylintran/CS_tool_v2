---
feature_slug: crisp-plugin-integration
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

# SRS: Crisp Plugin Integration & Ticket / Store Controls (`crisp-plugin-integration`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho **Crisp Livechat & Sidebar Plugin Simulator** trong hệ thống Helpdesk OS. Module này cung cấp không gian làm việc tích hợp: mô phỏng hội thoại Livechat trực tiếp với merchant và bảng điều khiển Helpdesk OS Plugin tại sidebar bên phải, cho phép CS Agent xem thông tin cửa hàng, quản lý danh sách sub-domains, đọc live Visitor Metadata, tạo ticket đẩy lên Slack Thread dưới danh nghĩa cá nhân (Slack OAuth User Token), và quản lý/chuyển giao ticket trực tiếp trong lúc chat.

---

## 2. Scope

### 2.1 In Scope
- **Crisp Chat Simulation Area (`#platform-crisp`)**:
  - Khung hội thoại Livechat: Tên khách hàng (`Woodesign Ireland`), Store URL, trạng thái online (`● Live`), nhãn kênh `Crisp Livechat`.
  - **Auto Crisp Segment Badge**: Hiển thị Segment viết tắt tự động gán theo trạng thái ticket (`NW`, `WR`, `POC`, `POD`).
  - Khung luồng tin nhắn trao đổi giữa Merchant và CS Agent.
  - Ô nhập phản hồi (`Reply to customer...`), nút `Send Reply` và nút `Internal Note`.
- **Crisp Auto Segment Mapping Rules**:
  - `NW` (no one waiting): Khi ticket hoàn thành hoặc không ai phải chờ (`Done - CS`, `Done - Need CS Check`, `Uninstall`, `Rejected - Dev lead`, `Done`).
  - `WR` (waiting reply): Khi CS/Dev đã phản hồi và đang chờ khách trả lời (`Đã check - Dev`, `Fl up 1 (12h)`, `Fl up 2 (24h)`, `Fl up 3 (36h)`).
  - `POC` (pending on customer): Khi chờ thông tin bổ sung từ merchant (`CHỜ KHÁCH - CS`, `Chờ CS`, `Chờ collab - CS`).
  - `POD` (pending on dev): Khi đang trong hàng đợi xử lý của Dev (`Chờ check - Dev`, `Đang check - Dev`).
- **Crisp Right Sidebar Plugin (`#plugin-content-tickets`, `#plugin-content-store`)**:
  - **Tab Controls**: Tab `Tickets (N)` (`#plugtab-tickets`) và Tab `Store info` (`#plugtab-store`).
  - **Quick Action Header**: Nút `+ Add ticket` mở Modal tạo ticket mới với domain của phiên chat hiện tại được điền tự động.
  - **Tab Tickets Content**:
    - Hiển thị thông tin Store URL đang chat (VD: `61cde3-42.myshopify.com`).
    - Danh sách ticket cards thu nhỏ (`#plugin-tickets-container`): Status Badge, Urgency Badge, Reopened Badge, Channel name, Assigned to, Timestamp, Request summary, Feature name (`Live preview`).
    - Cụm Action Buttons: `Edit` (mở Modal Edit), `View Slack` (chuyển sang tab mô phỏng Slack Thread), `Transfer` (mở Modal Transfer).
  - **Tab Store Info Content**:
    - Quản lý Sub-domains: Ô nhập sub-domain mới (`#new-subdomain-input`), nút `+` (`addSubdomain()`), và danh sách tag pills hiển thị sub-domains đã gắn (`#subdomain-list`).
    - Bảng dữ liệu **Visitor Data (Live Meta)**: Bảng chia 2 cột hiển thị các tham số live:
      - `store_url`: Domain cửa hàng.
      - `store_id`: Mã ID cửa hàng Shopify.
      - `store_country`: Quốc gia của merchant.
      - `store_plan`: Gói cước Shopify (e.g. `Shopify Premium`).
      - `store_email`: Email tài khoản quản trị store.
      - `user_agent`: Thông tin trình duyệt/thiết bị của merchant.
      - `add_charge`: Mã version cước phí (e.g. `V3`).
      - `app_version`: Phiên bản app đang cài đặt (e.g. `v2`).
      - `app_plan`: Gói cước ứng dụng (e.g. `FREE`, `GROWTH`).
      - `pricing_ver`: Version bảng giá áp dụng (e.g. `v5`).

### 2.2 Out of Scope
- Tự động thay đổi mã theme Liquid trực tiếp từ trong khung chat Crisp mà không có sự xác nhận của merchant.
- Nhúng gọi video/audio call (chỉ hỗ trợ Livechat text).

---

## 3. Key Business Rules

- **3.1. Pre-filled Domain on Ticket Creation**: Khi CS bấm `+ Add ticket` từ thanh tiêu đề của Crisp Plugin, hệ thống bắt buộc tự động gán giá trị Store Domain hiện tại của phiên chat (e.g. `61cde3-42.myshopify.com`) vào ô `#add-domain-input` trong Modal Add Ticket.
- **3.2. Dynamic Sub-domain Association**: Khi CS gõ một sub-domain mới vào ô `#new-subdomain-input` và bấm `+`:
  1. Kiểm tra chuỗi không được để trống.
  2. Tạo mới một tag pill sub-domain màu xanh nhạt (`bg-slate-100 text-slate-800 text-[10px] px-2 py-0.5 rounded border border-slate-300 font-mono`).
  3. Gắn sub-domain này vào danh sách sub-domains của store trong DB.
  4. Hiển thị Toast thông báo: `Đã gắn sub-domain [Val] vào tracking store!`.
- **3.3. Slack OAuth User Token Execution**: Khi ticket được tạo từ Crisp Plugin và bật tùy chọn `Auto-trigger Slack Thread`, bài post trên Slack bắt buộc hiển thị tác giả là tài khoản cá nhân của CS (Aylin Tran) thay vì Bot chung, giúp Dev nhận diện đúng người phụ trách để reply trực tiếp.
- **3.4. Tab State Persistence**: Khi chuyển đổi qua lại giữa Tab `Tickets` và `Store info`, trạng thái form và danh sách dữ liệu được giữ nguyên vẹn mà không bị tải lại toàn bộ plugin.
- **3.5. Plain UI & No-Emoji Policy**: Toàn bộ các nút bấm và nhãn trong Crisp Plugin đều tuân thủ phong cách plain text, không dùng icon emoji.

---

## 4. Domain Model

### 4.1 Entities
- `CrispVisitorMeta`:
  - `storeUrl` (string): URL chính của merchant.
  - `storeId` (string): ID duy nhất trên Shopify.
  - `storeCountry` (string): Tên quốc gia.
  - `storePlan` (string): Gói cước Shopify.
  - `storeEmail` (string): Email merchant.
  - `userAgent` (string): Thông tin User-Agent.
  - `addCharge` (string): Mã phụ phí.
  - `appVersion` (string): Version app.
  - `appPlan` (string): Gói app merchant đang dùng.
  - `pricingVer` (string): Phiên bản định giá.
- `Subdomain`:
  - `domain` (string): Chuỗi subdomain phụ (e.g. `checkout.woodesign.ie`).
  - `parentStoreUrl` (string): Domain store chính liên kết.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `store_url` | string | Yes | None | Phải là chuỗi domain hợp lệ, không chứa ký tự đặc biệt ngoài dấu gạch ngang và chấm. |
| `store_id` | string | Yes | None | Chuỗi số nguyên dương đại diện cho Shopify Shop ID. |
| `store_email` | string | Yes | None | Phải đúng cấu trúc email chuẩn RFC 5322. |
| `app_plan` | string | Yes | `FREE` | Phải thuộc danh mục gói cước hợp lệ (`FREE`, `BASIC`, `GROWTH`, `PREMIUM`). |
| `subdomain_input` | string | No | `""` | Tối thiểu 3 ký tự khi bấm thêm sub-domain. |

---

## 6. State Transitions

- **Chuyển Tab Plugin**:
  - `switchPluginTab('tickets')`: Kích hoạt tab Tickets (`#plugin-content-tickets`), ẩn tab Store info (`#plugin-content-store`).
  - `switchPluginTab('store')`: Kích hoạt tab Store info (`#plugin-content-store`), ẩn tab Tickets (`#plugin-content-tickets`).
- **Thêm Sub-domain**:
  - `Chưa liên kết` ➔ `Đã liên kết vào danh sách Tracking`.

---

## 7. Runtime/behavior contract

- **Input**: Phiên chat active trên Crisp, click chuyển tab hoặc click thêm sub-domain.
- **Output**: Cập nhật DOM của `#plugin-tickets-container`, `#subdomain-list` và bảng Visitor Data.
- **Fallback**: Nếu store chưa có ticket nào, render dòng thông báo rỗng an toàn: `Chưa có ticket nào được tạo cho store domain này.`

---

## 8. QA Scenarios

1. Kiểm tra chuyển đổi giữa Tab `Tickets` và Tab `Store info` hiển thị đúng panel nội dung tương ứng.
2. Kiểm tra Tab Tickets hiển thị đúng số lượng ticket mở của store đang chat.
3. Kiểm tra click `+ Add ticket` mở Modal Add Ticket với ô Store Domain đã được điền sẵn domain của phiên chat.
4. Kiểm tra trên từng thẻ ticket trong Crisp Plugin: hiển thị Status badge, Urgency badge, Channel và tên CS phụ trách.
5. Kiểm tra click `Edit` trên thẻ ticket của Crisp Plugin mở đúng Modal Edit Ticket của ticket đó.
6. Kiểm tra click `Transfer` mở Modal Transfer Ticket với Summary Note được nạp sẵn.
7. Kiểm tra click `View Slack` chuyển sang nền tảng mô phỏng Slack Thread.
8. Kiểm tra chuyển sang Tab Store Info: bảng Visitor Data hiển thị đầy đủ 10 trường metadata.
9. Kiểm tra nhập subdomain vào `#new-subdomain-input` và bấm `+`: tag pill mới được tạo và thêm vào `#subdomain-list`.
10. Kiểm tra nhập chuỗi rỗng vào `#new-subdomain-input` và bấm `+`: hệ thống không tạo tag pill rác.
11. Kiểm tra sau khi thêm subdomain thành công, ô input tự động được xóa trắng (`input.value = ''`).
12. Kiểm tra hiển thị Toast notification xác nhận sau khi thêm subdomain thành công.
13. Kiểm tra các nút trong Crisp Plugin không chứa icon emoji.
14. Kiểm tra giao diện Light Mode có độ tương phản cao, chữ rõ nét trên nền trắng.
15. Kiểm tra khi có ticket mới được tạo, biến đếm `#plugin-ticket-count` trên tiêu đề tab tự động cập nhật số lượng mới.

---

## 9. Implementation notes for AI code generation

- **DOM Preservation**: Bắt buộc giữ nguyên vẹn các ID: `plugtab-tickets`, `plugtab-store`, `plugin-content-tickets`, `plugin-content-store`, `plugin-ticket-count`, `plugin-tickets-container`, `new-subdomain-input`, `subdomain-list`.
- **Defensive Guard**: Sử dụng `safeSetHTML` khi render danh sách `plugin-tickets-container`.

---

## 10. Final implementation assumptions to review

- Crisp Plugin chạy dưới dạng Sidebar Iframe nhúng trong Crisp Inbox dashboard của CS Agent.

---

## 11. User Stories

### Story: US-01 — CS tra cứu Visitor Data và thêm Sub-domain từ Crisp Plugin
**As a** CS Agent  
**I want to** mở Tab Store info trên Crisp Plugin để xem thông số kỹ thuật của merchant và thêm sub-domain theo dõi  
**So that** tôi nắm rõ gói cước và cấu hình kỹ thuật của khách hàng ngay trong lúc tư vấn.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang mở cuộc trò chuyện với merchant trên Crisp  
  **When** CS click vào tab `Store info`  
  **Then** Bảng Visitor Data hiển thị đầy đủ thông tin: `store_url`, `store_id`, `store_plan`, `app_version`  
  **And** CS nhập `checkout.kaifit.vn` vào ô `#new-subdomain-input` và bấm nút `+`  
  **Then** Hệ thống thêm sub-domain vào danh sách hiển thị và bắn Toast thông báo thành công.
