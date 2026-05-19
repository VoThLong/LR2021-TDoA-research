**Đề tài:** Nghiên cứu mã nguồn mở Semtech USP trên nền tảng Zephyr RTOS tích hợp giao thức LoRa/LoRaWAN **Hệ thống mục tiêu (Target Architecture):** Vi điều khiển Seeed Studio XIAO nRF54L15 (MCU) kết hợp bộ thu phát vô tuyến Semtech LR2021 **Môi trường thực thi (Host OS):** Fedora 44 (x86_64) **Giai đoạn Nghiên cứu:** Giai đoạn 3.2 - Giao tiếp phần cứng cấp thấp, phân tích nhật ký hệ điều hành và can thiệp chuỗi công cụ (Toolchain) **Nghiên cứu viên:** Junior Embedded Systems R&D Engineer **Cố vấn chuyên môn:** Senior Embedded Systems + LoRa/LoRaWAN R&D Mentor

## 1. Giao tiếp Phần cứng và Quy trình Nạp Mã thực thi (Firmware Flashing)

Sau giai đoạn biên dịch thành công ứng dụng mẫu `ping_pong` trên môi trường Zephyr, quá trình chuyển giao mã thực thi (flashing) xuống thiết bị vật lý đòi hỏi việc thiết lập giao thức truyền thông USB mức thấp và khai thác bộ nạp phần cứng (Hardware Debugger) được tích hợp sẵn.

### 1.1. Cấu hình định tuyến và phân quyền truy cập ngoại vi (Udev Rules)

Theo mô hình bảo mật tiêu chuẩn của nhân (kernel) Linux, các tiến trình thuộc không gian người dùng (user space) không được cấp quyền thao tác trực tiếp với các khối thiết bị USB ngoại vi. Nhằm tự động hóa quá trình cấp quyền mà không cần leo thang đặc quyền (privilege escalation) cục bộ, các quy tắc Udev tĩnh đã được thiết lập:

```
sudo cp application/boards/seeed/xiao_nrf54l15/support/99-xiao-nrf54l15.rules /etc/udev/rules.d/
sudo udevadm control --reload-rules
sudo udevadm trigger
```

### 1.2. Phân tích Tiến trình Nạp tự động (Flash Runner Analysis)

Quá trình ghi cấu trúc nhị phân `zephyr.hex` vào vi điều khiển đích được thực thi thông qua lệnh điều phối của Zephyr:

```
west flash
```

**Phân tích nhật ký nạp mã (Flash Log Diagnostics):**

1. **Khởi tạo bộ nạp (Auto-Runner Selection):** Hệ thống Zephyr tự động định tuyến và kích hoạt thư viện **`pyocd`** — một mô-đun Python chuyên dụng cho giao thức SWD (Serial Wire Debug) trên lõi kiến trúc ARM Cortex-M33. Nhờ tích hợp phần cứng nạp (On-board Debugger) trên bo mạch mở rộng LoRa Plus, thao tác thiết lập Bootloader vật lý (phím nhấn phần cứng) được loại bỏ.
    
2. **Cảnh báo Không gian bảo mật (TrustZone Status):** Thông báo `NRF54L15 is not in a secure state` xác nhận tính năng phân vùng bộ nhớ vật lý ARM TrustZone đang hoạt động. Do đặc thù thử nghiệm, tiến trình cấp phát ứng dụng R&D được chỉ định thực thi tại vùng nhớ không bảo mật (Non-Secure environment).
    
3. **Thuật toán tối ưu hóa Chu kỳ ghi (Flash Optimization):** Ghi nhận `skipped 118784 bytes` thể hiện thuật toán đối chiếu bộ nhớ (Memory Diffing Algorithm) của trình nạp đã xác định tệp `zephyr.hex` trùng khớp hoàn toàn với cấu trúc dữ liệu hiện tại trên vi mạch NVM (Non-Volatile Memory), từ đó tự động bỏ qua chu trình Xóa/Ghi nhằm tối ưu hóa tuổi thọ phần cứng.
    

## 2. Hiện tượng Đánh lừa Giao diện (UI Trap) và Kỹ thuật Phân tích Nhật ký Hệ thống

### 2.1. Biểu hiện ngưng phản hồi ngoại vi (Hardware Unresponsiveness)

Hậu kỳ nạp firmware `ping_pong`, hệ thống phần cứng mục tiêu ghi nhận các trạng thái phản hồi phi tiêu chuẩn:

- Màn hình OLED không hiển thị thông tin khởi tạo ("Please connect me to LoRa Studio...").
    
- Diode phát quang (LED) biểu thị xung truyền/nhận vô tuyến (TX/RX) trong trạng thái tĩnh.
    
