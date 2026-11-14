# SL_valve – Khối điều khiển van

## 1. Chức năng
FB `SL_valve` điều khiển van điện bằng hai ngõ ra Open và Close, hỗ trợ hai chế độ Auto/Manual, giám sát trạng thái và phát hiện lỗi trong quá trình vận hành.

## 2. Chế độ hoạt động

- **Auto:** Nhận lệnh `OpenCmd` và `CloseCmd` từ khối điều khiển bên ngoài. Không cho phép đồng thời hai lệnh mở và đóng.
- **Manual:** Điều khiển trực tiếp bằng hai nút `Button_Open` và `Button_Close`.
- **Stop:** Dừng khẩn cấp, khóa các ngõ ra điều khiển. Sử dụng `Run` để cho phép hoạt động trở lại.

## 3. Chức năng giám sát và bảo vệ

- **Interlock:** Điều kiện cho phép FB hoạt động (`Enable AND Interlock`).
- **Timeout:** Giám sát thời gian mở/đóng van, mặc định 5 giây.
- **AutoConflict:** Phát hiện trường hợp đồng thời có lệnh mở và đóng.
- **FaultOpen / FaultClose:** Báo lỗi khi van không đạt trạng thái yêu cầu trong thời gian Timeout.
- **Button_fault:** Nhận lỗi từ tín hiệu `External_Fault`.
- **Reset:** Xóa các cờ lỗi đã ghi nhận.

## 4. Cấu trúc dữ liệu

UDT `valve_sl` quản lý các thông tin nội bộ:

| Nhóm | Biến |
|---|---|
| Chế độ | Auto, Manual |
| Trạng thái | Run, Open, Close |
| Cấu hình | TimeOut |
| Lỗi | FaultOpen, FaultClose, AutoConflict, Button_fault |

## 5. Ngõ vào / Ngõ ra

- **Input:** Enable, Interlock, Auto, Manual, Reset, Stop, Run, OpenCmd, CloseCmd, Button_Open, Button_Close, Sensor_open, Sensor_close, External_Fault.
- **Output:** Open, Close, Busy, Fault.

Trong đó:
- `Open`, `Close`: Tín hiệu điều khiển relay đóng/mở van.
- `Busy`: Tín hiệu điều kiện liên động cho các FB khác.
- `Fault`: Tổng hợp các lỗi của van.

## 6. Công nghệ sử dụng
- PLC Siemens – TIA Portal.
- Ngôn ngữ lập trình SCL.
- Function Block (FB).
- User Defined Type (UDT).
