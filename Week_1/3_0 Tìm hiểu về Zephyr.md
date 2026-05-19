# Bức tranh lớn: Embedded đã tiến hóa như thế nào?

## 1. Thời kỳ đầu: Bare-metal firmware

Ngày xưa embedded chủ yếu kiểu này:

```c
while(1) {
    read_sensor();
    process_data();
    send_uart();
}
```

MCU chạy một vòng lặp vô tận.  
Không OS. Không scheduler. Không abstraction.

### Ưu điểm

- Rất nhẹ
    
- Tốc độ cao
    
- Dễ tối ưu
    

### Nhưng vấn đề bắt đầu xuất hiện 😵

Khi hệ thống lớn hơn:

- nhiều sensor
    
- BLE/WiFi
    
- OTA update
    
- logging
    
- multitasking
    
- power management
    

thì code bắt đầu thành:

```c
if(timer1_flag) ...
if(uart_flag) ...
if(wifi_flag) ...
if(retry_flag) ...
```

=> firmware trở thành một “state machine khổng lồ”.

---

# 2. RTOS xuất hiện để giải quyết multitasking

Lúc này các RTOS như:

- FreeRTOS
    
- VxWorks
    
- ThreadX
    

xuất hiện.

---

## RTOS giải quyết cái gì?

Thay vì:

```txt
1 vòng loop làm mọi thứ
```

thì chuyển thành:

```txt
Task sensor
Task communication
Task logging
Task control
```

Kernel sẽ:

- chia CPU time
    
- ưu tiên task
    
- xử lý interrupt
    
- đồng bộ task
    

---

## Đây là bước tiến cực lớn

Firmware bắt đầu giống “mini operating system”.

---

# Nhưng RTOS truyền thống vẫn còn vấn đề

Đây là chỗ Zephyr xuất hiện.

## FreeRTOS cực mạnh nhưng...

FreeRTOS thực ra chủ yếu là:

```txt
Kernel + scheduler
```

Nó KHÔNG phải full ecosystem.

Nghĩa là:

|Thành phần|FreeRTOS|
|---|---|
|Scheduler|✅|
|Thread|✅|
|Mutex/Semaphore|✅|
|Networking|❌ tự ghép|
|Driver framework|❌|
|Device tree|❌|
|OTA|❌|
|Unified hardware abstraction|❌|

---

# Hậu quả là gì?

Mỗi công ty sẽ làm kiểu:

```txt
Team A tự viết BLE stack
Team B tự viết driver layer
Team C tự viết OTA
```

=> ecosystem bị phân mảnh.

---

# Đây chính là “vấn đề thời đại” mà Zephyr sinh ra để giải quyết

Zephyr hỏi:

> “Tại sao embedded lại phải tự ráp mọi thứ như LEGO mỗi lần làm sản phẩm?”

---

# Triết lý của Zephyr

Zephyr không muốn chỉ là:

```txt
RTOS kernel
```

Nó muốn là:

```txt
FULL EMBEDDED PLATFORM
```

Tức là:

```txt
Kernel
+ Driver framework
+ Networking
+ BLE
+ Filesystem
+ Device model
+ Security
+ OTA
+ Build system
+ Hardware abstraction
```

---

# Vậy cơ chế của Zephyr khác ở đâu?

Đây là phần QUAN TRỌNG NHẤT.

---

# Zephyr tách hệ thống thành các lớp

```txt
Application
↓
Zephyr APIs
↓
Kernel + Subsystems
↓
Driver Model
↓
Hardware
```

---

# Tại sao chuyện này quan trọng?

Vì ứng dụng KHÔNG còn phụ thuộc trực tiếp vào board nữa.

Ví dụ:

Thay vì:

```c
GPIOA->ODR |= ...
```

(application nói chuyện trực tiếp với register)

thì Zephyr làm:

```c
gpio_pin_set(dev, pin, value);
```

---

