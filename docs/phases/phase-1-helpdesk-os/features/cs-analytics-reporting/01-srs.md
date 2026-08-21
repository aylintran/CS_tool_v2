---
feature_slug: cs-analytics-reporting
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

# SRS: CS Analytics & Operational Reporting (`cs-analytics-reporting`)

## 1. Purpose
Tài liệu Đặc tả Yêu cầu Phần mềm (SRS) cho màn hình **CS Analytics** trong Internal Web App thuộc Helpdesk OS. Tính năng này tổng hợp và phân tích toàn diện các chỉ số vận hành của đội ngũ CS: tổng khối lượng ticket, tỷ lệ giải quyết thành công, thời gian phản hồi đầu tiên (First Response Time), thời gian xử lý trung bình (Resolution Time), phân bổ theo 12 trạng thái custom và 4 cấp độ ưu tiên, cùng bảng xếp hạng năng suất và chỉ số hài lòng CSAT của từng CS Agent.

---

## 2. Scope

### 2.1 In Scope
- **Date Range Filter Toolbar**:
  - Các mốc chọn nhanh: `Last 7 days`, `Last 14 days`, `Last 30 days`, `Custom Range`.
- **Top 4 Core Metric Stat Cards**:
  - `TOTAL VOLUME`: Tổng số ticket phát sinh trong kỳ (e.g. `142`).
  - `RESOLVED TICKETS`: Số ticket đã giải quyết xong (e.g. `128`).
  - `AVG FIRST RESPONSE`: Thời gian phản hồi đầu tiên trung bình (e.g. `34m`).
  - `AVG RESOLUTION TIME`: Thời gian giải quyết trung bình (e.g. `2h 15m`).
- **Charts & Distribution Section**:
  - **Status Distribution Bars**: Biểu đồ thanh ngang phân bổ khối lượng ticket theo nhóm trạng thái (Dev, CS, Followup, Resolved).
  - **Urgency Breakdown**: Phân bổ tỷ lệ theo 4 mức: `Urgent` (Đỏ), `High` (Vàng cam), `Medium` (Xanh dương), `Low` (Xám).
- **CS Agents Performance Table**:
  - Bảng đánh giá chi tiết từng nhân sự CS (`Aylin Tran`, `Ngan Pham`, `Ha Tran`, `Thao Vo`):
    - `Agent Name`: Tên nhân viên CS.
    - `Tickets Handled`: Số lượng ticket đã xử lý.
    - `Avg First Response`: Thời gian phản hồi trung bình.
    - `CSAT Score`: Điểm hài lòng khách hàng (e.g. `4.9 / 5.0`).
    - `SLA Compliance`: Tỷ lệ tuân thủ cam kết SLA (e.g. `98.2%`).

### 2.2 Out of Scope
- Dự báo lưu lượng ticket trong tương lai bằng mô hình Machine Learning / AI (thuộc Phase 3).

---

## 3. Key Business Rules

- **3.1. Filter Scoping Consistency**: Tất cả dữ liệu thẻ chỉ số, biểu đồ phân bố và bảng hiệu suất CS bắt buộc phải đồng bộ tính toán theo khoảng thời gian và bộ lọc ứng dụng đang chọn.
- **3.2. SLA Compliance Calculation**:
  - Tỷ lệ tuân thủ SLA (%) = `(Số ticket giải quyết trong hạn SLA / Tổng số ticket) * 100`.
  - Hạn SLA tính theo 4 mốc: Urgent (15 phút), High (1 giờ), Medium (4 giờ), Low (24 giờ).
- **3.3. Plain Light Mode UI Rule**: Toàn bộ các thẻ chỉ số và bảng biểu sử dụng nền trắng `#ffffff`, viền xám nhạt `#e2e8f0`, chữ rõ nét và không kèm emoji icons rác.

---

## 4. Domain Model

### 4.1 Entities
- `AnalyticsSummary`:
  - `totalVolume` (integer): Tổng số ticket.
  - `resolvedCount` (integer): Số ticket đóng.
  - `avgFirstResponseMinutes` (integer): Phút phản hồi đầu tiên.
  - `avgResolutionMinutes` (integer): Phút xử lý trung bình.
  - `statusDistribution` (object): Phân bổ số lượng theo trạng thái.
  - `urgencyDistribution` (object): Phân bổ số lượng theo độ khẩn cấp.
