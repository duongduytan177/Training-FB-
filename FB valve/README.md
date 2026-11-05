# SL_valve -- Khối điều khiển van Auto/Manual

## 1. Giới thiệu

`SL_valve` là Function Block được viết bằng SCL cho PLC Siemens S7, dùng
để điều khiển một van hai chiều thông qua hai relay `Open` và `Close`,
với hai chế độ vận hành Auto và Manual.

Khối có các chức năng: - Điều khiển van ở chế độ Auto hoặc Manual. -
Giám sát thời gian di chuyển của van trong chế độ Auto. - Phát hiện và
chốt lỗi khi van không đến vị trí yêu cầu. - Phát hiện xung đột khi đồng
thời có lệnh mở và đóng ở chế độ Auto. - Dừng khẩn cấp, khóa hệ thống và
yêu cầu thao tác chạy lại. - Cung cấp trạng thái van cho HMI và các
Function Block khác.

## 2. Tóm tắt hoạt động

-   **Auto:** Nhận lệnh `OpenCmd` và `CloseCmd` từ các FB bên ngoài.
-   **Manual:** Điều khiển trực tiếp bằng cách giữ nút `Button_Open`
    hoặc `Button_Close`.
-   **Stop:** Ngắt ngay hai relay và khóa hệ thống. Sau khi nhả Stop, hệ
    thống chỉ hoạt động lại khi có thao tác `Run` hoặc `Rest`.
-   **Fault:** Lỗi được chốt bằng `LatchFault`; phải thực hiện `Rest`
    mới xóa được lỗi.

## 3. Ngõ vào và ngõ ra

### 3.1. Ngõ vào

  -----------------------------------------------------------------------
  Tên                                 Ý nghĩa
  ----------------------------------- -----------------------------------
  `Enable`, `Interlock`               Điều kiện cho phép chạy; cả hai
                                      phải bằng 1.

  `Auto`, `Manual`                    Tín hiệu chọn chế độ vận hành.

  `Stop`                              Dừng khẩn cấp.

  `Run`                               Nút nhấn chạy tiếp sau khi Stop; sử
                                      dụng dạng xung.

  `Rest`                              Nút nhấn reset lỗi; sử dụng dạng
                                      xung.

  `OpenCmd`, `CloseCmd`               Lệnh mở/đóng từ khối bên ngoài, sử
                                      dụng trong Auto.

  `Button_Open`, `Button_Close`       Nút nhấn mở/đóng tại chỗ, sử dụng
                                      trong Manual.

  `Sensor_open`, `Sensor_close`       Cảm biến xác nhận vị trí mở/đóng
                                      của van.
  -----------------------------------------------------------------------

### 3.2. Ngõ ra

  Tên               Ý nghĩa
  ----------------- ----------------------------------------------
  `Open`, `Close`   Tín hiệu điều khiển relay mở/đóng van.
  `Busy`            Tín hiệu interlock cung cấp cho các FB khác.
  `Fault`           Báo lỗi đã được chốt.

### 3.3. Biến trạng thái `Figure`

`Figure` có kiểu dữ liệu `valve`, cung cấp trạng thái cho HMI và các FB
liên quan:

  -----------------------------------------------------------------------
  Biến                                Ý nghĩa
  ----------------------------------- -----------------------------------
  `Figure.Open`                       Van đã được cảm biến xác nhận ở vị
                                      trí mở.

  `Figure.Close`                      Van đã được cảm biến xác nhận ở vị
                                      trí đóng.

  `Figure.Auto`                       Trạng thái chế độ Auto hiện tại.

  `Figure.Manual`                     Trạng thái chế độ Manual hiện tại.

  `Figure.FaultOpen`                  Lỗi mở van.

  `Figure.FaultClose`                 Lỗi đóng van.

  `Figure.AutoConflict`               Xung đột lệnh mở và đóng trong
                                      Auto.

  `Figure.TimeOut`                    Thời gian timeout cấu hình cho quá
                                      trình di chuyển van.
  -----------------------------------------------------------------------

## 4. Chi tiết hoạt động từng chế độ

### 4.1. Chọn chế độ

Việc chọn chế độ chỉ được thực hiện khi: - `Enable = 1`. -
`Interlock = 1`. - Không có `Stop`.

Quy tắc chọn chế độ:

-   `Auto = 1`, `Manual = 0` và không có lỗi chốt: chuyển sang Auto.
-   `Manual = 1`, `Auto = 0`: chuyển sang Manual. Có thể vào Manual ngay
    cả khi đang có lỗi để xử lý bằng tay.
-   Cả hai cùng bằng 1 hoặc cùng bằng 0: giữ nguyên chế độ hiện tại.
-   Khi chuyển chế độ, các lệnh thuộc chế độ cũ được xóa.

### 4.2. Chế độ Auto

-   Khi chỉ có `OpenCmd`, khối cấp relay `Open`.
-   Nếu sau khoảng thời gian `Figure.TimeOut` mà `Sensor_open` chưa tác
    động, khối báo `FaultOpen` và ngắt relay.
