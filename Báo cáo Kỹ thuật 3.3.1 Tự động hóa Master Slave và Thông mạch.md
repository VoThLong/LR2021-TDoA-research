# BÁO CÁO KỸ THUẬT CHI TIẾT: CẤU HÌNH MÔI TRƯỜNG, ĐIỀU HƯỚNG MÁY TRẠNG THÁI (FSM) VÀ TỰ ĐỘNG HÓA CHUỖI CÔNG CỤ (TOOLCHAIN) TRÊN HỆ THỐNG LR2021 VÀ XIAO NRF54L15

**Đề tài:** Phân tích kiến trúc đa kho lưu trữ (Multi-repo Workspace), giải phẫu Máy trạng thái vô tuyến và chuẩn hóa quy trình biên dịch/nạp tự động cho mạng LoRa Point-to-Point.
**Hệ thống mục tiêu:** Seeed Studio XIAO nRF54L15 (MCU) kết hợp bộ thu phát Semtech LR2021.
**Môi trường Host:** Fedora 44 (x86_64).
**Kỹ sư R&D:** VO THANH LONG & Gemini CLI Assistant.

---

## 1. PHÂN TÍCH KIẾN TRÚC MÔI TRƯỜNG LÀM VIỆC ĐA KHO LƯU TRỮ (WORKSPACE ARCHITECTURE)

Dự án này được thiết kế với tư duy tách biệt hoàn toàn giữa nền tảng lõi (Core Framework) và mã nguồn ứng dụng nghiên cứu (Out-of-tree Application). Cụ thể, không gian làm việc được chia thành hai phân vùng chính:

### 1.1. Phân vùng Nền tảng Lõi (The Upstream Workspace)
*   **Đường dẫn:** `/home/dashtrad/lora_usp_workspace`
*   **Nguồn gốc:** Thư mục này được khởi tạo (clone) từ kho lưu trữ GitHub chính thức của Semtech (`https://github.com/Lora-net/usp_zephyr.git`).
*   **Vai trò:** Đây là trái tim của hệ thống, chứa tệp định tuyến `west.yml`. Nó quản lý lõi hệ điều hành Zephyr RTOS, ngăn xếp giao thức vô tuyến LoRa Basics Modem (LBM), các trình điều khiển lớp trừu tượng phần cứng (HAL) của Nordic Semiconductor và Semtech.
*   **Môi trường độc lập:** Thư mục này chứa môi trường ảo Python (`.venv`) đi kèm các công cụ điều phối `west` và bộ nạp phần cứng `pyocd`.

### 1.2. Phân vùng Ứng dụng Nghiên cứu (Out-of-Tree Project)
*   **Đường dẫn:** `/home/dashtrad/Documents/lr2021-tdoa-firmware`
*   **Vai trò:** Chứa toàn bộ logic ứng dụng tùy chỉnh (`src/main.c`), các tệp tin Device Tree Overlay tùy biến và tệp cấu hình biên dịch (`CMakeLists.txt`, `prj.conf`).
*   **Thách thức kỹ thuật:** Trình điều phối `west` mặc định chỉ hoạt động bên trong không gian chứa thư mục ẩn `.west`. Do phân vùng ứng dụng nghiên cứu nằm hoàn toàn bên ngoài, chúng ta bắt buộc phải sử dụng các lệnh điều hướng tuyệt đối (Absolute Path Redirection) và khai báo biến môi trường `ZEPHYR_BASE` để `west` có thể tham chiếu ngược lại Phân vùng Nền tảng Lõi khi tiến hành biên dịch.

---

## 2. GIẢI PHẪU MÁY TRẠNG THÁI PING-PONG VÀ KỸ THUẬT CAN THIỆP MÃ NGUỒN

Ứng dụng mẫu Ping-Pong của Semtech được vận hành dựa trên một Máy trạng thái hữu hạn (Finite State Machine - FSM).

### 2.1. Phân tích nguyên nhân nghẽn luồng truyền thông (Deadlock Analysis)
Theo thiết kế gốc, hệ thống yêu cầu một nút bấm vật lý (Physical Button) để kích hoạt chế độ Chủ quản (Manager/Master). Nếu không phát hiện tín hiệu ngắt (Interrupt) từ nút bấm, tất cả các nút mạng đều mặc định tự gán vai trò Cấp dưới (Subordinate/Slave) và gọi hàm `smtc_rac_submit_radio_transaction()` để chuyển bộ khuếch đại nhiễu thấp (LNA) sang trạng thái chờ thu (RX Mode). Việc cả hai thiết bị cùng chờ đợi một gói tin "PING" không bao giờ xuất hiện đã dẫn đến vòng lặp hết hạn (RX Timeout) vô tận.

