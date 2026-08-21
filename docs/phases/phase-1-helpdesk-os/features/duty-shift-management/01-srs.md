---
feature_slug: duty-shift-management
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

# SRS: Duty Shift Roster & SLA Routing (`duty-shift-management`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **Duty Shift Roster** trong Internal Web App thuộc Helpdesk OS. Tính năng này hỗ trợ CS Lead và Dev Lead quản lý lịch phân ca trực hàng tuần (từ Thứ Hai đến Chủ Nhật) cho cả đội ngũ CS và Dev, xuất bản thông báo lịch làm việc lên kênh Slack `#cs-announcements`, và chuẩn hóa bảng quy tắc **SLA Routing Rules** theo 4 cấp độ ưu tiên (Urgent, High, Medium, Low).

---

## 2. Scope

### 2.1 In Scope
- **Duty Shift Roster Table (Mon – Sun)**:
  - Danh sách nhân sự CS Team: `Aylin Tran`, `Ngan Pham`, `Ha Tran`, `Thao Vo`.
  - Danh sách nhân sự Dev Team: `Tuan Nguyen`, `Minh Dao`, `Hung Le`, `Bao Nguyen`.
  - 5 loại ca trực có mã màu rõ ràng:
    - `Morning 08-16` (badge xanh dương `bg-blue-100 text-blue-800 border-blue-200`).
    - `Afternoon 14-22` (badge vàng cam `bg-amber-100 text-amber-800 border-amber-200`).
    - `Night 20-04` (badge tím `bg-purple-100 text-purple-800 border-purple-200`).
    - `On-call` (badge xanh lục `bg-emerald-100 text-emerald-800 border-emerald-200`).
    - `Day off` (badge xám `bg-slate-100 text-slate-600 border-slate-200`).
  - Thao tác click vào từng ô ca trực để xoay vòng trạng thái ca.
- **Publish Roster to Slack Button**:
  - Nút `Publish Roster to #cs-announcements` phát webhook gửi bảng tổng hợp lịch trực tuần lên kênh Slack của team.
- **SLA Routing Rules Section**:
  - Bảng định nghĩa 4 cấp độ Urgency và SLA phản hồi mục tiêu:
    - `Urgent`: High risk tickets (Lỗi nghiêm trọng, hỏng checkout / order) ➔ SLA Target: **15 phút**.
    - `High`: Khách hàng gói Premium, Advance plan ➔ SLA Target: **1 giờ**.
    - `Medium`: Khách hàng gói Pro, Basic plan ➔ SLA Target: **4 giờ**.
    - `Low`: Khách hàng gói Free, Old, Dev plan ➔ SLA Target: **24 giờ**.

### 2.2 Out of Scope
- Tự động chấm công qua vân tay / GPS.
- Tính toán bảng lương và phụ cấp ca đêm tự động.

---

## 3. Key Business Rules

- **3.1. Weekly Roster Cycle**: Lịch trực tính từ Thứ Hai (Mon) đến Chủ Nhật (Sun). Chỉ người dùng có vai trò `cs_lead` hoặc `admin` mới có quyền chỉnh sửa và bấm Publish.
- **3.2. Shift Rotation Logic**: Khi click vào một ô ca trực trong bảng, trạng thái ca trực xoay vòng tuần tự theo thứ tự: `Morning 08-16` ➔ `Afternoon 14-22` ➔ `Night 20-04` ➔ `On-call` ➔ `Day off` ➔ `Morning 08-16`.
- **3.3. SLA Routing Alignment**: Bảng SLA Routing Rules đảm bảo mọi ticket tạo mới trong ca đều có mốc cam kết xử lý rõ ràng. Khi ticket có mức `Urgent`, hệ thống ưu tiên hiển thị cảnh báo đỏ và đếm ngược 15 phút.
- **3.4. No-Emoji Plain Styling**: Toàn bộ các nút và nhãn trong màn hình Duty Shift Roster tuân thủ phong cách plain text, không kèm emoji icons.

---

## 4. Domain Model

### 4.1 Entities
- `ShiftAssignment`:
  - `memberId` (string): Tên/Mã nhân sự.
  - `role` (string): `CS` hoặc `Dev`.
  - `dayOfWeek` (string): `MON`, `TUE`, `WED`, `THU`, `FRI`, `SAT`, `SUN`.
  - `shiftType` (string): Loại ca trực.
