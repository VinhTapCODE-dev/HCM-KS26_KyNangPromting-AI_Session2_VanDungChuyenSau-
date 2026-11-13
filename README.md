## Bước 1: Thiết kế giải pháp (I/O)

**Input:**

*Tin nhắn của khách: văn bản viết tự do.*
*Menu cửa hàng: tên món, size, giá.*

**Output:**

a) *JSON gồm các mục:*

`is_order`: có phải đơn hàng không (đúng/sai)
`customer`: tên, số điện thoại, số điện thoại hợp lệ hay không, địa chỉ
`items`: danh sách món (tên, size, số lượng, đơn giá, trạng thái)
`total`: tổng tiền
`missing_fields`: danh sách thông tin còn thiếu
`need_confirm`: có cần nhân viên xác nhận lại không

b) *Bảng: Món | Size | SL | Đơn giá | Thành tiền | Trạng thái.*

## Bước 2: Viết prompt và chạy thử

 **CONTEXT:**
Cửa hàng trà sữa nhận đơn qua tin nhắn Zalo. Khách viết tự do, hay viết tắt,
sai chính tả, thiếu thông tin. Kết quả sẽ nhập thẳng vào hệ thống.

 **REQUEST:**
Đọc TIN NHẮN KHÁCH ở cuối prompt và rút ra: tên, số điện thoại, địa chỉ,
các món (tên, size, số lượng).

 **EXPLANATION:**
MENU (chỉ bán các món này):
- Trà sữa trân châu: M 30000 | L 35000
- Trà đào cam sả: M 32000 | L 38000
- Hồng trà: M 25000 | L 30000

Quy tắc:
- Thiếu thông tin -> để null, không tự đoán.
- Món không có trong menu -> status = "KHONG_CO_TRONG_MENU".
- Món có trong menu nhưng thiếu size hoặc số lượng -> status = "THIEU_THONG_TIN".
- Món đủ thông tin -> status = "OK".
- Số lượng mơ hồ ("vài", "mấy") -> quantity = null.
- SĐT hợp lệ = đúng 10 số, bắt đầu bằng 0. Sai -> phone_valid = false.
- Không phải tin đặt hàng -> is_order = false, các mục còn lại để trống.
- Sai chính tả, viết tắt -> sửa về đúng tên món trong menu.
- total chỉ tính khi mọi món đều có status = "OK", ngược lại total = null.
- Có chỗ thiếu hoặc sai -> need_confirm = true.

 **ACTION:**
1. Kiểm tra có phải đặt hàng không.
2. Rút thông tin khách và từng món.
3. Đối chiếu menu, tính tiền theo quy tắc.
4. Liệt kê mục còn thiếu vào missing_fields.

 **TONE:**
Ngắn gọn, chính xác, không suy đoán.

**EXTRAS:**
**Ví dụ 1:**
Tin nhắn: "Nam 0901234567, 12 Nguyễn Huệ, 2 trà sữa trân châu size L"
Kết quả:
{"is_order":true,"customer":{"name":"Nam","phone":"0901234567","phone_valid":true,"address":"12 Nguyễn Huệ"},"items":[{"name":"Trà sữa trân châu","size":"L","quantity":2,"unit_price":35000,"line_total":70000,"status":"OK"}],"total":70000,"missing_fields":[],"need_confirm":false}

**Ví dụ 2:**
Tin nhắn: "cho vài ly sinh tố bơ, sdt 0912"
Kết quả:
{"is_order":true,"customer":{"name":null,"phone":"0912","phone_valid":false,"address":null},"items":[{"name":"Sinh tố bơ","size":null,"quantity":null,"unit_price":null,"line_total":null,"status":"KHONG_CO_TRONG_MENU"}],"total":null,"missing_fields":["name","phone","address","size","quantity"],"need_confirm":true}

**Định dạng trả về (chỉ gồm 2 phần này, không viết thêm gì khác):**
PHẦN 1: JSON đúng cấu trúc như ví dụ.
PHẦN 2: Bảng gồm cột Món | Size | SL | Đơn giá | Thành tiền | Trạng thái.
Trước khi trả lời, kiểm tra lại tổng tiền và không được bịa thông tin.

 **TIN NHẮN KHÁCH**
Lan 0987654321, 45 Lê Lợi, lấy 1 tra dao cam sa size M với 2 hong tra size L