- _Giả thuyết lâm sàng:_ Xảy ra hiện tượng lỗi hạt nhân (Kernel Panic) do định tuyến sai giao diện bus nối tiếp (SPI/I2C) trong mô tả cấu trúc phần cứng (Device Tree).
    

### 2.2. Phương pháp Giám sát qua Giao diện Nối tiếp (Serial Console Monitoring)

Nhằm xác minh trạng thái thực thi của hệ điều hành, phương pháp phân tích chuỗi dữ liệu thô (Raw Log Analysis) qua giao diện UART ảo (USB CDC ACM) được áp dụng, thiết lập cơ sở tham chiếu độc lập (Single Source of Truth) thay cho việc quan sát tín hiệu thị giác.

**Thiết lập tham số Terminal trên Host:**

```
# Triển khai gói công cụ picocom
sudo dnf install picocom -y
# Khởi tạo kết nối Serial tại tốc độ baud 115200
sudo picocom -b 115200 /dev/ttyACM0
```

**Nhật ký Khởi động (Boot Log) thu thập sau khi thiết lập lại phần cứng (Hard Reset):**

```
*** Booting Zephyr OS build v4.2.0 ***
[00:00:00.007,122] <inf> usp: ===== Ping Pong example =====
[00:00:00.007,142] <inf> usp: Starting loop...
[00:00:00.045,057] <inf> lorawan: Defined Hook IDs:
...
[00:00:00.045,399] <inf> usp: receiving...
[00:00:00.059,136] <inf> usp: usp/rac: transaction is starting
```

### 2.3. Chẩn đoán Nguyên nhân Cốt lõi (Root Cause Analysis)

Việc đối chiếu Boot Log đã bác bỏ hoàn toàn các giả thuyết về sự cố phần cứng cấp thấp:

1. **Giao tiếp phần cứng ổn định:** Hệ điều hành khởi động thành công. Chuỗi log `Defined Hook IDs` minh chứng cho việc giao thức bus SPI giữa vi điều khiển nRF54L15 và bộ thu phát Semtech LR2021 được thiết lập toàn vẹn. Hiện tượng ngưng hiển thị OLED bắt nguồn từ việc ứng dụng `ping_pong` không tích hợp trình điều khiển đồ họa (Display Driver) trong vòng lặp thực thi chính.
    
2. **Trạng thái nghẽn luồng thực thi (Blocking Wait State):** Nguồn gốc sự cố xuất phát từ thiết kế luồng của thư viện Semtech USP. Khai báo `usp/rac: transaction is starting` xác nhận ứng dụng đang gọi mô-đun **RAC (Radio Abstraction Config)**. Mô-đun này khởi tạo một vòng lặp chờ vô tận (infinite blocking loop) để tiếp nhận các cấu hình vô tuyến (Tần số, Hệ số trải phổ, Công suất phát) từ phần mềm phân tích (LoRa Studio). Trong điều kiện thiếu vắng nguồn cấp dữ liệu cấu hình, Máy trạng thái (FSM) rơi vào trạng thái ngưng trệ (deadlock) logic, ngăn chặn hệ thống chuyển tiếp sang pha truyền/nhận sóng RF.
    

## 3. Quy trình Khôi phục Firmware (System Reversion) và Can thiệp Chuỗi công cụ (Toolchain Manipulation)

Nhằm đảm bảo khả năng tương thích của thiết bị với các bài kiểm thử cơ sở định chuẩn, quyết định khôi phục lại cấu trúc firmware ban đầu **Hardware Modem** (Tệp `LoRaStudio_nrf54l15_xiao_v1.5.2.elf` do Semtech phát hành) được thông qua. Quá trình này ghi nhận những giới hạn hệ thống từ trình quản lý Toolchain và yêu cầu các phương pháp can thiệp mức thấp.

### 3.1. Phương pháp tiếp cận 1: Trình nạp độc lập `pyocd` (Không khả thi)

- **Lệnh thực thi:** `pyocd flash LoRaStudio_nrf54l15_xiao_v1.5.2.elf -t nrf54l15`
    
- **Lỗi trả về:** `Target type nrf54l15 not recognized`.
    
- **Phân tích kỹ thuật:** Thư viện pyocd độc lập (standalone) trên máy Host chưa được cấu hình gói định nghĩa kiến trúc (CMSIS-Pack) cho dòng vi điều khiển nRF54L15. Trong khi đó, lệnh `west flash` trước đó khởi chạy thành công nhờ cơ chế sử dụng tệp mô tả ẩn `runners.yaml` chứa các tham số bản đồ bộ nhớ nội bộ của Zephyr.
    

