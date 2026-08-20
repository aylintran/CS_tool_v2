---
feature_slug: cs-workflow-guide-documentation
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

# SRS: CS Operational Workflow Guide Documentation (`cs-workflow-guide-documentation`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **CS Workflow Guide (`📖 CS Workflow Guide`)** tích hợp sẵn trong ứng dụng Internal Tool. Màn hình giúp đào tạo và làm tài liệu tra cứu chuẩn hóa quy trình cho nhân viên CS, bao gồm sơ đồ luồng 4 bước, luồng Follow-up thủ công (Fl up 1-2-3 -> Resolve), quy tắc 12 trạng thái custom và quy tắc 2 thẻ ghi chú vàng. Mỗi mục nội dung được đóng gói trong từng Accordion component có thể thu gọn/mở rộng.

---

## 2. Scope

### 2.1 In Scope
- **Giao diện Accordion (Collapsible Sections)**: Từng mục nội dung hướng dẫn được đóng gói riêng trong 1 Accordion card có thể đóng/mở (expand/collapse) linh hoạt để CS dễ tra cứu.
- **Sơ đồ luồng 4 bước (4-Step Operational Flow)**:
  1. Tiếp nhận Crisp Chat ➔ Check Plugin Visitor Data.
  2. Tạo Ticket (+ Add ticket) ➔ Đẩy Slack Thread dưới tên cá nhân CS (User Token).
  3. Thảo luận duy nhất tại Slack ➔ Web App lắng nghe im lặng không gây nhiễu alert.
  4. Bàn giao ca (Transfer) ➔ Handoff Note ngắn hạn + Summary Note lưu DB auto-fill.
- **Quy trình Follow-up (Follow-up Flow)**:
  - Khi Dev trả case trên Slack và báo Request Done: CS tiến hành Resolve đoạn chat trên Crisp với merchant.
  - Sau đó, CS chuyển trạng thái ticket sang `Fl up 1 (12h)` thủ công (manually).
  - Tiếp tục chuyển thủ công (manually) qua `Fl up 2 (24h)` ➔ `Fl up 3 (36h)`.
  - Sau khi hoàn tất `Fl up 3 (36h)` (hoặc merchant đã xác nhận OK), CS chuyển trạng thái ticket sang `Done - CS` (Resolve ticket).
- **Quy tắc 12 Trạng Thái Custom**: Phân loại theo 3 nhóm trách nhiệm (Nhóm Dev, Nhóm CS, Nhóm Follow-up).
- **Quy tắc 2 Thẻ Ghi Chú Vàng (Dual Yellow Note Cards)**: Phân biệt `Note transfer case` ngắn hạn ca trực và `Summary note` lưu DB lâu dài.

### 2.2 Out of Scope
- Chỉnh sửa văn bản hướng dẫn quy trình bởi user thông thường (chỉ Admin mới được cập nhật doc gốc).

---

## 3. Key Business Rules

- **3.1. Accordion UI Component**: Mọi phần nội dung quy trình trên màn hình `📖 CS Workflow Guide` phải nằm trong từng Accordion độc lập. Mặc định mở (Expanded) hoặc thu gọn (Collapsed) theo thiết kế, hỗ trợ bấm vào header để đóng/mở.
- **3.2. Manual Follow-up Process**:
  1. Dev báo hoàn thành trên Slack Thread (Request Done) ➔ CS thực hiện Resolve chat trên Crisp.
  2. CS chủ động đổi trạng thái ticket sang `Fl up 1 (12h)` thủ công.
  3. Tiếp tục theo dõi và chuyển lần lượt sang `Fl up 2 (24h)` và `Fl up 3 (36h)` thủ công (manually).
  4. Sau `Fl up 3 (36h)`, CS thực hiện Resolve ticket (chuyển trạng thái `Done - CS`).

---

## 4. QA Scenarios

1. Kiểm tra mục `📖 CS Workflow Guide` xuất hiện nổi bật màu vàng ở Sidebar bên trái.
2. Kiểm tra click chọn `📖 CS Workflow Guide` mở đúng màn hình hướng dẫn.
3. Kiểm tra từng phần nội dung (Sơ đồ 4 bước, Luồng Follow-up, 12 Trạng thái, 2 Khung vàng) được bọc trong từng Accordion card riêng biệt.
4. Kiểm tra thao tác click vào header Accordion để đóng lại / mở ra mượt mà.
5. Kiểm tra sơ đồ luồng Follow-up thể hiện đúng các bước: Dev trả case ➔ CS Resolve chat Crisp ➔ Đổi trạng thái `Fl up 1 (12h)` (manually) ➔ `Fl up 2 (24h)` ➔ `Fl up 3 (36h)` ➔ Resolve ticket (`Done - CS`).
6. Kiểm tra danh sách 12 trạng thái custom được phân loại đúng 3 nhóm (Dev, CS, Follow-up).
7. Kiểm tra phân biệt rõ ràng vai trò của Card 1 (`Note transfer case`) và Card 2 (`Summary note`).

---

## 5. User Stories

### Story: US-05 — CS Agent tra cứu quy trình làm việc chuẩn hóa trên CS Workflow Guide
**As a** CS Agent mới  
**I want to** mở màn hình CS Workflow Guide dạng Accordion trực tiếp trên Sidebar  
**So that** dễ dàng đóng/mở tra cứu sơ đồ 4 bước, quy trình Follow-up (Fl up 1-2-3 -> Resolve), 12 trạng thái và 2 khung vàng bàn giao ca.

#### Acceptance Criteria (Gherkin)
- **Given** CS đang làm việc trong Internal Web App  
  **When** CS click vào `📖 CS Workflow Guide` ở Sidebar bên trái  
  **Then** Màn hình hiển thị các Accordion chứa sơ đồ 4 bước, luồng Follow-up thủ công (Fl up 1-2-3 -> Resolve), quy tắc 12 trạng thái custom và 2 khung vàng note chi tiết  
  **And** CS có thể bấm vào từng Accordion header để đóng bớt hoặc mở ra phần nội dung cần tra cứu.

### Links
- Business rule: [SRS Section 3.1](#31-accordion-ui-component) & [SRS Section 3.2](#32-manual-follow-up-process)