# Lợi ích khổng lồ 💥

## Đổi board ít phải sửa code

Ví dụ:

- hôm nay STM32
    
- mai nRF52
    
- mốt ESP32
    

Application gần như giữ nguyên.

Chỉ đổi:

- device tree
    
- driver binding
    
- config
    

---

# Device Tree là một “vũ khí” rất lớn của Zephyr

Đây là thứ học embedded truyền thống ít gặp.

---

# Trước đây

Code thường hard-code:

```c
UART1
GPIOA pin 5
SPI2
```

=> đổi board là chết 😭

---

# Zephyr làm khác

Nó mô tả phần cứng bằng data:

```dts
&uart1 {
    status = "okay";
    current-speed = <115200>;
};
```

Application KHÔNG quan tâm:

- UART nào
    
- chân nào
    
- chip nào
    

Nó chỉ lấy device theo abstract API.

---

# Đây là bước chuyển tư duy rất lớn

Từ:

```txt
Code bám phần cứng
```

sang:

```txt
Code bám abstraction
```

---

# Đây chính là cách Zephyr giải quyết fragmentation

## Không còn:

- mỗi hãng một driver
    
- mỗi board một API
    
- mỗi project một architecture
    

---

# Zephyr giống Linux ở tư tưởng

Và đây là điểm cực thú vị.

Zephyr được Linux Foundation hỗ trợ vì nó mang tư duy của Linux xuống embedded nhỏ.

---

# Linux đã thành công nhờ:

- chuẩn hóa driver
    
- abstraction layer
    
- portable architecture
    
- ecosystem mở
    

Zephyr cố làm điều tương tự cho MCU.

---

# Vì sao IoT đặc biệt cần Zephyr?

IoT không còn là:

```txt
1 MCU + 1 sensor
```

mà là:

```txt
cloud
BLE
WiFi
security
OTA
mesh
logging
power management
```

---

# Nếu dùng bare-metal 😵

Ông sẽ phải:

- tự xử lý reconnect WiFi
    
- tự viết retry
    
- tự viết OTA
    
- tự viết scheduler
    
- tự quản memory
    
- tự ghép TLS
    
- tự build architecture
    

=> sản phẩm scale cực khó.

---

# Zephyr biến embedded thành “system engineering”

Đây là ý quan trọng nhất.

---

## Bare-metal mindset

```txt
Tôi đang code cho con chip này
```

---

## Zephyr mindset

```txt
Tôi đang xây một platform software cho thiết bị
```

---

# Zephyr giải quyết bài toán công nghiệp nhiều hơn bài toán học thuật

Đó là lý do:

- Nordic thích
    
- NXP thích
    
- Intel thích
    
- nhiều hãng IoT thích
    

vì nó giúp:

- maintain lâu dài
    
- đổi hardware dễ hơn
    
- team lớn làm việc dễ hơn
    
- reusable architecture
    

---

# Nhưng cái giá phải trả là gì? 😅

Zephyr phức tạp hơn rất nhiều.

---

# Vì sao người mới thấy Zephyr “khó”?

Vì ông phải học:

- RTOS
    
- scheduler
    
- Kconfig
    
- device tree
    
- west build
    
- subsystem
    
- driver model
    
- linker/memory
    
- concurrency
    

---

# Nhưng đổi lại...

Ông đang học:

```txt
Cách embedded industry thật sự build product
```

chứ không chỉ “blink LED”.

---

# Một câu mô tả rất đúng về Zephyr

> FreeRTOS giúp MCU chạy đa nhiệm.  
> Zephyr giúp xây cả một embedded platform.

---

# Nếu nhìn dưới góc độ kiến trúc

Thì evolution sẽ kiểu này:

```txt
Bare-metal
    ↓
RTOS kernel
    ↓
RTOS ecosystem
    ↓
Embedded software platform
```

Và Zephyr nằm ở tầng cuối. 🚀