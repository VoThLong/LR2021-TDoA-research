**Đề tài:** Nghiên cứu Nền tảng Zephyr RTOS và Ứng dụng Tích hợp Hệ thống Thu phát Vô tuyến Semtech LR2021 trên Vi điều khiển nRF54L15

**Môi trường triển khai:** Fedora 44 (x86_64)

## 1. PHÂN TÍCH NỀN TẢNG KIẾN TRÚC ZEPHYR RTOS

Zephyr là một hệ điều hành thời gian thực (RTOS) mã nguồn mở, được thiết kế chuyên biệt cho các thiết bị nhúng và Internet vạn vật (IoT) bị giới hạn về tài nguyên. Khác với mô hình phát triển phần mềm nguyên khối (Monolithic), Zephyr áp dụng triết lý phân tách nghiêm ngặt giữa logic ứng dụng, cấu hình phần mềm và đặc tả phần cứng.

### 1.1. Ba Trụ Cột Nền Tảng (Core Pillars)

Để làm chủ việc phát triển trên Zephyr, nghiên cứu viên cần thấu hiểu ba thành phần kiến trúc cốt lõi:

- **Bộ công cụ điều phối `west` (Meta-Tool):** `west` không phải là một hệ thống quản lý phiên bản thay thế cho Git, mà là một công cụ quản lý không gian làm việc đa kho lưu trữ (multi-repo workspace). Thông qua tệp khai báo `west.yml`, nó tự động hóa việc đồng bộ hóa mã nguồn của nhân (kernel) Zephyr, các phân hệ (subsystems), và trình điều khiển (HAL) từ các nhà sản xuất phần cứng khác nhau. Đồng thời, `west` cung cấp giao diện dòng lệnh hợp nhất cho quy trình biên dịch (`west build`) và nạp bộ nhớ nhị phân (`west flash`).
    
- **Hệ thống Trừu tượng hóa Phần cứng - Device Tree (`.dts`, `.overlay`):** Zephyr nghiêm cấm việc định nghĩa cứng (hardcode) các thông số phần cứng (như địa chỉ thanh ghi, số thứ tự chân GPIO) trong mã nguồn C/C++. Mọi cấu trúc liên kết phần cứng được mô tả bằng cấu trúc dữ liệu dạng cây (Device Tree).
    
    - **Tệp `.dts` (Device Tree Source):** Mô tả cấu hình vật lý nội tại của Vi điều khiển (SoC) và Bo mạch chủ.
        
    - **Tệp `.overlay`:** Cung cấp cơ chế ghi đè linh hoạt, cho phép kỹ sư khai báo các thiết bị ngoại vi gắn thêm (Shields) mà không làm thay đổi tệp `.dts` gốc.
        
- **Hệ thống Cấu hình Phân hệ - Kconfig (`prj.conf`):** Kế thừa từ kiến trúc của Linux Kernel, Kconfig chịu trách nhiệm quản lý cấu hình các phân hệ phần mềm (ví dụ: kích hoạt ngăn xếp giao thức mạng, trình điều khiển I2C, cơ chế ghi nhật ký hệ thống). Các tham số được khai báo dưới dạng công tắc logic (`CONFIG_...=y`) trong tệp `prj.conf`, giúp trình biên dịch tối ưu hóa dung lượng ROM/RAM bằng cách loại trừ hoàn toàn các đoạn mã không được kích hoạt.
    

### 1.2. Quy trình Khởi tạo Hệ thống Tiêu chuẩn (Zephyr Getting Started)

Dựa trên tài liệu tham chiếu kỹ thuật chính thức của Zephyr, quy trình thiết lập môi trường phát triển trên nền tảng Linux (Fedora) được thực hiện qua các bước tuần tự như sau:

**Bước 1: Cập nhật và Cài đặt Công cụ Nền tảng (Host OS Dependencies)** Quy trình này cung cấp bộ công cụ thiết yếu để phân tích cú pháp và xây dựng tệp nhị phân, bao gồm `CMake` (công cụ tạo kịch bản biên dịch), `Ninja` (hệ thống biên dịch song song hiệu năng cao) và `dtc` (trình biên dịch Device Tree).

```
sudo dnf install -y cmake ninja-build gperf dfu-util dtc wget \
  python3-pip python3-ply python3-yaml xz-utils file rsync
```

**Bước 2: Phân lập Môi trường và Đồng bộ Mã nguồn (Workspace Initialization)** Nhằm ngăn chặn xung đột không gian tên (namespace) đối với các thư viện Python của hệ điều hành Host, một môi trường ảo (Virtual Environment) được thiết lập. Công cụ `west` sau đó được cài đặt để khởi tạo và đồng bộ hóa toàn bộ kho lưu trữ.

```
python3 -m venv .venv
source .venv/bin/activate
pip install west

# Khởi tạo workspace tại thư mục lora_usp_workspace
west init lora_usp_workspace
cd lora_usp_workspace
west update
```

