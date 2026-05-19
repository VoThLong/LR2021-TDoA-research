
> Link repo USP_Zephyr: 
> https://github.com/Lora-net/usp_zephyr.git

**Dự án:** Nghiên cứu mã nguồn mở Semtech USP trên Zephyr RTOS tích hợp LoRa/LoRaWAN **Thiết bị mục tiêu (Target):** Seeed Studio XIAO nRF54L15 (MCU) + Semtech LR2021 (LoRa Transceiver) **Hệ điều hành Host:** Fedora 44 (x86_64) **Giai đoạn R&D:** Giai đoạn 3 - Khởi tạo Workspace Đa nguồn & Cấu hình Toolchain Biên dịch chéo **Kỹ sư thực hiện:** Junior Embedded Systems R&D Engineer **Mentor hướng dẫn:** Senior Embedded Systems + LoRa/LoRaWAN R&D Mentor

## 1. Phân tích So sánh các Mô hình Kiến trúc Phần mềm Nhúng

Trong quá trình chuyển dịch từ việc kiểm thử hộp đen (Blackbox Testing) bằng các công cụ đóng gói sẵn sang việc phát triển firmware hệ thống độc lập tại thiết bị biên, việc lựa chọn mô hình kiến trúc phần mềm quyết định khả năng mở rộng và hiệu năng của hệ thống.

### 1.1. Mô hình Bare-metal (Vòng lặp vô tận)

- **Cấu trúc:** Vận hành dựa trên vòng lặp tuần tự `while(1)` kết hợp với các trình phục vụ ngắt (ISR).
    
- **Đặc tính kỹ thuật:** Tiêu hao tài nguyên bộ nhớ tối thiểu, tốc độ thực thi trực tiếp trên phần cứng cao.
    
- **Hạn chế:** Khi tích hợp nhiều tác vụ đồng thời (như đọc cảm biến, quản lý tiết kiệm năng lượng, duy trì ngăn xếp giao thức vô tuyến), hệ thống dễ rơi vào trạng thái bị nghẽn (blocking) do thiếu cơ chế lập lịch phi tuần tự. Việc quản lý thời gian (timing) của các tác vụ vô tuyến đòi hỏi độ chính xác cao, điều khó thực hiện ổn định trên mô hình bare-metal khi quy mô mã nguồn tăng lên.
    

### 1.2. Mô hình RTOS truyền thống (như FreeRTOS, ThreadX)

- **Cấu trúc:** Phân chia ứng dụng thành các luồng xử lý độc lập (Tasks) được điều phối bởi một bộ lập lịch phân chia thời gian (Scheduler) dựa trên độ ưu tiên.
    
- **Đặc tính kỹ thuật:** Giải quyết được bài toán đa nhiệm đồng thời, cung cấp các cơ chế đồng bộ hóa luồng (semaphores, mutexes, message queues).
    
- **Hạn chế:** Chỉ cung cấp lõi hệ điều hành (kernel). Nhà phát triển vẫn phải tự tích hợp và duy trì các lớp trừu tượng hóa phần cứng (HAL) của nhà sản xuất vi điều khiển, dẫn đến mã nguồn phụ thuộc chặt chẽ vào một dòng chip cụ thể, gây khó khăn khi chuyển đổi phần cứng (hardware porting).
    

### 1.3. Mô hình Hệ điều hành Toàn diện (Zephyr RTOS)

- **Cấu trúc:** Hệ điều hành tích hợp đầy đủ bao gồm lõi RTOS, mô hình Driver đồng nhất (Driver Model), quản lý nguồn (Power Management Subsystem) và các ngăn xếp giao thức (Network Stack).
    
- **Đặc tính kỹ thuật:** Tách biệt hoàn toàn mã nguồn ứng dụng C khỏi mô tả cấu hình phần cứng thông qua hệ thống Device Tree.
    
