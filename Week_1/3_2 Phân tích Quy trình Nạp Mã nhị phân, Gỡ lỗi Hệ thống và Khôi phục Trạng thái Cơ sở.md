**Đề tài:** Nghiên cứu mã nguồn mở Semtech USP trên nền tảng Zephyr RTOS tích hợp giao thức LoRa/LoRaWAN

**Hệ thống mục tiêu (Target Architecture):** Vi điều khiển Seeed Studio XIAO nRF54L15 (MCU) kết hợp bộ thu phát vô tuyến Semtech LR2021

**Môi trường thực thi (Host OS):** Fedora 44 (x86_64)

**Giai đoạn Nghiên cứu:** Giai đoạn 3.2 - Giao tiếp phần cứng cấp thấp, thực nghiệm chẩn đoán và can thiệp chuỗi công cụ (Toolchain)

**Nghiên cứu viên:** Junior Embedded Systems R&D Engineer

**Cố vấn chuyên môn:** Senior Embedded Systems + LoRa/LoRaWAN R&D Mentor

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
    

## 2. Nhật ký Thực nghiệm: Quá trình Xác minh và Loại trừ Giả thuyết Lỗi (The R&D Dead-ends)

Sau khi quá trình nạp mã báo cáo thành công, hệ thống rơi vào trạng thái ngưng phản hồi ngoại vi (màn hình OLED tối đen, hệ thống diode phát quang TX/RX tắt hoàn toàn). Nghiên cứu viên đã tiến hành xây dựng và thực nghiệm các giả thuyết kiểm chứng nhằm phân lập lỗi.

### 2.1. Giả thuyết 1: Sự cố phần cứng vật lý và Cơ chế phục hồi qua Bootloader (Bị loại trừ)

- **Nội dung giả thuyết:** Hệ thống gặp lỗi nghiêm trọng (Kernel Panic) hoặc phân vùng ứng dụng mới xung đột với ROM, làm cô lập giao tiếp cổng COM. Cần cưỡng bức vi điều khiển quay về môi trường nạp an toàn bằng tổ hợp phím Reset phần cứng (Hardware Recovery).
    
- **Phương pháp thực nghiệm:** Thực hiện kích hoạt chu kỳ ngắt kép (Double-Click Hard Reset) trên phím bấm vật lý của cấu trúc Seeed Studio XIAO nhằm ép chip nRF54L15 gọi phân vùng UF2 Bootloader mặc định để giả lập ổ đĩa Mass Storage.
    
- **Kết quả thực nghiệm:** Hệ thống vi điều khiển hoàn toàn không phản hồi, diode không chuyển sang trạng thái nhấp nháy định thời (breathing mode), không xuất hiện ngoại vi lưu trữ mới trên Host.
    
- **Kết luận kỹ thuật:** Dòng chip kiến trúc mới nRF54L15 của Nordic trên cấu hình bo mạch này đã cấu hình khóa hoặc loại bỏ cơ chế nạp kéo thả UF2 thô sơ qua USB. Tiến trình giao tiếp bộ nhớ phụ thuộc hoàn toàn vào mạch nạp SWD thông qua chip chuyển đổi tích hợp trên Evaluation Kit. Giả thuyết lỗi kết nối vật lý bị loại trừ.
    

### 2.2. Giả thuyết 2: Hiện tượng Treo Hạt nhân (Kernel Panic) do Device Tree (Bị loại trừ)

- **Nội dung giả thuyết:** Hệ điều hành Zephyr RTOS cấu hình sai lệch các chân ngoại vi (SPI Bus, ngắt GPIO) trong tệp `.overlay`, dẫn đến việc driver của Semtech LR2021 kích hoạt lỗi hệ thống mức thấp, phong tỏa luồng CPU ứng dụng ngay tại pha khởi động (`SYS_INIT`).
    