- `SlaPolicy`:
  - `urgencyLevel` (string): `Urgent`, `High`, `Medium`, `Low`.
  - `description` (string): Tiêu chuẩn phân loại merchant/lỗi.
  - `slaMinutes` (integer): Thời gian mục tiêu (15, 60, 240, 1440).

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `member_name` | string | Yes | None | Tên nhân sự trong `csTeam` hoặc `devTeam`. |
| `day_of_week` | string | Yes | `MON` | Thuộc danh sách: `MON`, `TUE`, `WED`, `THU`, `FRI`, `SAT`, `SUN`. |
| `shift_type` | enum | Yes | `Day off` | Thuộc 1 trong 5 loại ca quy chuẩn. |
| `sla_target_minutes` | integer | Yes | `240` | Phải là một trong các giá trị: 15, 60, 240, 1440. |

---

## 6. State Transitions

- **Đổi ca trực**:
  - `Morning 08-16` ➔ `Afternoon 14-22` ➔ `Night 20-04` ➔ `On-call` ➔ `Day off`.
- **Xuất bản lịch**:
  - `Lịch nháp ➔ Bấm Publish ➔ Gửi Slack Webhook ➔ Toast xác nhận`.

---

## 7. Runtime/behavior contract

- **Input**: User click vào ô ca trực trên bảng Roster hoặc click nút `Publish Roster`.
- **Output**: Cập nhật class màu và nhãn của ô ca trực; phát webhook tới Slack `#cs-announcements`.
- **Fallback**: Nếu kết nối Slack bị gián đoạn, lịch vẫn được lưu trên Web App và hiển thị thông báo lỗi mạng an toàn.

---

## 8. QA Scenarios

1. Kiểm tra bảng Roster hiển thị đầy đủ 7 cột từ MON đến SUN.
2. Kiểm tra danh sách nhân sự CS hiển thị đủ 4 thành viên (Aylin Tran, Ngan Pham, Ha Tran, Thao Vo).
3. Kiểm tra danh sách nhân sự Dev hiển thị đủ 4 thành viên (Tuan Nguyen, Minh Dao, Hung Le, Bao Nguyen).
4. Kiểm tra click vào ô ca trực đổi màu và đổi nhãn đúng chu trình 5 ca.
5. Kiểm tra ca Morning có màu xanh dương, Afternoon có màu vàng cam, Night có màu tím, On-call màu xanh lục, Day off màu xám.
6. Kiểm tra click nút `Publish Roster to #cs-announcements` hiển thị Toast thông báo xuất bản thành công.
7. Kiểm tra bảng SLA Routing hiển thị đủ 4 cấp độ: Urgent, High, Medium, Low.
8. Kiểm tra SLA của Urgent hiển thị `SLA: 15 mins`.
9. Kiểm tra SLA của High hiển thị `SLA: 1 hour`.
10. Kiểm tra SLA của Medium hiển thị `SLA: 4 hours`.
11. Kiểm tra SLA của Low hiển thị `SLA: 24 hours`.
12. Kiểm tra các nút bấm không chứa icon emoji.
13. Kiểm tra giao diện hiển thị sắc nét trên nền Light Mode.
14. Kiểm tra quyền chỉnh sửa ca trực không gây crash JavaScript.
15. Kiểm tra bố cục bảng không bị vỡ trên màn hình desktop độ phân giải chuẩn.

---

## 9. Implementation notes for AI code generation

- **Preserved Layout**: Giữ nguyên cấu trúc HTML bảng phân ca trực và bảng SLA cards.
- **Defensive Event Binding**: Đảm bảo các hàm toggle ca trực có guard check phần tử trước khi thay đổi class.

---

## 10. Final implementation assumptions to review

- Kênh xuất bản lịch trực mặc định là `#cs-announcements`.

---

## 11. User Stories

### Story: US-01 — CS Lead phân ca trực và xuất bản lịch lên Slack
**As a** CS Lead  
**I want to** phân công ca trực tuần cho toàn bộ nhân sự CS/Dev trên bảng Roster và bấm xuất bản lên Slack  
**So that** toàn bộ team nắm rõ lịch làm việc và phân công ca trực trong tuần mới.

#### Acceptance Criteria (Gherkin)
- **Given** CS Lead đang ở màn hình Duty Shift Roster  
  **When** CS Lead click vào ô ca trực của `Aylin Tran` vào Thứ Hai để chọn ca `Morning 08-16`  
  **And** CS Lead bấm nút `Publish Roster to #cs-announcements`  
  **Then** Hệ thống xuất bản lịch trực lên Slack và hiển thị Toast thông báo hoàn tất.