- **Mục đích áp dụng:** Đối với dự án tích hợp Semtech USP và nRF54L15, Zephyr RTOS cung cấp khả năng điều phối đa luồng tối ưu giữa tác vụ quản lý Modem Stack (LoRa Basics Modem) chạy ngầm và tác vụ xử lý logic ứng dụng, đồng thời tự động hóa việc quản lý trạng thái năng lượng thấp (Low Power States) của vi điều khiển mà không cần can thiệp thủ công vào thanh ghi cấu hình.
    

## 2. Nhật ký Cài đặt & Phân tích Cơ sở Kỹ thuật của các Gói phụ thuộc trên Host

Quá trình thiết lập môi trường biên dịch chéo trên hệ điều hành Host Fedora 44 (x86_64) yêu cầu cài đặt các công cụ hệ thống dưới đây.

### 2.1. Tiến trình thực thi cài đặt

Hệ thống Fedora 44 đã ghi nhận và hoàn thành cài đặt các gói tin cơ sở:

```
# Cập nhật cache của trình quản lý gói dnf
sudo dnf makecache

# Cài đặt nhóm công cụ phát triển cơ bản
sudo dnf install -y @development-tools

# Cài đặt các thư viện liên kết và trình biên dịch bổ trợ cho Zephyr RTOS
sudo dnf install -y gcc gcc-c++ make cmake ninja-build gperf dfu-util dtc wget which \
                    python3-pip python3-tkinter xz file python3-devel SDL2-devel

# Khởi tạo và kích hoạt môi trường ảo Python (Virtual Environment)
python3 -m venv ~/lora_usp_workspace/.venv
source ~/lora_usp_workspace/.venv/bin/activate

# Nâng cấp pip và cài đặt trình quản lý west vào môi trường ảo
pip install --upgrade pip
pip install west
```

### 2.2. Phân tích chức năng kỹ thuật của các gói thành phần

| Tên Package cài đặt                 | Vai trò kỹ thuật trong Toolchain     | Lý do bắt buộc cài đặt                                                                                                                                                                                                                           |
| ----------------------------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`@development-tools`**            | Nhóm công cụ phát triển phần mềm     | Cung cấp các công cụ biên dịch cơ sở của hệ điều hành như `make`, `autoconf`, và các thư viện liên kết động phục vụ việc biên dịch các script sinh mã chạy trên Host.                                                                            |
| **`cmake`**                         | Hệ thống Meta-build                  | Đóng vai trò cấu hình quy trình biên dịch. `cmake` phân tích tệp `CMakeLists.txt` trong dự án để tự động tạo ra các kịch bản xây dựng hệ thống tương thích với cấu trúc của Zephyr và phần cứng mục tiêu.                                        |
| **`ninja-build`**                   | Trình xây dựng hệ thống (Build tool) | Thay thế cho trình `make` truyền thống. `ninja` được thiết kế để tối ưu hóa tốc độ biên dịch thông qua việc phân tích cây phụ thuộc mã nguồn và phân bổ tác vụ biên dịch song song trên đa nhân CPU của máy Host.                                |
| **`dtc`**                           | Device Tree Compiler                 | Trình dịch cây thiết bị. Có nhiệm vụ biên dịch các file mô tả phần cứng tĩnh (`.dtsi`, `.dts`) và các file ghi đè cấu hình chân (`.overlay`) từ định dạng văn bản sang định dạng nhị phân cây thiết bị (`.dtb`) để nạp vào bộ nhớ vi điều khiển. |
| **`gperf`**                         | Perfect Hash Generator               | Bộ sinh hàm băm tối ưu. Được sử dụng để tối ưu hóa việc quản lý các lời gọi hệ thống (system calls) và các phân vùng bộ nhớ trong nhân Zephyr RTOS, làm giảm kích thước mã nguồn thực thi cuối cùng.                                             |
| **`python3-devel` / `python3-pip`** | Môi trường lập trình Python          | Cung cấp thư viện tiêu đề (headers) và trình quản lý gói của Python. Cần thiết để xây dựng môi trường cho công cụ `west`.                                                                                                                        |
| **`sdl2-compat-devel`**             | Thư viện đồ họa và mô phỏng          | Cho phép biên dịch và chạy thử nghiệm (simulation) ứng dụng Zephyr trực tiếp trên máy tính Host Linux (native target) để gỡ lỗi thuật toán trước khi nạp xuống vi điều khiển thực tế.                                                            |
| **`python3 -m venv`**               | Khởi tạo môi trường ảo               | Tạo không gian tách biệt cho Python để cài đặt các gói mở rộng (như `west`), giúp tuân thủ chính sách bảo mật (PEP 668) của Fedora và không gây xung đột hệ điều hành.                                                                           |
| **`west`**                          | Trình quản lý Meta-tool              | Công cụ điều phối cốt lõi của dự án Zephyr. Dùng để quản lý đa kho mã nguồn, phân tích tệp `west.yml` và kích hoạt luồng biên dịch/nạp phần cứng.                                                                                                |
## 3. Kiến trúc Quản lý Workspace bằng công cụ `west`

