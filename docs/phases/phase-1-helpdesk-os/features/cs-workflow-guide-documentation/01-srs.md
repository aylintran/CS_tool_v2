---
feature_slug: cs-workflow-guide-documentation
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

# SRS: CS Operational Workflow Guide Documentation (`cs-workflow-guide-documentation`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **CS Workflow Guide** trong Internal Web App thuộc Helpdesk OS. Tính năng này cung cấp một trung tâm tài liệu chuẩn hóa quy trình vận hành trực quan được render động từ mã nguồn Markdown (`marked.js`), phục vụ đào tạo nhân sự mới và tra cứu quy trình hàng ngày cho CS Agent. Tài liệu bao gồm: Sơ đồ luồng vận hành 4 bước, Quy tắc 4 cấp độ Urgency và SLA, Quy tắc 12 trạng thái custom, và Quy tắc phân biệt Dual Note Cards. Đồng thời, trang cung cấp nút điều hướng nhanh `Edit Guide in Admin` cho phép PO / Dev Lead truy cập trình soạn thảo Markdown trong Admin Settings.

---

## 2. Scope

### 2.1 In Scope
- **Header Toolbar**:
  - Tiêu đề: `CS Operational Workflow Guide — Helpdesk OS`.
  - Nút tác vụ: `Edit Guide in Admin` (chuyển thẳng tới Tab Markdown Editor trong Admin Settings).
- **Markdown Rendered Documentation View (`#guide-markdown-rendered-view`)**:
  - Render trực tiếp từ `state.config.guideMarkdown` (hoặc `defaultGuideMarkdownContent`) qua thư viện `marked.parse()`.
  - Bố cục tài liệu gồm các phân mục chính:
    1. **1. Sơ Đồ Luồng Vận Hành 4 Bước (Step-by-Step Flow)**:
       - *Step 1: Tiếp nhận trên Crisp Chat*: Tra cứu Visitor Data, Store ID, Sub-domain trên Crisp Plugin.
       - *Step 2: Tạo Ticket (`+ Add ticket`)*: Điền form, chọn Target App, Slack Channel, Status, Urgency, Tag ➔ Tự động tạo Slack Thread dưới tên CS cá nhân (Slack OAuth User Token).
       - *Step 3: Thảo luận duy nhất tại Slack*: Thảo luận file và tag Dev duy nhất trên Slack Thread. Web App đồng bộ webhook im lặng.
       - *Step 4: Bàn giao ca trực (`Transfer Ticket`)*: Mở Modal Transfer, điền `Note transfer case` ngắn hạn + tự động load `Summary note` lưu DB ➔ Chuyển ca không đứt đoạn.
    2. **2. Quy Tắc 4 Cấp Độ Urgency & SLA**:
       - *Urgent*: Lỗi nghiêm trọng, hỏng checkout / order (SLA: 15 phút).
       - *High*: Khách hàng gói Premium, Advance plan (SLA: 1 giờ).
       - *Medium*: Khách hàng gói Pro, Basic plan (SLA: 4 giờ).
       - *Low*: Khách hàng gói Free, Old, Dev plan (SLA: 24 giờ).
    3. **3. Quy Tắc 2 Thẻ Ghi Chú Vàng (Dual Yellow Note Cards)**:
       - *Card 1 - Note transfer case*: Ghi chú ngắn hạn ca trực giữa các CS với nhau (được ghi đè theo từng ca).
       - *Card 2 - Summary note*: Tóm tắt bản chất kỹ thuật gốc lưu trong DB (tự động nạp vào Form Transfer).
- **Dynamic Content Synchronization**:
  - Khi nội dung được chỉnh sửa và bấm Publish trong Admin Settings, nội dung trên màn hình CS Workflow Guide tự động cập nhật ngay lập tức mà không cần reload trang.

### 2.2 Out of Scope
- Chỉnh sửa trực tiếp nội dung văn bản ngay trên màn hình đọc (phải thông qua Tab Admin Guide Editor).

---

## 3. Key Business Rules