### 3.2. Phương pháp tiếp cận 2: Sử dụng trình Runner của Zephyr (Không khả thi)

- **Lệnh thực thi:** `west flash --skip-rebuild --elf-file LoRaStudio_nrf54l15_xiao_v1.5.2.elf`
    
- **Lỗi trả về:** Trình quản lý bỏ qua tệp `.elf` tham số, tiến hành lặp lại chu trình nạp tệp `zephyr.hex` và báo lỗi `skipped 118784 bytes`.
    
- **Phân tích kỹ thuật:** Trình runner `pyocd` được thiết kế cấu hình tĩnh (hardcoded) trong môi trường Zephyr nhằm thiết lập độ ưu tiên tuyệt đối cho tệp output biên dịch mặc định, do đó từ chối nạp cấu trúc bộ nhớ ngoại lai.
    

### 3.3. Giải pháp Khắc phục: Kỹ thuật Thay thế Khối dữ liệu Nhị phân (Payload Substitution Hack)

Nhằm khắc phục những giới hạn phần mềm của trình quản lý `west` trong khi vẫn bảo toàn được tính toàn vẹn của tệp định tuyến SWD chuẩn xác, phương pháp can thiệp tệp nhị phân thông qua biên dịch ngược được thực nghiệm.

**Bước 1: Trích xuất và Chuyển đổi định dạng mã máy (Binary Translation)** Ứng dụng trình liên kết `objcopy` từ Zephyr SDK để trích xuất khối dữ liệu thực thi từ tệp `.elf` của Semtech, chuyển đổi về định dạng chuẩn Intel HEX và **thay thế trực tiếp** tệp output mặc định của dự án Zephyr:

```
~/.local/zephyr-sdk-0.16.8/arm-zephyr-eabi/bin/arm-zephyr-eabi-objcopy -O ihex LoRaStudio_nrf54l15_xiao_v1.5.2.elf build/zephyr/zephyr.hex
```

**Bước 2: Cưỡng bức Nạp mã thực thi** Khởi chạy lệnh nạp thông qua `west` với chỉ thị bỏ qua chu trình biên dịch tệp CMake:

```
west flash --skip-rebuild
```

**Kết quả thực nghiệm:** Ghi nhận hệ thống tiến hành xóa định mức bộ nhớ (Erasing) và ghi chuỗi dữ liệu mới (Programming) đạt 100%. Thiết bị mục tiêu được khôi phục thành công về trạng thái vận hành xuất xưởng, đảm bảo tương thích hoàn toàn với nền tảng phần mềm LoRa Studio.

## 4. TỔNG KẾT KỸ THUẬT VÀ ĐỊNH HƯỚNG NGHIÊN CỨU

### 4.1. Bài học Kinh nghiệm (Lessons Learned)

1. **Nguyên tắc Tham chiếu Cơ sở (Rule of Embedded Debugging):** Trong kiến trúc RTOS đa nhiệm, sự ngưng trệ của các tín hiệu ngoại vi (màn hình hiển thị, đèn LED) không hoàn toàn phản ánh trạng thái đóng băng của hệ thống phần cứng. Giao diện UART Console đóng vai trò là cơ sở tham chiếu duy nhất (Single Source of Truth) để đánh giá trạng thái vi nhân (micro-kernel) và máy trạng thái (FSM).
    
2. **Kỹ năng Kiểm soát Chuỗi công cụ (Toolchain Mastery):** Việc thấu hiểu kiến trúc của các định dạng tệp thực thi nhị phân (`.elf`, `.hex`) và khả năng vận dụng các công cụ thao tác bộ nhớ tĩnh (điển hình như `objcopy`) cung cấp giải pháp kỹ thuật vượt ra ngoài giới hạn đóng gói của hệ thống Framework, duy trì quyền chủ động tối đa đối với quy trình R&D.
    

### 4.2. Định hướng Giai đoạn 4 (Next Action Items)

Việc phân lập sự cố thành công đã cung cấp dữ liệu cơ sở quan trọng về cơ chế hoạt động của thư viện RAC. Để hiện thực hóa mục tiêu triển khai ứng dụng `ping_pong` dưới dạng một Thiết bị Nút Tự trị (Standalone Node) trên nền tảng Zephyr, giai đoạn tiếp theo cần tập trung vào các hạng mục:

- Tái cấu trúc mã nguồn C (`main.c` và thư viện lõi tương ứng).
    
- Vô hiệu hóa (Bypass) các điểm chặn luồng (blocking calls) của hàm cấu hình `usp_rac_...()`.
    
- Thiết lập Cấu hình Tĩnh (Static Configuration) cho các tham số vô tuyến cơ sở (Radio Parameters) trực tiếp vào máy trạng thái hệ thống.