Kiến trúc đa kho lưu trữ của Zephyr đòi hỏi một công cụ điều phối trung tâm để đồng bộ hóa các thành phần mã nguồn từ nhiều nguồn độc lập.

```
                           +------------------------+
                           |  My West Workspace     |
                           |  (lora_usp_workspace)  |
                           +------------------------+
                                       |
                  +--------------------+--------------------+
                  |                    |                    |
                  v                    v                    v
       +--------------------+ +--------------------+ +--------------------+
       |  Application Repo  | |  Zephyr RTOS Core  | | Vendor HALs (Nordic|
       |    (usp_zephyr)    | |    (zephyr-core)   | |  NXP, STM32, etc.) |
       | - main.c           | | - RTOS Kernel      | | - Low-level drivers|
       | - .overlay         | | - OS Subsystems    | | - Chip-specific    |
       | - west.yml         | | - Base Driver model| |   registers        |
       +--------------------+ +--------------------+ +--------------------+
```

### 3.1. Phân tích cấu trúc tệp Manifest (`west.yml`)

Mã nguồn dự án `usp_zephyr` của Semtech đóng vai trò là một **Manifest Repository**. Bên trong thư mục này chứa tệp `west.yml` quy định cấu trúc và liên kết phiên bản của toàn bộ hệ thống:

- Đường dẫn Git và phiên bản cụ thể của lõi hệ điều hành Zephyr.
    
- Các module driver phần cứng ngoại vi của hãng thứ ba (Nordic HAL cho SoC nRF54L15).
    
- Driver của dòng chip thu phát vô tuyến Semtech LR2021.
    

### 3.2. Cơ chế thực thi của các lệnh điều phối

- **`west init`:** Khởi tạo thư mục quản lý ẩn `.west/` và tải về Manifest Repository làm gốc tham chiếu.
    
- **`west update`:** Phân tích nội dung tệp `west.yml`, tự động thực hiện các tiến trình clone mã nguồn từ các máy chủ Git tương ứng để xây dựng cây thư mục hệ thống đồng nhất.
    
- **`west zephyr-export`:** Đăng ký vị trí cài đặt của nhân Zephyr hiện tại vào CMake Package Registry của người dùng trên hệ thống Linux Host. Điều này giúp trình biên dịch `CMake` tự động định tuyến đường dẫn include khi xây dựng ứng dụng.
    
- **`west packages pip --install`:** Cài đặt các gói Python chuyên biệt phục vụ quá trình phân tích cú pháp tệp cấu hình Kconfig và định nghĩa Device Tree.
    

## 4. Thiết lập Zephyr SDK và Toolchain Biên dịch chéo