-   Quá trình đóng van hoạt động tương tự: sử dụng `CloseCmd`,
    `Sensor_close` và báo `FaultClose` nếu hết thời gian mà chưa xác
    nhận vị trí.
-   Nếu `OpenCmd` và `CloseCmd` cùng có hiệu lực, khối báo
    `AutoConflict` và ngắt cả hai relay.
-   `Figure.Open` hoặc `Figure.Close` chỉ bằng 1 khi có lệnh hợp lệ
    tương ứng và cảm biến xác nhận vị trí.

### 4.3. Chế độ Manual

-   Giữ `Button_Open` để điều khiển mở van hoặc giữ `Button_Close` để
    điều khiển đóng van.
-   Hai nút mở và đóng khóa chéo nhau.
-   Chế độ Manual không sử dụng timeout.
-   `Figure.Open` và `Figure.Close` bám theo trạng thái cảm biến.
-   Các lỗi `FaultOpen`, `FaultClose` và `AutoConflict` thuộc chế độ
    Auto không chặn thao tác Manual.

## 5. Quy trình vận hành

  -----------------------------------------------------------------------
  Tình huống              Thao tác                Kết quả
  ----------------------- ----------------------- -----------------------
  Dừng để kiểm tra        `Stop` → kiểm tra →     Chế độ được giữ; Auto
                          `Run`                   tiếp tục chạy nếu lệnh
                                                  vẫn còn.

  Có lỗi                  `Stop` → `Rest` → chọn  Lỗi và chế độ bị xóa;
                          lại Auto/Manual         cần chọn lại chế độ để
                                                  chạy.

  Có sự cố ở Auto         Chuyển sang Manual,     Có thể điều khiển van
                          không cần reset         bằng nút nhấn tại chỗ.

  Quay lại Auto sau lỗi   `Rest` rồi chọn Auto    Không thể vào Auto khi
                                                  lỗi chưa được reset.

  PLC khởi động lại       Nhấn `Run`              `Stop_state` mặc định
                                                  bằng 1 nên hệ thống
                                                  không tự chạy.
  -----------------------------------------------------------------------

## 6. Các lớp bảo vệ

-   Relay `Open` và `Close` không thể cùng bật:
    -   Auto và Manual loại trừ nhau.
    -   Lệnh mở và đóng trong từng chế độ khóa chéo nhau.
-   `Stop` ngắt relay ngay lập tức và xóa các lệnh.
-   Nhả `Stop` không làm van tự chạy lại; cần thao tác `Run` hoặc
    `Rest`.
-   Lỗi được chốt và không tự xóa khi điều kiện gây lỗi đã hết.
-   Khi mất `Enable` hoặc `Interlock`, `Busy = 0` và hai relay bị ngắt.

## 7. Lưu ý khi sử dụng

1.  `Run` và `Rest` phải là nút nhấn dạng xung. Nếu giữ tín hiệu ở mức
    1, nút `Stop` sẽ không có tác dụng giữ trạng thái dừng như mong
    muốn.
2.  `Figure.TimeOut` phải lớn hơn 0. Nếu bằng 0, timer có thể hết thời
    gian ngay và gây báo lỗi giả.
3.  Relay `Open`/`Close` hiện chưa tự ngắt khi cảm biến báo van đã tới
    vị trí. Cần đảm bảo actuator có limit switch cơ khí bên trong hoặc
    bổ sung điều kiện ngắt relay theo `Sensor_open`/`Sensor_close`.
4.  Trong lúc đang dừng (`Stop`), `Busy` và `Figure.Open`/`Figure.Close`
    giữ giá trị cuối cùng, chưa được cập nhật lại.
5.  Nếu mất `Enable`/`Interlock` trong khi chế độ vẫn được giữ, khi điều
    kiện cho phép trở lại, Auto sẽ tự chạy tiếp nếu `OpenCmd` hoặc
    `CloseCmd` vẫn còn hiệu lực.

## 8. Quy trình Git cơ bản

Nếu quản lý source code của `SL_valve` bằng Git, quy trình cập nhật sau
mỗi lần sửa code là:

``` bash
git status
git add .
git commit -m "Mo ta noi dung thay doi"
git push
```

Ý nghĩa: - `git status`: kiểm tra file đã thay đổi. - `git add .`: đưa
các thay đổi vào Staging Area, bao gồm file mới, file đã sửa và file bị
xóa. - `git commit -m "..."`: tạo commit lưu lại phiên bản thay đổi. -
`git push`: đẩy commit lên remote branch đã liên kết.

Khi thay file cũ bằng file mới trong folder, vẫn dùng `git add .`; Git
sẽ ghi nhận cả thao tác xóa file cũ và thêm file mới. Không cần tạo lại
repository hoặc chạy `git init` cho mỗi lần cập nhật.

Để kiểm tra branch hiện tại:

``` bash
git branch
```

Branch có dấu `*` là branch đang được checkout. Đảm bảo đang ở đúng
branch, ví dụ `SL_valve`, trước khi commit và push.