- `AgentPerformance`:
  - `agentName` (string): Tên CS Agent.
  - `ticketsHandled` (integer): Số ticket xử lý.
  - `firstResponseTime` (string): Thời gian phản hồi trung bình.
  - `csatScore` (string): Điểm CSAT.
  - `slaCompliance` (string): Tỷ lệ tuân thủ SLA.

---

## 5. Data Model & Validation Rules

| Field | Type | Required | Default | Rule |
| :--- | :--- | :--- | :--- | :--- |
| `date_range` | string | Yes | `Last 7 days` | Phải thuộc: `Last 7 days`, `Last 14 days`, `Last 30 days`, `Custom`. |
| `total_volume` | integer | Yes | `0` | Số nguyên không âm. |
| `resolved_count` | integer | Yes | `0` | Số nguyên không âm, không vượt quá `total_volume`. |
| `csat_score` | numeric(2,1) | Yes | `5.0` | Thang điểm từ `1.0` đến `5.0`. |

---

## 6. State Transitions

- **Đổi mốc thời gian**:
  - `Chọn Date Range mới ➔ Tính toán lại Metric Cards ➔ Re-render Biểu đồ & Bảng Agent`.

---

## 7. Runtime/behavior contract

- **Input**: Bộ lọc khoảng thời gian và mảng `state.tickets`.
- **Output**: Render các giá trị thống kê vào giao diện CS Analytics.
- **Fallback**: Nếu chưa có dữ liệu trong kỳ, hiển thị giá trị `0` và thông báo không có dữ liệu thay vì lỗi runtime.

---

## 8. QA Scenarios

1. Kiểm tra màn hình CS Analytics hiển thị đầy đủ 4 thẻ chỉ số cốt lõi (Total Volume, Resolved, First Response, Resolution Time).
2. Kiểm tra chuyển đổi bộ lọc `Last 7 days`, `14 days`, `30 days` hoạt động trơn tru.
3. Kiểm tra biểu đồ phân bổ trạng thái phản ánh đúng các nhóm trạng thái Dev, CS, Followup, Closed.
4. Kiểm tra phân bổ Urgency hiển thị đủ 4 màu phân biệt (Đỏ, Cam, Xanh, Xám).
5. Kiểm tra bảng CS Performance hiển thị đầy đủ 4 nhân sự CS của team.
6. Kiểm tra các cột trong bảng CS Performance (Tickets Handled, First Response, CSAT Score, SLA Compliance) hiển thị chính xác.
7. Kiểm tra giao diện Light Mode có độ tương phản cao, số liệu dễ đọc.
8. Kiểm tra không có bất kỳ icon emoji nào trong màn hình Analytics.
9. Kiểm tra bảng tính toán tỷ lệ % không bị lỗi chia cho 0 khi dữ liệu rỗng.
10. Kiểm tra thanh cuộn của bảng dữ liệu hoạt động mượt mà.

---

## 9. Implementation notes for AI code generation

- **Tailwind Classes**: Nền thẻ `bg-white border border-slate-200 rounded-xl p-4 shadow-sm`.
- **Defensive Guard**: Kiểm tra null trước khi gán các chỉ số thống kê.

---

## 10. Final implementation assumptions to review

- Chỉ số CSAT được tổng hợp từ khảo sát tự động sau khi ticket chuyển sang `Closed`.

---

## 11. User Stories

### Story: US-01 — CS Lead theo dõi báo cáo hiệu suất vận hành CS
**As a** CS Lead  
**I want to** xem màn hình CS Analytics với các chỉ số tổng quan và bảng hiệu suất chi tiết của từng CS Agent  
**So that** tôi đánh giá được khối lượng công việc, tốc độ phản hồi và chất lượng dịch vụ của toàn đội ngũ.

#### Acceptance Criteria (Gherkin)
- **Given** CS Lead đang ở màn hình CS Analytics  
  **When** CS Lead chọn khoảng thời gian `Last 7 days`  
  **Then** 4 thẻ chỉ số chính hiển thị: `Total Volume: 142`, `Resolved: 128`, `Avg First Response: 34m`, `Avg Resolution Time: 2h 15m`  
  **And** Bảng hiệu suất hiển thị chi tiết chỉ số của `Aylin Tran`, `Ngan Pham`, `Ha Tran`, `Thao Vo`.