Do kiến trúc tập lệnh của CPU máy Host ($x86\_64$) khác biệt hoàn toàn với kiến trúc tập lệnh của vi điều khiển đích nRF54L15 ($ARMv8-M$ - lõi ARM Cortex-M33), việc thiết lập bộ dịch chéo (Cross-Compiler) là bắt buộc để chuyển đổi mã nguồn C thành mã máy nhị phân chạy được trên thiết bị nhắm chọn.

```
[ Máy tính Host: Fedora x86_64 ]
               |
               | (Sử dụng Toolchain: arm-zephyr-eabi-gcc)
               v
[ Biên dịch ra mã nhị phân ARM Cortex-M33 (hex/bin) ]
               |
               | (Nạp qua USB / Debugger)
               v
[ Thiết bị Target: XIAO nRF54L15 ]
```

### 4.1. Bản chất kỹ thuật của Toolchain `arm-zephyr-eabi`

- **`arm`:** Xác định tập lệnh đích của kiến trúc vi xử lý ARM.
    
- **`zephyr`:** Thể hiện trình biên dịch đã được tùy biến và tối ưu hóa theo các tiêu chuẩn liên kết vùng nhớ của hệ điều hành Zephyr.
    
- **`eabi` (Embedded Application Binary Interface):** Quy chuẩn định dạng file nhị phân cho hệ thống nhúng, quy định cách thức truyền tham số qua các thanh ghi vật lý và tối ưu toán xử lý số học dấu phẩy động (FPU) trên lõi Cortex-M33.
    

### 4.2. Tiến trình nạp Toolchain

Quá trình tải, giải nén và đăng ký bộ biên dịch chéo được thực hiện qua các câu lệnh:

```
cd ~/.local
wget [https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz](https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.8/zephyr-sdk-0.16.8_linux-x86_64.tar.xz)
tar xvf zephyr-sdk-0.16.8_linux-x86_64.tar.xz
cd zephyr-sdk-0.16.8
./setup.sh -t arm-zephyr-eabi
```

Lệnh `./setup.sh -t arm-zephyr-eabi` thực hiện đăng ký đường dẫn tuyệt đối của trình biên dịch `arm-zephyr-eabi-gcc` vào hệ thống CMake Registry cục bộ của tài khoản người dùng, loại bỏ yêu cầu phải thiết lập thủ công các biến môi trường hệ thống trong tệp cấu hình shell (`.bashrc` hoặc `.zshrc`).

## 5. Quy trình Kiểm thử Biên dịch (Smoke Test)

Để xác thực tính liên kết hệ thống giữa Host Dependencies, cấu trúc thư mục quản lý bởi West và bộ biên dịch chéo Zephyr SDK, quy trình biên dịch thử nghiệm dự án mẫu `ping_pong` được thực thi:

```
cd ~/lora_usp_workspace
west build -b xiao_nrf54l15/nrf54l15/cpuapp samples/usp/sdk/ping_pong
```

### Phân tích ý nghĩa các tham số biên dịch

- **`-b xiao_nrf54l15/nrf54l15/cpuapp`:** Định nghĩa board mạch và lõi vi xử lý đích. Đối với dòng chip SoC đa lõi nRF54L15, phân vùng lõi `cpuapp` (Cortex-M33 Application Core) chịu trách nhiệm chính trong việc thực thi logic ứng dụng, quản lý tài nguyên RTOS và kiểm soát các giao tiếp ngoại vi SPI/GPIO kết nối với transceiver Semtech LR2021.
    
- **`samples/usp/sdk/ping_pong`:** Đường dẫn thư mục chứa cấu trúc mã nguồn logic của ứng dụng kiểm thử.
    

Tiến trình biên dịch được ghi nhận là hoàn tất khi hệ thống liên kết thành công mã nguồn, hiển thị bảng thông số phân bổ dung lượng bộ nhớ tĩnh (ROM/RAM) và tạo ra tệp tin thực thi nhị phân `zephyr.bin` hoặc `zephyr.hex` tại đường dẫn `build/zephyr/`.