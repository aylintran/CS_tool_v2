# Theme Architecture & SDS Rules

Hệ thống Design System của dự án tuân thủ nghiêm ngặt mô hình phân tách Theme (Primitives) và Mode (Semantics):

- **Theme (Primitives)**: Chứa các mã màu cơ bản (VD: `Brand-100`). Bộ default là `Value`. Các bộ custom themes gồm `Mode`, `Morning Dew`, `Ash Blue`, `Emerald Waltz`, v.v... Kích hoạt qua attribute `[data-theme="..."]`.
- **Mode (Semantics)**: Chứa logic giao diện (VD: `Background-Primary`). Chỉ có 2 mode là `SDS Light` và `SDS Dark`. Kích hoạt qua attribute `[data-mode="light"]` và `[data-mode="dark"]`.

## Nguyên tắc tối thượng
1. **Không Hardcode HEX vào Semantics**: Biến Semantic (trong `semantics.css`) BẮT BUỘC phải tham chiếu đến biến Primitive (VD: `var(--sds-color-brand-100)`). TUYỆT ĐỐI KHÔNG dùng mã HEX (VD: `#FFFFFF`) vì nó sẽ làm hỏng cơ chế đổi Theme!
2. **Xử lý file JSON mới**: Khi gen CSS từ file JSON Figma, phải luôn kiểm tra `targetVariableName` để map Semantics về Primitives thay vì trích xuất mã HEX. Primitives sẽ được lưu vào `primitives.css` hoặc `custom-themes.css` dưới dạng `[data-theme="tên-theme"]`.


# UI Development & Core JS Safety Rules (P0 Critical)

1. **Tuyệt đối không Hard-code Màu sắc**: Khi code layout hoặc file `index.css` (hoặc các component UI), tuyệt đối không được hard-code các giá trị màu sắc (ví dụ: `#FFFFFF`, `#2C2C2C`) hay các biến CSS cũ. Bắt buộc phải sử dụng hệ thống biến Semantic Design System (SDS) variables (ví dụ: `var(--sds-color-background-default-default)`).
2. **Thay đổi Màu sắc/Theme KHÔNG ĐƯỢC làm thay đổi Core JS hoặc Cấu trúc DOM**:
   - Việc thay đổi giao diện, chế độ sáng/tối (Light/Dark Mode) chỉ được phép can thiệp vào CSS / Style / Classes hoặc CSS Variables.
   - **CẢNH BÁO LỖI NGHIÊM TRỌNG**: Tuyệt đối **KHÔNG ĐƯỢC** đổi tên `id` của thẻ HTML (ví dụ: sửa `admin-apps-grid` thành `admin-apps-list` hoặc thay đổi cấu trúc thẻ mà JS đang bind). Việc làm đứt gãy liên kết giữa DOM ID và JavaScript là lỗi nghiêm trọng mức độ P0, sẽ gây ra Fatal `TypeError: null.innerHTML`, ngắt toàn bộ luồng khởi tạo và làm sập toàn bộ ứng dụng trên client/server.
3. **Áp dụng Defensive Programming cho mọi thao tác DOM**:
   - Mọi hàm JavaScript thao tác với DOM (`getElementById`, `querySelector`) đều phải có guard check kiểm tra `if (element)` hoặc thông qua hàm an toàn `safeSetHTML(id, content)` để chống crash dây chuyền.