**Bước 3: Cấu hình Chuỗi công cụ Biên dịch chéo (Zephyr SDK)** Môi trường Host (x86_64) yêu cầu một trình biên dịch chéo (Cross-compiler) để tạo ra tập lệnh thực thi tương thích với kiến trúc ARM Cortex-M. Tập lệnh cài đặt sẽ tự động đăng ký biến môi trường vào hệ thống CMake Registry cục bộ.

```
cd ~
wget [https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz](https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz)
tar xvf zephyr-sdk-0.16.8_linux-x86_64.tar.xz
cd zephyr-sdk-0.16.8
./setup.sh -t arm-zephyr-eabi
```

**Bước 4: Kiểm thử Biên dịch Cơ sở (Blinky Smoke Test)** Để xác thực tính toàn vẹn của hệ thống biên dịch, một ứng dụng kiểm thử cơ bản (blinky) được thực thi. Lệnh này chỉ định trực tiếp kiến trúc đích.

```
cd ~/lora_usp_workspace/zephyr
west build -b xiao_nrf54l15/nrf54l15/cpuapp samples/basic/blinky
```

## 2. ÁNH XẠ KIẾN TRÚC VÀO DỰ ÁN SEMTECH LR2021 (PROJECT MAPPING)

Sau khi thấu hiểu nguyên lý vận hành của Zephyr, bước tiếp theo là tiến hành ánh xạ (mapping) các khái niệm này vào cấu trúc dự án Unified Software Platform (USP) của hãng Semtech.

### 2.1. Luận chứng Chuyển dịch từ Kiến trúc Bare-metal sang Zephyr RTOS

Hãng Semtech cung cấp hai giải pháp tích hợp: Hệ thống không hệ điều hành (**USP Bare-metal**) và Hệ thống dựa trên RTOS (**USP Zephyr**). Việc lựa chọn USP Zephyr mang lại các ưu thế vượt trội về mặt kiến trúc hệ thống:

1. **Loại bỏ Máy trạng thái Kế hoạch Vô tuyến (Radio Planner FSM):** Trong môi trường Bare-metal, việc điều phối các tác vụ có tính thời gian thực ngặt nghèo (như Ranging và LoRaWAN transmission) phải dựa vào một module phần mềm đồng bộ phức tạp gọi là _Radio Planner_. Module này sử dụng các bộ định thời (Hardware Timers) để cấp phát khe thời gian (time-slot). Khi chuyển sang Zephyr, bộ lập lịch (Scheduler) của vi nhân (Microkernel) sẽ tiếp quản nhiệm vụ này thông qua cơ chế quản lý đa luồng (Multi-threading) dựa trên độ ưu tiên (Priority-based preemption), giảm thiểu rủi ro bế tắc luồng (deadlock).
    
2. **Khả năng Chuyển giao Phần cứng (Hardware Portability):** Thay vì lập trình trực tiếp các thanh ghi cấu hình SPI/GPIO (HAL-bound) như trong bản Bare-metal, USP Zephyr sử dụng giao diện API trừu tượng. Sự thay đổi nền tảng vi điều khiển (từ STM32 sang nRF54) chỉ yêu cầu thay đổi tham số truyền vào trong quá trình biên dịch thông qua Device Tree.
    

### 2.2. Phân tích Dòng lệnh Biên dịch Dự án (Build Command Deconstruction)

Xem xét kịch bản biên dịch ứng dụng `ping_pong` thực tế trên phần cứng mục tiêu:

```
west build -b xiao_nrf54l15/nrf54l15/cpuapp samples/usp/sdk/ping_pong -- -DSHIELD=semtech_wio_lr2021
```

Quá trình này kích hoạt bộ phân tích cú pháp (Parser) thực thi các tác vụ sau:

- **`-b xiao_nrf54l15/nrf54l15/cpuapp` (Xác định Kiến trúc Lõi):** SoC nRF54L15 hoạt động trên cơ sở kiến trúc đa nhân. Hậu tố `/cpuapp` điều hướng trình biên dịch tạo ra mã máy (machine code) dành riêng cho _Application Core_ (Cortex-M33). Lõi này sẽ đảm nhận việc thực thi nhân RTOS và điều khiển bộ thu phát vô tuyến.
    
- **`-DSHIELD=semtech_wio_lr2021` (Tiêm tệp Overlay):** Hệ thống tìm kiếm tệp `semtech_wio_lr2021.overlay` tương ứng và thực hiện quá trình hợp nhất (merge) với tệp `.dts` của bo mạch XIAO. Quá trình này tạo ra cầu nối liên kết các chân chức năng (MISO, MOSI, SCK, BUSY, RESET) giữa MCU và vi mạch LR2021 tại thời điểm biên dịch (Compile-time).
    
- **Xử lý Kconfig (`samples/.../prj.conf`):** Các tham số trong tệp này kích hoạt việc nạp (link) thư viện lõi của Semtech (`CONFIG_SMTC_MODEM_HAL=y`) và các trình điều khiển giao thức cấp thấp (`CONFIG_SPI=y`).
    

### 2.3. Quy trình Triển khai và Gỡ lỗi Ngoại vi

Quá trình giao tiếp với phần cứng tại môi trường Linux (Fedora) đòi hỏi sự can thiệp vào hệ thống phân quyền:

