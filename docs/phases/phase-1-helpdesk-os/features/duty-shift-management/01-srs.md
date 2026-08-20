---
feature_slug: duty-shift-management
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

# SRS: Duty Shift Roster & SLA Routing (`duty-shift-management`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **Duty Shift Roster (`📅 Duty Shift Roster`)** trong Internal Web App. Màn hình giúp quản lý lịch phân ca trực của đội ngũ CS & Dev (Mon–Sun), xuất bản lịch làm việc lên kênh Slack team channel, và cấu hình bảng điều hướng **SLA Routing Rules** tự động gán ticket mở mới trong ca trực cho nhân viên CS đang trực ca.

---

## 2. Scope

### 2.1 In Scope
- **Bảng Lịch Phân Ca Trực (Roster Table Mon-Sun)**:
  - Danh sách nhân sự thuộc CS Team và Dev Team.
  - Phân loại 5 loại ca trực: `Morning 08-16` (blue), `Afternoon 14-22` (amber), `Night 20-04` (purple), `On-call` (emerald), `Day off` (slate).
  - Click vào ô ca trực để toggle xoay vòng các loại ca.
- **Nút Publish Roster**: Phát thông báo thông tin lịch trực tuần mới lên Slack Channel chung của team.
- **Bảng Cấu Hình SLA Routing Rules**:
  - `🔴 Urgent` (High risk tickets) ➔ SLA Target: 15 phút.
  - `🟠 High` (Premium / Advance plan) ➔ SLA Target: 1 giờ.
  - `🔵 Medium` (Pro / Basic plan) ➔ SLA Target: 4 giờ.
  - `⚪ Low` (Free / Old / Dev plan) ➔ SLA Target: 24 giờ.
  - Tự động gán nhân sự CS On-duty hiện tại làm mặc định cho các ticket tạo mới trong khung giờ trực.

### 2.2 Out of Scope
- Chỉnh sửa quy trình tính lương / overtime nhân sự.

---

## 3. Key Business Rules

- **3.1. Roster Cycle**: Lịch trực tính từ Thứ Hai (Mon) đến Chủ Nhật (Sun), được phép chỉnh sửa bởi `admin` hoặc `cs_lead`.
- **3.2. Slack Publish Notification**: Khi bấm `Publish Roster`, hệ thống phát tin nhắn webhook tới Slack Team Channel liệt kê danh sách trực ca tuần của toàn bộ CS/Dev.
- **3.3. Auto SLA Assignee**: Ticket mới được tạo tự động liên kết với CS Agent đang trực ca theo bảng SLA Routing Rules tương ứng với Urgency Level của ticket.

---

## 4. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `member_id` | string | Yes | None | ID nhân sự CS/Dev. |
| `day_of_week` | string | Yes | `MON` | `MON` đến `SUN`. |
| `shift_type` | enum | Yes | `Day off` | `Morning 08-16`, `Afternoon 14-22`, `Night 20-04`, `On-call`, `Day off`. |
| `sla_target_minutes` | integer | Yes | `240` | Thời gian SLA target tính bằng phút (15, 60, 240, 1440). |

---

## 5. QA Scenarios

1. Kiểm tra bảng Roster hiển thị đủ các ngày từ MON đến SUN.
2. Kiểm tra click vào ô ca trực toggle qua lại giữa 5 loại ca và đổi màu nhãn tương ứng.
3. Kiểm tra bấm `Publish Roster` gửi tin nhắn lịch trực tuần lên kênh Slack.
4. Kiểm tra bảng SLA Routing hiển thị đúng 4 mức độ Urgency cùng SLA target (15m, 1h, 4h, 24h) và hiển thị CS On-duty hiện tại.