### 2.2. Kỹ thuật Can thiệp bằng Phần mềm (Software-Triggered FSM Injection)
Do thiết bị đánh giá thực nghiệm (Evaluation Kit) không hỗ trợ hoặc không cấu hình đúng nút bấm vật lý, kỹ thuật giả lập sự kiện (Event Simulation) đã được áp dụng trực tiếp vào tệp `/src/main.c`.

**Bước 1: Khai báo cờ điều khiển tĩnh**
Định nghĩa một Macro điều khiển tại phần đầu của tệp nguồn:
```c
#define FORCE_MASTER_MODE 1  // Giá trị 1 chỉ định vai trò Master, giá trị 0 chỉ định vai trò Subordinate.
```

**Bước 2: Cưỡng bức chuyển đổi trạng thái FSM**
Ngay sau pha khởi tạo các dịch vụ cốt lõi (`ping_pong_init()`), hệ thống kiểm tra Macro và trực tiếp can thiệp vào biến bộ nhớ của vòng lặp sự kiện:
```c
if( FORCE_MASTER_MODE )
{
    LOG_INF( "!!! FORCING MASTER MODE ON STARTUP !!!" );
    user_button_is_press = true;
}
```
Biến `user_button_is_press` bị ép chuyển thành `true` sẽ đánh lừa vòng lặp sự kiện chính (`while(1)`), khiến hệ thống tin rằng người dùng vừa nhấn nút. Hệ thống ngay lập tức gọi hàm `ping_pong_on_button_press()`.

**Bước 3: Quy trình nội tại của hàm `ping_pong_on_button_press()`**
1.  Gán `ping_pong.is_manager = true`.
2.  Thực thi `smtc_rac_abort_radio_submit()` để cưỡng bức ngắt quá trình chờ thu (RX) hiện tại của vi mạch LR2021.
3.  Phát lệnh cấu hình Radio sang chế độ truyền (TX) và đẩy gói dữ liệu PING đầu tiên ra dải tần không trung (Air).

---

## 3. THIẾT KẾ KỊCH BẢN TỰ ĐỘNG HÓA BIÊN DỊCH VÀ NẠP PHẦN CỨNG (BASH SCRIPTING)

Để loại bỏ hoàn toàn các sai số do thao tác thủ công (Human Error) trong việc thay đổi cấu hình mã nguồn và biên dịch cho hai thiết bị khác nhau, hai kịch bản điều phối (`flash_master.sh` và `flash_subordinate.sh`) đã được xây dựng. Dưới đây là giải phẫu chi tiết luồng thực thi của kịch bản Master:

### 3.1. Sửa đổi mã nguồn động (Dynamic Stream Editing)
```bash
sed -i 's/#define FORCE_MASTER_MODE [0-1]/#define FORCE_MASTER_MODE 1/g' "$PROJECT_ROOT/src/main.c"
```
Lệnh `sed` (Stream Editor) tìm kiếm chuỗi định nghĩa Macro và thay thế trực tiếp giá trị của nó thành `1` (Master) ngay trên tệp đĩa cứng trước khi hệ thống biên dịch đọc vào. Điều này bảo đảm tệp nhị phân sắp được tạo ra mang đúng logic Master.

### 3.2. Tiêm biến môi trường (Environment Variables Injection)
```bash
source "$VENV_PATH/bin/activate"
source "$ZEPHYR_ENV"
export ZEPHYR_BASE="/home/dashtrad/lora_usp_workspace/zephyr"
```
Hệ thống nạp môi trường ảo Python chứa công cụ `west`, đồng thời khai báo đường dẫn `ZEPHYR_BASE`. Đây là chìa khóa để lệnh `west build` (chạy tại phân vùng Out-of-tree) có thể xác định được nhân Zephyr và Toolchain `arm-zephyr-eabi` từ phân vùng Nền tảng Lõi.

### 3.3. Biên dịch có phân lập cấu hình (Pristine Build with Shield)
```bash
west build -b xiao_nrf54l15/nrf54l15/cpuapp --shield semtech_wio_lr2021 -p always
```
*   Tham số `--shield semtech_wio_lr2021` là tham số bắt buộc. Nó điều hướng trình biên dịch CMake nạp bổ sung tệp Device Tree Overlay của bo mạch mở rộng Wio LR2021. Nếu thiếu tham số này, chip nRF54L15 sẽ không thể xác định bản đồ định tuyến tín hiệu chân SPI giao tiếp với chip Radio.
*   Tham số `-p always` kích hoạt quy trình dọn dẹp bộ đệm (Pristine). Nó xóa bỏ hoàn toàn tệp `CMakeCache.txt` cũ, ngăn chặn tình trạng hệ thống sử dụng lại tệp nhị phân đã biên dịch từ lần chạy trước.

