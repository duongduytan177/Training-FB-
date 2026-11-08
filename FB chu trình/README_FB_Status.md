# FB_Status – Khối quản lý trạng thái thiết bị

## 1. Chức năng

`FB_Status` quản lý trình tự khởi động, vận hành và dừng thiết bị theo mô hình State Machine. FB hỗ trợ hai chế độ Auto/Manual, giám sát tín hiệu phản hồi chạy và quản lý lỗi bằng cơ chế Fault Latch.

Các kiểu dữ liệu đi kèm:
- `eISM_State`: Khai báo các mã trạng thái.
- `stISM_Status`: Cấu trúc dữ liệu trạng thái cung cấp ra ngoài FB.

## 2. Các trạng thái

| Mã | Tên trạng thái | Ý nghĩa |
|---:|---|---|
| 0 | Idle | Thiết bị không hoạt động |
| 10 | Starting | Đang thực hiện khởi động |
| 20 | Running | Thiết bị đang chạy |
| 30 | Stopping | Đang thực hiện dừng |
| 100 | Error | Thiết bị đang ở trạng thái lỗi |

Khi `Enable = FALSE`, FB đưa `State` và `NextState` về `Idle`, xóa `StateInit` và reset các timer.

## 3. Chế độ vận hành

### 3.1. Auto Mode

Khi `AutoMode = TRUE`, FB điều khiển thiết bị theo State Machine:

- **Idle:** Nhận yêu cầu `CmdStart`. Nếu `Interlock = TRUE`, chuyển sang `Starting`; nếu không, chốt lỗi Interlock.
- **Starting:** Bật `DoStart`, chờ `FB_Running = TRUE`. Nếu quá `StartTimeOut`, phát sinh lỗi Start Timeout. Có thể nhận `CmdStop` để chuyển sang `Stopping`.
- **Running:** Duy trì `DoStart`. Nếu mất `Interlock` hoặc mất phản hồi `FB_Running`, FB chốt lỗi. Nhận `CmdStop` thì chuyển sang `Stopping`.
- **Stopping:** Bật `DoStop`, chờ `FB_Running = FALSE`. Nếu quá `StopTimeOut`, phát sinh lỗi Stop Timeout.
- **Error:** Tắt `DoStart`, bật `DoStop`; chờ xử lý và Reset lỗi.

`CmdStart` chỉ được nhận khi `CmdStop = FALSE`.

### 3.2. Manual Mode

Khi `AutoMode = FALSE`:

- `DoStart` được điều khiển trực tiếp bởi `CmdStart`, `Interlock`, trạng thái lỗi và `ExternalFault`.
- `DoStop` được điều khiển trực tiếp bởi `CmdStop`.
- Trạng thái nội bộ được cập nhật theo phản hồi `FB_Running`: có phản hồi thì `Running`, không có phản hồi thì `Idle`.

## 4. Quản lý lỗi

Các lỗi được chốt bằng `FaultLatched` và lưu mã lỗi trong `FaultCode`.

| FaultCode | Nguyên nhân |
|---:|---|
| 0 | Không có lỗi |
| 1 | Interlock không thỏa mãn khi có yêu cầu khởi động hoặc bị mất khi đang chạy |
| 2 | Khởi động quá thời gian `StartTimeOut` |
| 3 | Lỗi bên ngoài `ExternalFault` |
| 4 | Mất phản hồi `FB_Running` khi đang ở trạng thái Running |
| 5 | Dừng quá thời gian `StopTimeOut` |

`CmdResetFault` xóa `FaultLatched` và đưa `FaultCode` về 0. Nếu đang ở trạng thái Error, `NextState` được yêu cầu chuyển về Idle. Nguyên nhân lỗi bên ngoài vẫn còn có thể làm lỗi được chốt lại ở chu kỳ quét tiếp theo.

## 5. Timer và cơ chế StateInit

- `tonTimeout`: Giám sát thời gian khởi động hoặc dừng, tùy trạng thái.
- `tonStateTime`: Đo thời gian khi `Enable` đang TRUE; giá trị `ET` được đưa ra `stStatus.StateTime`.
- `StateInit`: Cờ khởi tạo trạng thái, bảo đảm thao tác khởi tạo và kích hoạt timer chỉ thực hiện khi mới vào trạng thái.
- Khi `NextState` khác `State`, FB lưu `PrevState`, cập nhật `State`, xóa `StateInit` và reset timer để chuẩn bị cho trạng thái tiếp theo.

## 6. Input / Output chính

### Input

| Tên biến | Chức năng |
|---|---|
| `Enable` | Cho phép FB hoạt động |
| `AutoMode` | Chọn Auto hoặc Manual |
| `CmdStart` | Yêu cầu khởi động |
| `CmdStop` | Yêu cầu dừng |
| `CmdResetFault` | Reset lỗi |
| `FB_Running` | Phản hồi thiết bị đang chạy |
| `Interlock` | Điều kiện liên động cho phép chạy |
| `ExternalFault` | Tín hiệu lỗi từ bên ngoài |
| `StartTimeOut` | Thời gian giám sát khởi động, mặc định 5 giây |
| `StopTimeOut` | Thời gian giám sát dừng, mặc định 5 giây |

### Output

| Tên biến | Chức năng |
|---|---|
| `DoStart` | Lệnh khởi động thiết bị |
| `DoStop` | Lệnh dừng thiết bị |
| `stStatus` | Cấu trúc trạng thái và thông tin chẩn đoán |

Các trường chính trong `stStatus` gồm `State`, `PrevState`, `StateName`, `StateTime`, `Auto`, `Manual`, `Busy`, `Running`, `FaultActive`, `FaultLatched` và `FaultCode`.

## 7. Lưu ý triển khai

- `FB_Running` là tín hiệu phản hồi trạng thái thực tế từ thiết bị; cần đấu nối và cấu hình đúng nguồn tín hiệu.
- `Interlock` phải được quy ước `TRUE` là đủ điều kiện cho phép chạy.
- `DoStart` và `DoStop` là lệnh điều khiển đầu ra của FB; việc đấu nối đến thiết bị thực tế được thực hiện ở chương trình cấp trên.
- `FaultText` trong `stISM_Status` hiện được khai báo nhưng chưa được gán giá trị trong đoạn code.
- Trong Auto Mode, trạng thái được xử lý theo State Machine; trong Manual Mode, lệnh Start/Stop được xuất trực tiếp theo các điều kiện đã lập trình.
