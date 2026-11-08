# FB_Tank – Khối giám sát mức liệu bồn chứa

## 1. Chức năng
FB_Tank thực hiện giám sát mức liệu trong bồn chứa thông qua hai cảm biến Rotary Paddle Level Switch:
- **Low Level:** Phát hiện mức liệu thấp.
- **High Level:** Phát hiện mức liệu cao.

Khối FB có nhiệm vụ xử lý tín hiệu cảm biến, xác định trạng thái mức liệu và phát hiện lỗi bất thường.

## 2. Nguyên lý hoạt động

### 2.1. Điều kiện hoạt động
- `Latch_Ready = Enable AND Interlock`.
- Khi đủ điều kiện hoạt động, FB tiếp nhận và xử lý tín hiệu cảm biến.
- Khi mất Enable hoặc Interlock, FB giữ nguyên trạng thái cảm biến đã xác nhận trước đó.

### 2.2. Xử lý tín hiệu cảm biến
Sử dụng TON để xác nhận trạng thái ON/OFF của từng cảm biến:
- `TonHighOn`, `TonHighOff`: Xử lý tín hiệu cảm biến mức cao.
- `TonLowOn`, `TonLowOff`: Xử lý tín hiệu cảm biến mức thấp.

Thời gian xác nhận được thiết lập thông qua `HighDelay` và `LowDelay`.

### 2.3. Xác định trạng thái mức liệu

| Low Sensor | High Sensor | Trạng thái |
|---|---|---|
| 0 | 0 | Empty – Dưới mức Low |
| 1 | 0 | Valid – Mức liệu bình thường |
| 1 | 1 | Full – Đạt mức High |
| 0 | 1 | Invalid – Trạng thái bất thường |

### 2.4. Phát hiện lỗi
- `LowSensor_Fault`: Lỗi cảm biến mức thấp từ cấu hình phần cứng.
- `HighSensor_Fault`: Lỗi cảm biến mức cao từ cấu hình phần cứng.
- `LevelConflict`: Phát hiện trạng thái High = 1 và Low = 0, được xác nhận bằng TON `TonConflict`.
- `Interlock_Fault`: Mất điều kiện Interlock khi Enable vẫn được kích hoạt.

Lệnh `Reset` dùng để xóa các cờ lỗi được lưu trong Config.

## 3. Input / Output chính

| Tên biến | Chức năng |
|---|---|
| `Enable` | Cho phép FB hoạt động |
| `Interlock` | Điều kiện liên động |
| `Reset` | Xóa trạng thái lỗi |
| `InSensorHigh` | Tín hiệu cảm biến mức cao |
| `InSensorLow` | Tín hiệu cảm biến mức thấp |
| `Empty` | Bồn dưới mức Low |
| `Valid` | Mức liệu bình thường |
| `Full` | Bồn đạt mức High |
| `Fault` | Lỗi tổng hợp |

## 4. Nguyên tắc thiết kế
- Giữ nguyên trạng thái cảm biến đã xác nhận khi FB mất điều kiện hoạt động.
- Sử dụng TON để hạn chế ảnh hưởng của nhiễu tín hiệu.
- Tách biệt trạng thái mức liệu và trạng thái lỗi.
- FB chỉ thực hiện giám sát mức liệu, không trực tiếp điều khiển thiết bị cấp liệu hoặc xả liệu.