- **Phương pháp thực nghiệm:** Sử dụng công cụ giám sát chuỗi dữ liệu nối tiếp `picocom` tại tốc độ baud `115200` qua cổng vật lý `/dev/ttyACM0` nhằm thu thập tín hiệu chẩn đoán thô (Raw Serial Log) ngay khi hệ thống nhận lệnh khởi động lại (Hard Reset).
    
- **Nhật ký Hệ thống (Serial Log thu thập thực tế):**
    
    ```
    *** Booting Zephyr OS build v4.2.0 ***
    [00:00:00.007,122] <inf> usp: ===== Ping Pong example =====
    [00:00:00.007,142] <inf> usp: Starting loop...
    [00:00:00.045,057] <inf> lorawan: Defined Hook IDs:
    ...
    [00:00:00.045,399] <inf> usp: receiving...
    [00:00:00.059,136] <inf> usp: usp/rac: transaction is starting
    ```
    
- **Phân tích kết quả:** Chuỗi log hiển thị thông báo liên kết hạt nhân `*** Booting Zephyr OS ***` và tiến trình đăng ký hàm băm thành công (`Defined Hook IDs`). Điều này chứng minh cấu trúc vi nhân (micro-kernel) vận hành ổn định, giao tiếp bus SPI vật lý giữa MCU và Transceiver hoàn toàn bình thường. Sự cố màn hình tối đen chỉ là do ứng dụng `ping_pong` mẫu không tích hợp trình điều khiển đồ họa (Display Driver) trong vòng lặp chính. Giả thuyết Kernel Panic bị loại trừ.
    

### 2.3. Biện lý kết luận Nguyên nhân Cốt lõi (Root Cause)

Sự cố hệ thống không phát sóng phát sinh từ **Trạng thái nghẽn luồng thực thi (Blocking Wait State)** quy định bởi kiến trúc phần mềm Semtech USP.

Dòng log `usp/rac: transaction is starting` xác nhận FSM (Machine State) đang gọi mô-đun **RAC (Radio Abstraction Config)**. Mô-đun này tạo ra một rào cản logic ngắt luồng (blocking event), phong tỏa vòng lặp truyền nhận sóng RF để ngóng đợi gói dữ liệu định dạng tham số vô tuyến (Tần số, Hệ số trải phổ, Công suất phát) được truyền xuống từ phần mềm LoRa Studio máy Host. Trong điều kiện thiếu vắng kết nối này, vi điều khiển rơi vào trạng thái chờ vô hạn.

## 3. Quy trình Khôi phục Hệ thống và Các rào cản can thiệp Chuỗi công cụ (Toolchain)

Nhằm phục vụ công tác đối chiếu tham chiếu, nghiên cứu viên quyết định hoàn nguyên thiết bị về firmware gốc **Hardware Modem** (`LoRaStudio_nrf54l15_xiao_v1.5.2.elf`). Tiến trình hoàn nguyên ghi nhận các xung đột công cụ và cách thức cô lập xử lý:

### 3.1. Ngõ cụt Can thiệp độc lập bằng `pyocd standalone` (Thất bại)

- **Phương pháp:** Thực thi lệnh nạp trực tiếp thông qua bộ cài Python gốc trên máy Host:
    
    ```
    pyocd flash LoRaStudio_nrf54l15_xiao_v1.5.2.elf -t nrf54l15
    ```
    
- **Hạn chế kỹ thuật:** Trình `pyocd` báo lỗi từ chối `Target type nrf54l15 not recognized`. Do dòng chip nRF54L15 quá mới, cơ sở dữ liệu mặc định của pyocd độc lập thiếu tệp cấu hình bản đồ bộ nhớ (CMSIS-Pack). Lệnh `west flash` trước đó thực hiện được do Zephyr tự động liên kết qua tệp cấu hình trung gian `runners.yaml`.
    

### 3.2. Ngõ cụt Can thiệp bằng Chỉ thị Tham số West (Thất bại)