- **Cơ chế Udev Rules (`99-xiao-nrf54l15.rules`):** Việc thiết lập các tập luật Udev cho phép tiến trình không đặc quyền (non-root process) của `west` được quyền truy cập trực tiếp vào giao diện USB của mạch nạp (CMSIS-DAP/J-Link), đảm bảo an toàn cho môi trường thực thi.
    
- **Trạm kiểm soát UART (UART Console):** Đối với các ứng dụng RTOS, sự ngưng trệ của một luồng xử lý cụ thể không phản ánh sự cố hệ thống toàn cục (HardFault). Giao tiếp UART xuất nhật ký (System Logging) là phương tiện kỹ thuật duy nhất đáng tin cậy để giám sát trạng thái vòng đời của LoRa Basics Modem (LBM).
    

## 3. TRIỂN KHAI ỨNG DỤNG R&D ĐỘC LẬP (OUT-OF-TREE APPLICATION)

Nhằm phục vụ công tác kiểm thử và phát triển các kịch bản sử dụng (Use-cases) chuyên biệt, nghiên cứu viên cần tách rời việc phát triển khỏi các thư mục mã nguồn mẫu (samples) của nhà sản xuất. Cấu trúc sau đây minh họa một ứng dụng Zephyr độc lập tiêu chuẩn.

### 3.1. Thiết lập Cấu trúc Thư mục

```
custom_lora_node/
├── CMakeLists.txt        # Tệp cấu hình kịch bản biên dịch dự án
├── prj.conf              # Tệp cấu hình phân hệ phần mềm (Kconfig)
├── app.overlay           # Tệp ghi đè cấu trúc phần cứng (Tùy chọn)
└── src/
    └── main.c            # Tệp mã nguồn logic lõi
```

### 3.2. Cấu trúc Tệp Kịch bản Biên dịch (`CMakeLists.txt`)

Tệp này đảm nhận nhiệm vụ định vị hệ thống biên dịch của Zephyr và khai báo các mã nguồn phụ thuộc.

```
# Yêu cầu phiên bản CMake tối thiểu
cmake_minimum_required(VERSION 3.20.0)

# Khởi tạo quy trình tìm kiếm và nhúng hệ thống Zephyr (sử dụng biến môi trường)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})

# Định danh không gian dự án
project(custom_lora_node)

# Tích hợp mã nguồn vào mục tiêu biên dịch (target)
target_sources(app PRIVATE src/main.c)
```

### 3.3. Định nghĩa Cấu hình Phân hệ (`prj.conf`)

Khai báo tĩnh các trình điều khiển và thư viện bên thứ ba cần thiết cho vòng đời của ứng dụng.

```
# Kích hoạt Phân hệ Ghi nhật ký (Logging Subsystem) qua giao thức nối tiếp
CONFIG_LOG=y
CONFIG_LOG_MODE_IMMEDIATE=y

# Kích hoạt Trình điều khiển Ngoại vi Cơ sở
CONFIG_GPIO=y
CONFIG_SPI=y

# Tích hợp Khối trừu tượng hóa Phần cứng của Semtech (Modem HAL)
CONFIG_SMTC_MODEM_HAL=y
```

### 3.4. Hiện thực hóa Logic Ứng dụng (`src/main.c`)

Mã nguồn khởi tạo vi nhân Zephyr và vòng lặp sự kiện chính của ứng dụng.

```
#include <zephyr/kernel.h>
#include <zephyr/logging/log.h>

/* Đăng ký không gian tên (namespace) cho phân hệ ghi nhật ký */
LOG_MODULE_REGISTER(custom_node, LOG_LEVEL_INF);

int main(void) {
    LOG_INF("Khởi tạo Hệ thống Custom LoRa Node trên nền tảng Zephyr RTOS");

    while (1) {
        LOG_INF("Heartbeat luồng (Thread) đang hoạt động...");
        
        /* * Lệnh chuyển giao ngữ cảnh (Context Switch).
         * Đình chỉ luồng hiện tại trong 1000ms, cho phép Bộ lập lịch (Scheduler)
         * kích hoạt các luồng có độ ưu tiên thấp hơn (như luồng xử lý Modem LBM)
         * hoặc đưa MCU vào trạng thái Idle/Sleep để tối ưu hóa năng lượng.
         */
        k_msleep(1000); 
    }

    return 0;
}
```

### 3.5. Quy trình Trình dịch và Triển khai (Build & Deploy)

Tại giao diện dòng lệnh (Terminal) đặt tại thư mục gốc của dự án `custom_lora_node`:

```
# Bắt buộc: Nạp các biến môi trường cấu hình của Python Virtual Environment
source ../lora_usp_workspace/.venv/bin/activate

# Khởi chạy quá trình phân tích ngữ nghĩa (Parse) và biên dịch mã nguồn (Compile)
west build -b xiao_nrf54l15/nrf54l15/cpuapp -- -DSHIELD=semtech_wio_lr2021

# Xóa bộ nhớ và ghi mã thực thi nhị phân xuống vùng Flash của thiết bị
west flash
```