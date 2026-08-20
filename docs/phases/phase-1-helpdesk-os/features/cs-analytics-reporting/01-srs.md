---
feature_slug: cs-analytics-reporting
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

# SRS: CS Analytics & Operational Reporting (`cs-analytics-reporting`)

## 1. Purpose
Cung cấp tài liệu tả chi tiết yêu cầu phần mềm cho màn hình **CS Analytics (`📈 CS Analytics`)** trong Internal Web App. Màn hình cung cấp bức tranh dữ liệu phân tích vận hành CS theo thời gian: số lượng ticket tạo mới/đã resolve, thời gian xử lý trung bình (Median Resolution Time), tỷ lệ leo thang lên Dev (Escalation Rate), biểu đồ phân bổ trạng thái/độ khẩn cấp và bảng hiệu suất tuân thủ cam kết SLA.

---

## 2. Scope

### 2.1 In Scope
- **Bộ Lọc Khoảng Thời Gian (Date Range Filter)**: `Last 7 days`, `Last 14 days`, `Last 30 days`, `Custom range`.
- **Hàng Thẻ Chỉ Số Cốt Lõi (Core Metric Stat Cards)**:
  - `CREATED`: Tổng số ticket khởi tạo trong kỳ.
  - `RESOLVED`: Số ticket đã giải quyết xong.
  - `MEDIAN RESOLUTION TIME`: Thời gian xử lý trung bình (e.g. `6h 12m`).
  - `ESCALATION RATE`: Tỷ lệ ticket cần chuyển qua Dev xử lý (%).
- **Chỉ Số Phụ (Secondary Metrics)**: First Response Time (p50 / p95) và Tỷ lệ vi phạm SLA (SLA Breach Rate).
- **Hệ Thống Biểu Đồ Trực Quan (Charts Section)**:
  - **Chart 1 — Tickets by Status**: Biểu đồ cột phân bổ ticket theo 12 trạng thái (nhóm Dev, CS, Follow-up).
  - **Chart 2 — Tickets by Urgency Level**: Phân bổ độ khẩn cấp (`Urgent` red, `High` amber, `Medium` blue, `Low` slate).
  - **Chart 3 — Volume Over Time**: Xu hướng Ticket Created vs Resolved theo ngày.
  - **Chart 4 — Top CS Agents Performance**: Xếp hạng năng suất nhân viên CS.
  - **Chart 5 — SLA Performance Table**: Bảng đánh giá chi tiết tỷ lệ đạt/vi phạm SLA theo từng Urgency Level.

### 2.2 Out of Scope
- Dự báo tự động lượng ticket bằng machine learning trong tương lai.

---

## 3. Key Business Rules

- **3.1. Date & App Scoping**: Tất cả biểu đồ và thẻ chỉ số trên màn hình CS Analytics phải đồng bộ theo khoảng thời gian được chọn và lọc theo ứng dụng (APO/APB/ACS).
- **3.2. SLA Breach Calculation**: Ticket bị tính vi phạm SLA (`Breach`) nếu thời gian Resolve thực tế vượt quá thời gian cam kết của Urgency Level tương ứng (Urgent 15m, High 1h, Medium 4h, Low 24h).

---

## 4. QA Scenarios

1. Kiểm tra chuyển đổi bộ lọc `Last 7 days`, `14 days`, `30 days` làm toàn bộ thẻ chỉ số và biểu đồ cập nhật dữ liệu tương ứng.
2. Kiểm tra hiển thị đúng 4 thẻ chỉ số chính (CREATED, RESOLVED, MEDIAN RESOLUTION TIME, ESCALATION RATE).
3. Kiểm tra biểu đồ phân bổ trạng thái hiển thị chuẩn 12 custom statuses.
4. Kiểm tra bảng SLA Performance tính toán chính xác tỷ lệ Met SLA % và Breach count.