- **3.1. Markdown-Driven Rendering**: Toàn bộ nội dung của trang được quản lý dưới dạng Markdown string trong `state.config.guideMarkdown`. Khi hiển thị, hàm `renderCSGuideMarkdownView()` chuyển đổi Markdown sang HTML chuẩn qua `marked.parse()`.
- **3.2. Role-Based Edit Shortcut**: Nút `Edit Guide in Admin` chỉ mở giao diện soạn thảo khi user có quyền quản trị, kích hoạt hàm `switchInternalView('admin')` và `switchAdminTab('guide-edit')`.
- **3.3. Plain Light Mode Typography**: Các thẻ heading (`h1`, `h2`, `h3`), đoạn văn (`p`), danh sách (`ul`, `li`), và đoạn mã (`code`) tuân thủ style CSS `.prose` nền sáng với chữ đậm sắc nét, không chèn emoji icons.

---

## 4. Domain Model

### 4.1 Entities
- `GuideDocumentation`:
  - `version` (string): Phiên bản tài liệu (e.g. `v5.7`).
  - `markdownContent` (string): Toàn bộ mã nguồn Markdown của bài hướng dẫn.
  - `lastUpdated` (string): Thời điểm cập nhật gần nhất.
  - `publishedBy` (string): Tên người xuất bản.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `guide_markdown` | string | Yes | Default text | Chuỗi Markdown hợp lệ, tối thiểu 100 ký tự. |

---

## 6. State Transitions

- **Đọc và Điều hướng**:
  - `Xem Guide ➔ Bấm Edit Guide in Admin ➔ Chuyển sang Tab Admin Guide Editor`.

---

## 7. Runtime/behavior contract

- **Input**: Chuỗi Markdown từ `state.config.guideMarkdown`.
- **Output**: Render HTML vào `#guide-markdown-rendered-view` qua `safeSetHTML`.
- **Fallback**: Nếu `state.config.guideMarkdown` rỗng, tự động lấy `defaultGuideMarkdownContent`.

---

## 8. QA Scenarios

1. Kiểm tra click vào `CS Workflow Guide` ở Sidebar bên trái mở đúng màn hình hướng dẫn.
2. Kiểm tra tài liệu được render đầy đủ định dạng: Tiêu đề, danh sách có dấu chấm tròn, các khối mã inline font-mono.
3. Kiểm tra nội dung thể hiện đầy đủ 3 phần: Sơ đồ 4 bước, 4 Cấp Urgency & SLA, và Quy tắc 2 Thẻ ghi chú vàng.
4. Kiểm tra click nút `Edit Guide in Admin` chuyển ngay sang Tab CS Guide Editor trong Admin Settings.
5. Kiểm tra sau khi Publish nội dung mới trong Admin, quay lại màn hình Guide thấy nội dung đã được cập nhật.
6. Kiểm tra sau khi bấm `Reset Default` trong Admin, nội dung Guide quay về bản chuẩn mặc định.
7. Kiểm tra giao diện hiển thị sạch đẹp, không có emoji icons.
8. Kiểm tra thanh cuộn của khung tài liệu hoạt động mượt mà.
9. Kiểm tra các liên kết trong văn bản Markdown (nếu có) mở an toàn.
10. Kiểm tra hiển thị đúng độ tương phản trên nền Light Mode.

---

## 9. Implementation notes for AI code generation

- **Preserved DOM Identifiers**: `guide-markdown-rendered-view`, `internal-view-guide`.
- **Marked.js Integration**: Đảm bảo thư viện `marked.min.js` được nạp tại thẻ `<head>`.

---

## 10. Final implementation assumptions to review

- Tài liệu hướng dẫn dùng chung cho toàn bộ các ứng dụng trong hệ sinh thái Shopify Apps.

---

## 11. User Stories

### Story: US-01 — CS Agent tra cứu quy trình vận hành trên CS Workflow Guide
**As a** CS Agent  
**I want to** mở màn hình CS Workflow Guide trên Sidebar để tra cứu sơ đồ 4 bước và quy tắc 2 thẻ note  
**So that** tôi thực hiện đúng quy trình tiếp nhận, tạo ticket Slack và giao ca chuẩn chỉnh.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang làm việc trong Internal Web App  
  **When** CS click vào `CS Workflow Guide` ở Sidebar  
  **Then** Màn hình hiển thị đầy đủ tài liệu quy trình được render từ Markdown với 3 phân mục cốt lõi  
  **And** CS có thể đọc rõ ràng các bước và hạn SLA của từng cấp độ Urgency.