- **Phương pháp:** Gọi trình điều phối của Zephyr nhằm ép buộc nạp tệp nhị phân ngoại lai thông qua cờ chỉ định:
    
    ```
    west flash --skip-rebuild --elf-file LoRaStudio_nrf54l15_xiao_v1.5.2.elf
    ```
    
- **Hạn chế kỹ thuật:** Trình runner `pyocd` nội bộ của West hoàn toàn bỏ qua tham số ngoại lai, tự động định tuyến về tệp `build/zephyr/zephyr.hex` hiện hữu của ứng dụng Ping-Pong và báo trạng thái `skipped`.
    

### 3.3. Giải pháp Khắc phục: Kỹ thuật Thay thế Khối dữ liệu Nhị phân (Payload Substitution Hack)

Để hóa giải sự cứng nhắc của trình điều phối `west` mà vẫn khai thác được tập lệnh định tuyến bộ nhớ chuẩn xác của nó, giải pháp can thiệp cấu trúc tệp mã máy ở mức thấp đã được áp dụng thành công.

**Bước 1: Chuyển đổi định dạng mã máy (Binary Translation)**

Sử dụng công cụ biên dịch ngược `objcopy` từ kiến trúc GCC Toolchain của Zephyr SDK nhằm bóc tách dữ liệu thực thi từ tệp `.elf` của Semtech, mã hóa về định dạng cấu trúc Intel HEX và ghi đè trực tiếp, giả mạo tệp đầu ra mặc định của dự án:

```
~/.local/zephyr-sdk-0.16.8/arm-zephyr-eabi/bin/arm-zephyr-eabi-objcopy -O ihex LoRaStudio_nrf54l15_xiao_v1.5.2.elf build/zephyr/zephyr.hex
```

**Bước 2: Kích hoạt Nạp cưỡng bức**

Ra lệnh cho West thực thi tiến trình tương tác phần cứng, bỏ qua giai đoạn phân tích cấu trúc cấu hình CMake:

```
west flash --skip-rebuild
```

**Kết quả thực nghiệm:** Trình điều phối pyocd bị đánh lừa logic, ghi nhận cấu trúc nhị phân có sự thay đổi lớn nên hủy bỏ trạng thái `skipped`, thực hiện chu trình `Erasing` và `Programming` đạt **100%**. Bo mạch quay về trạng thái Hardware Modem tiêu chuẩn, kết nối thành công với phần mềm LoRa Studio trên môi trường Windows.

## 4. TỔNG KẾT KỸ THUẬT VÀ ĐỊNH HƯỚNG NGHIÊN CỨU

### 4.1. Bài học Kinh nghiệm (Lessons Learned)

1. **Giá trị của các thực nghiệm thất bại:** Việc kiểm chứng chu kỳ ngắt kép (Double Reset) hay các lệnh tham số thất bại của West đã giúp định hình chính xác biên giới hoạt động của chip nRF54L15 và trình nạp pyocd, rút ngắn thời gian chẩn đoán cấu hình cho các thành viên trong Lab.
    
2. **Nguyên tắc Tham chiếu Cơ sở (Single Source of Truth):** Không bao giờ đặt lòng tin kỹ thuật vào biểu hiện ngoại vi trực quan (OLED/LED). Nhật ký dòng lệnh từ Serial Console là dữ liệu duy nhất phản ánh trung thực bản chất logic của hạt nhân hệ điều hành.
    

### 4.2. Định hướng Giai đoạn 4 (Next Action Items)

- Tái cấu trúc mã nguồn `main.c` của dự án mẫu vô tuyến.
    
- Vô hiệu hóa (Bypass) các hàm gọi kiểm tra giao tiếp RAC (`usp_rac_...()`).
    
- Nạp cứng (Hardcode) các hằng số vô tuyến để triển khai Thiết bị Nút Tự trị (Standalone Node).