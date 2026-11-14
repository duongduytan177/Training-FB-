# SF_valve – Van chuyển hướng 2 ngả

## 1. Chức năng

`SF_valve` là FB điều khiển cơ cấu van chuyển hướng có hai vị trí trái/phải, sử dụng hai cảm biến hành trình. Cơ cấu di chuyển sang trái hoặc phải để lựa chọn cửa xả tương ứng.

Theo quy ước công nghệ:
- Cơ cấu sang trái (`Sensor_Left`) → mở cửa xả bên phải.
- Cơ cấu sang phải (`Sensor_Right`) → mở cửa xả bên trái.

FB hỗ trợ điều khiển Auto/Manual, dừng/chạy lại, giám sát thời gian di chuyển và tổng hợp lỗi.

## 2. Giao diện FB

### Input

| Tên | Kiểu | Mô tả |
|---|---|---|
| `Enable` | Bool | Cho phép FB hoạt động |
| `Interlock` | Bool | Điều kiện liên động bên ngoài |
| `External_fault` | Bool | Lỗi bên ngoài; được ghi nhận vào `button_fault` |
| `Run` | Bool | Xóa trạng thái dừng đã chốt |
| `Stop` | Bool | Kích hoạt trạng thái dừng |
| `Reset` | Bool | Xóa các cờ lỗi nội bộ |
| `Auto` | Bool | Yêu cầu chọn chế độ Auto |
| `Manual` | Bool | Yêu cầu chọn chế độ Manual |
| `RightCmd` | Bool | Lệnh Auto di chuyển sang phải |
| `LeftCmd` | Bool | Lệnh Auto di chuyển sang trái |
| `Button_Right` | Bool | Nút Manual di chuyển sang phải |
| `Button_Left` | Bool | Nút Manual di chuyển sang trái |
| `Sensor_Right` | Bool | Cảm biến xác nhận cơ cấu ở bên phải |
| `Sensor_Left` | Bool | Cảm biến xác nhận cơ cấu ở bên trái |

### Output

| Tên | Kiểu | Mô tả |
|---|---|---|
| `Fault` | Bool | Tổng hợp trạng thái lỗi |
| `Right` | Bool | Đầu ra điều khiển di chuyển sang phải |
| `Left` | Bool | Đầu ra điều khiển di chuyển sang trái |
| `Busy` | Bool | Đầu ra khai báo trạng thái bận (hiện chưa được gán trong code) |

## 3. Tham số và dữ liệu nội bộ

UDT `valve_sf` được sử dụng cho dữ liệu cấu hình và trạng thái nội bộ.

| Thành viên | Kiểu | Mặc định / mô tả |
|---|---|---|
| `TimeOut` | Time | `T#5s`, thời gian giám sát di chuyển |
| `Auto` | Bool | Chế độ Auto đã chọn |
| `Manual` | Bool | Chế độ Manual đã chọn |
| `Run` | Bool | Điều kiện cho phép hoạt động: `Enable AND Interlock` |
| `Right`, `Left` | Bool | Trạng thái điều khiển/trạng thái vị trí được tổng hợp trong FB |
| `FaultRight` | Bool | Lỗi Timeout hướng phải |
| `FaultLeft` | Bool | Lỗi Timeout hướng trái |
| `AutoConflict` | Bool | Hai lệnh Auto trái/phải cùng ON |
| `Interlock_fault` | Bool | Lỗi liên động |
| `button_fault` | Bool | Cờ lỗi từ `External_fault` |

## 4. Nguyên lý hoạt động

### Điều kiện hoạt động

FB cho phép hoạt động khi `Enable` và `Interlock` cùng ON. Khi `Interlock` OFF, cờ `Interlock_fault` được đặt. Các cờ lỗi được xóa bằng `Reset`.

### Chọn chế độ

- Auto được chọn khi `Auto = 1`, `Manual = 0`, đủ điều kiện chạy và chưa có Fault.
- Manual được chọn khi `Manual = 1`, `Auto = 0`, đủ điều kiện chạy và chưa có Fault.
- Nếu hai tín hiệu chọn chế độ cùng ON hoặc cùng OFF, FB không cập nhật lựa chọn chế độ mới.

### Auto

- `RightCmd` điều khiển đầu ra `Right`.
- `LeftCmd` điều khiển đầu ra `Left`.
- Nếu `RightCmd` và `LeftCmd` cùng ON, FB chốt `AutoConflict`.
- Hai timer `TonRight` và `TonLeft` giám sát thời gian đầu ra tương ứng khi chế độ Auto đang được chọn. Hết `TimeOut` mà cảm biến được kiểm tra trong chương trình chưa tác động thì đặt cờ lỗi hướng tương ứng.

### Manual

- `Button_Right` điều khiển hướng phải khi `Button_Left` không ON.
- `Button_Left` điều khiển hướng trái khi `Button_Right` không ON.
- Nếu hai nút cùng ON, không phát sinh lệnh Manual từ hai biến nhớ nút.

### Stop / Run

- `Stop` đặt chốt `Stop_state`.
- `Run` xóa chốt `Stop_state`.
- Khi `Stop_state` đang ON, hai đầu ra `Right` và `Left` bị ngắt.

### Tổng hợp lỗi

`Fault` được tạo từ các cờ:
- `FaultLeft`
- `FaultRight`
- `AutoConflict`
- `button_fault`
- `Interlock_fault`

## 5. Lưu ý khi tích hợp

- Trong chương trình hiện tại, `Config.Right` được tổng hợp từ `Right` hoặc `Sensor_Left`; `Config.Left` được tổng hợp từ `Left` hoặc `Sensor_Right`. Cách ánh xạ này phản ánh quy ước cơ cấu sang trái mở cửa phải và ngược lại.
- `Busy` hiện được khai báo ở `VAR_OUTPUT` nhưng chưa có phép gán trong FB.
- `Reset` xóa các cờ lỗi được liệt kê trong code; nếu nguyên nhân lỗi vẫn còn, một số cờ có thể được đặt lại ở chu kỳ quét tiếp theo.
- Nhánh điều khiển Manual hiện không kiểm tra `NOT Fault` như nhánh Auto. Cần xác nhận đây có phải chủ ý thiết kế trước khi đưa vào vận hành.
- Cần kiểm tra lại điều kiện cảm biến trong hai timer Timeout với sơ đồ đấu dây và quy ước vị trí thực tế trước khi chạy máy.
- `Stop` trong code là chốt dừng điều khiển; chức năng dừng khẩn cấp an toàn phải được thực hiện bằng phần cứng/mạch an toàn phù hợp, không thay thế bằng logic FB thông thường.

## 6. Tệp đi kèm

- `SF_valve.scl`: mã nguồn FB điều khiển van.
- `valve_sf.udt`: kiểu dữ liệu cấu hình và trạng thái của FB.