### 3.4. Gỡ lỗi thuật toán trình nạp và Nạp cưỡng bức (Forced Mass Erase Flashing)
```bash
west flash -i $1 -- --erase
```
*   Tham số `-i $1` nhận vào Unique ID của mạch đích (Ví dụ: `290EB85B`), giúp định tuyến dữ liệu chính xác khi có nhiều mạch cắm cùng lúc vào cổng USB.
*   Thuật toán đối chiếu bộ nhớ (Memory Diffing Algorithm) của trình nạp `pyocd` vốn tự động bỏ qua quá trình ghi nếu phát hiện tệp nhị phân mới có cấu trúc tương đồng với dữ liệu đang lưu trong bộ nhớ Flash của vi điều khiển (thông báo lỗi `skipped`). Để phá vỡ cơ chế này, tham số `-- --erase` được truyền vào. Cặp dấu gạch ngang `--` ra lệnh cho công cụ `west` chuyển thẳng tham số `--erase` xuống cho trình `pyocd`, yêu cầu thực hiện hành động Mass Erase (xóa sạch toàn bộ sector bộ nhớ) trước khi nạp tệp `.hex`.

---

## 4. QUY TRÌNH KIỂM CHỨNG TÍN HIỆU VÔ TUYẾN THÔNG QUA GIAO DIỆN NỐI TIẾP (UART DIAGNOSTICS)

Quá trình thu thập và phân tích nhật ký giao tiếp nối tiếp (Serial Log) được thực hiện bằng cách khởi chạy song song các quy trình nền trên hệ điều hành Linux Host.

### 4.1. Lệnh giám sát hệ thống
```bash
stty -F /dev/ttyACM0 115200 raw -echo && cat /dev/ttyACM0 > /tmp/log_acm0.txt & PID0=$!
stty -F /dev/ttyACM1 115200 raw -echo && cat /dev/ttyACM1 > /tmp/log_acm1.txt & PID1=$!
sleep 15
kill $PID0 $PID1
```
*   Lệnh `stty -F` cấu hình giao thức truyền nhận trên cổng `/dev/ttyACM0` và `ACM1` về tốc độ 115200 baud, định dạng dữ liệu thô (`raw`) và tắt chế độ dội âm (`-echo`) để tránh nhiễu tín hiệu.
*   Lệnh `cat` tiến hành đọc liên tục luồng dữ liệu từ bộ đệm thiết bị ngoại vi và chuyển hướng (`>`) đầu ra vào tệp văn bản.
*   Toán tử `&` đưa tiến trình đọc vào chế độ chạy ngầm (Background Task) và gán ID của tiến trình đó vào biến `PID0` / `PID1`. Lệnh `kill` sau đó sẽ tự động tiêu diệt các tiến trình này sau 15 giây thu thập dữ liệu.

### 4.2. Phân tích kết quả Nhật ký Thực thi
*   **Trên mạch Master (`/dev/ttyACM0`):** Hệ thống ghi nhận các chuỗi log `[inf] usp: sending (value=18, delay=250ms)` và `[inf] usp: usp/rac: new event: transaction (reception) has ended (success)`. Điều này chứng minh thiết bị đã kích hoạt PA phát gói tin và thu nhận thành công gói tin phản hồi từ môi trường không khí. Khi `value` đạt ngưỡng 24 (tương ứng hằng số `MAX_EXCHANGE_COUNT = 25`), hệ thống ngắt phiên giao tiếp và khởi động lại vòng lặp Máy trạng thái theo thiết kế an toàn.
*   **Trên mạch Subordinate (`/dev/ttyACM1`):** Hệ thống liên tục ghi nhận `[inf] usp: payload is correct, continuing` và lập tức phản hồi bằng `[inf] usp: sending (value=18, delay=50ms)`.

**KẾT LUẬN TỔNG THỂ:** Cơ chế Máy trạng thái Ping-Pong và luồng dữ liệu vô tuyến (RF Data Path) thông qua phần cứng LR2021 đã được đả thông hoàn toàn nhờ vào quy trình can thiệp cấu trúc và tự động hóa chuỗi công cụ đã mô tả. Đây là điều kiện tiên quyết (Prerequisite) vững chắc để dự án tiến tới tích hợp thuật toán định vị độ trễ thời gian (TDoA).
