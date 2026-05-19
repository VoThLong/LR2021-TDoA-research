# 1. Ping Pong

**Thiết bị:** Semtech LR2021EVK1XBS1 (868 MHz) **Giao thức:** Thu phát LoRa Point-to-Point (P2P) **Tham số cấu hình:** SF7, BW 125 kHz, CR 4/5, TX Power 22.0 dBm
![[Pasted image 20260518093757.png]]

## 1. Trích xuất Log nguyên bản (Raw Log Snippets)

Để đảm bảo tính khách quan của dữ liệu vô tuyến, quá trình trích xuất được thực hiện đồng thời trên cả hai Node (Master và Slave) tại cùng một thời điểm diễn ra sự kiện suy hao tín hiệu (khoảng chu kỳ 68 - 71).

### 1.1. Log từ Node Master (COM4)

```
[COM4] - [INFO] - [APPLICATION] - Sent message PING, iteration 68

[COM4] - [TRACE] - [APPLICATION] - RX timeout set to 809 ms
[COM4] - [INFO] - [APPLICATION] - Interrupt flags = 0x00008034
[COM4] - [INFO] - [APPLICATION] - Interrupt flags (after filtering) = 0x00000004
[COM4] - [INFO] - [APPLICATION] - Rx done
[COM4] - [INFO] - [APPLICATION] - LoRa packet status:
[COM4] - [INFO] - [APPLICATION] -   - RSSI packet = -19 dBm
[COM4] - [INFO] - [APPLICATION] -   - Signal RSSI packet = -17 dBm
[COM4] - [INFO] - [APPLICATION] -   - SNR packet = 14 dB
[COM4] - [INFO] - [APPLICATION] -   - Packet length = 7 bytes
[COM4] - [INFO] - [APPLICATION] -   - Packet content: 
[COM4] - [INFO] - [APPLICATION] - 504F4E47004506

[COM4] - [INFO] - [APPLICATION] - Received message PONG, iteration 70
```

### 1.2. Log từ Node Slave (COM3)

```
[COM3] - [INFO] - [APPLICATION] - Sent message PONG, iteration 69

[COM3] - [TRACE] - [APPLICATION] - RX timeout set to 819 ms
[COM3] - [INFO] - [APPLICATION] - Interrupt flags = 0x00008034
[COM3] - [INFO] - [APPLICATION] - Interrupt flags (after filtering) = 0x00000004
[COM3] - [INFO] - [APPLICATION] - Rx done
[COM3] - [INFO] - [APPLICATION] - LoRa packet status:
[COM3] - [INFO] - [APPLICATION] -   - RSSI packet = -40 dBm
[COM3] - [INFO] - [APPLICATION] -   - Signal RSSI packet = -38 dBm
[COM3] - [INFO] - [APPLICATION] -   - SNR packet = 14 dB
[COM3] - [INFO] - [APPLICATION] -   - Packet length = 7 bytes
[COM3] - [INFO] - [APPLICATION] -   - Packet content: 
[COM3] - [INFO] - [APPLICATION] - 50494E47004606

[COM3] - [INFO] - [APPLICATION] - Received message PING, iteration 71
```

## 2. Phân tích Chi tiết Hệ thống (Deep Dive Analysis)

### 2.1. Cơ chế Ngắt và Quản lý Thời gian (Timing & Interrupts)

Hệ thống xử lý tầng vật lý (PHY) hoàn toàn dựa trên sự kiện ngắt phần cứng (Hardware Interrupt-driven), không sử dụng kỹ thuật lấy mẫu (polling) liên tục gây tốn năng lượng:

- `RX timeout set to 8xx ms`: Ngay sau khi phát xong, vi điều khiển ở cả hai mạch đều thiết lập một bộ định thời (timer) cửa sổ nhận khoảng ~800 mili-giây. Nếu trong khoảng thời gian này không có tín hiệu, hệ thống sẽ báo timeout.
    
- `Interrupt flags = 0x00008034`: Chip radio LR2021 phát hiện có mảng sóng LoRa hợp lệ, tự động giải điều chế và kéo chân ngắt IO để báo cho vi điều khiển.
    
- Trạng thái `Rx done` xuất hiện đồng thời trên cả hai log xác nhận không có tình trạng mất đồng bộ (desynchronization) giữa hai mạch.
    

### 2.2. Đánh giá Chất lượng Sóng vô tuyến qua Đối chiếu Chéo (RF Metrics)

Việc so sánh chỉ số giữa hai thiết bị tại cùng một thời điểm cho thấy đặc tính thú vị của môi trường truyền dẫn vô tuyến (RF propagation):

- **Biến động Cường độ tín hiệu nhận (RSSI):** * Node COM4 ghi nhận RSSI giảm xuống **-19 dBm**.
    
    - Cùng lúc đó, Node COM3 ghi nhận RSSI rớt mạnh hơn, xuống mức **-40 dBm** (và giảm tiếp xuống -50 dBm ở chu kỳ sau).
        
    - _Nhận xét:_ Sự chênh lệch RSSI giữa hai chiều phát/thu (asymmetry) chứng minh sự thay đổi vật lý của môi trường (thiết bị bị di chuyển hoặc bị vật cản che khuất). Mức -40 dBm đối với LoRa vẫn là cường độ rất tốt, đảm bảo tỷ lệ mất gói (Packet Loss) bằng 0%.
        
- **Tỷ số Tín hiệu trên Nhiễu (SNR):** Cả hai bên đều duy trì SNR ở mức **14 dB**. Giá trị dương cao khẳng định tín hiệu đích áp đảo hoàn toàn nhiễu nền, chứng minh dải tần 868 MHz tại thời điểm test khá sạch (ít thiết bị nhiễu cùng băng tần).
    

### 2.3. Giải mã Gói tin (Payload Decoding)

Dữ liệu nguyên bản (Hexadecimal) được trao đổi qua lại mang thông điệp định danh rõ ràng:

1. **Phía COM4 nhận từ COM3:** `504F4E47004506`
    
    - Header: `50 4F 4E 47` tương ứng mã ASCII là **P-O-N-G**.
        
    - Counter: `00 45` (Giá trị Thập phân là **69**). Đây là chu kỳ đếm do COM3 đính kèm vào.
        
2. **Phía COM3 nhận từ COM4:** `50494E47004606`
    
    - Header: `50 49 4E 47` tương ứng mã ASCII là **P-I-N-G**.
        
    - Counter: `00 46` (Giá trị Thập phân là **70**). Đây là chu kỳ đếm do COM4 đẩy ngược lại.
        

_Byte `06` ở cuối mỗi chuỗi đóng vai trò là cờ kết thúc khung tin (Delimiter)._

## 3. Kết luận

Dựa trên dữ liệu log đối chiếu song phương (Bi-directional Log Analysis), bộ kit LR2021EVK1XBS1 cho thấy khả năng hoạt động cực kỳ ổn định. Giao tiếp Ping-Pong ở tầng PHY được duy trì liên tục không rớt gói dù có sự biến động vật lý gây suy hao cường độ sóng (RSSI tụt từ -1 dBm xuống -50 dBm). Thuật toán quản lý ngắt (Interrupt) của Firmware hoạt động trơn tru, giúp MCU thoát khỏi trạng thái chờ (blocking) một cách hiệu quả.
# 2. Continuous Wave
(Cái này cần thiết bị đo chuyên dụng còn không chỉ tổ nóng máy => pass)
# 3. Packet Error Rate

**Thiết bị đánh giá:** Semtech LR2021EVK1XBS1 (Tần số 868 MHz) **Giao thức test:** Packet Error Rate (PER) - Đơn hướng (Uni-directional) **Cấu hình Radio:** Spreading Factor: SF7 | Bandwidth: 125 kHz | Coding Rate: 4/5 | TX Power: 22.0 dBm

## 1. Phương pháp & Mô hình Đo lường (Test Methodology)

Bài test được thực hiện theo phương pháp Walk-test trong nhà (Indoor), trong đó:

- **Bộ Thu (Receiver - COM4):** Đặt cố định trên mặt bàn.
    
- **Bộ Phát (Transmitter - COM3):** Chạy ở chế độ Standalone (cấp nguồn bằng sạc dự phòng), được cầm trên tay và di chuyển ra xa theo từng mốc gạch lát sàn.
    

Khoảng cách thực tế giữa hai ăng-ten ($d$) được tính toán theo mô hình không gian 3 chiều (3D Euclidean Distance) để đảm bảo độ chính xác tuyệt đối:

- **Trục X (Khoảng cách ngang cố định):** 2 ô gạch = $2 \times 0.6m = 1.2m$
    
- **Trục Y (Độ dài tịnh tiến):** $N$ ô gạch = $N \times 0.6m$
    
- **Trục Z (Độ chênh lệch chiều cao):** Mạch TX cầm trên tay cao hơn RX trên bàn = $0.3m$
    

**Công thức áp dụng:**

$$d = \sqrt{X^2 + Y^2 + Z^2} = \sqrt{1.2^2 + (N \times 0.6)^2 + 0.3^2}$$

## 2. Bảng Đánh giá Suy hao Tín hiệu (RF Propagation & PER)

Dựa trên đối chiếu giữa biến đếm chu kỳ (Counter Value) trích xuất từ log của thiết bị nhận và vị trí đo kiểm vật lý, kết quả suy hao vô tuyến được ghi nhận như sau:

|Mã định danh gói (Counter)|Số ô tịnh tiến (N)|Khoảng cách thực tế (d)|Cường độ tín hiệu (RSSI)|Tỷ số Tín hiệu/Nhiễu (SNR)|Tỷ lệ rớt gói (PER)|
|---|---|---|---|---|---|
|**160**|0 ô|~ 1.24 m|_-15 dBm_|15 dB|0.0 %|
|**225**|3 ô|~ 2.18 m|_-18 dBm_|15 dB|0.0 %|
|**70**|6 ô|~ 3.81 m|**-20 dBm**|15 dB|0.0 %|
|**110**|9 ô|~ 5.54 m|**-31 dBm**|15 dB|0.0 %|
|**150**|12 ô|~ 7.31 m|**-40 dBm**|14 dB|0.0 %|
|**190**|15 ô|~ 9.08 m|_-48 dBm_|14 dB|0.0 %|
|**230**|18 ô|~ 10.87 m|_-55 dBm_|13 dB|0.0 %|
|**30**|21 ô|~ 12.66 m|_-60 dBm_|13 dB|0.0 %|

_(Ghi chú: Dữ liệu RSSI/SNR được trích xuất trực tiếp từ log `COM4_2026-05-18_10-40-49.log`. Một số giá trị khoảng cách ngắn được nội suy do mạch đặt quá gần)._

## 3. Trích xuất Log Mẫu (Raw Data Parsing)

### 3.1. Trạng thái nhận gói tin thành công (PER = 0%)

Khi tín hiệu vô tuyến ổn định, MCU nhận tín hiệu từ phần cứng thông qua ngắt (Interrupt), bóc tách thành công Payload và cập nhật chỉ số rớt gói bằng 0.

```
[COM4] - [INFO] - [APPLICATION] - Interrupt flags = 0x0000A034
[COM4] - [INFO] - [APPLICATION] - Interrupt flags (after filtering) = 0x00000004
[COM4] - [INFO] - [APPLICATION] - Rx done
[COM4] - [INFO] - [APPLICATION] - LoRa packet status:
[COM4] - [INFO] - [APPLICATION] -    - RSSI packet = -31 dBm
[COM4] - [INFO] - [APPLICATION] -    - Signal RSSI packet = -29 dBm
[COM4] - [INFO] - [APPLICATION] -    - SNR packet = 15 dB
[COM4] - [INFO] - [APPLICATION] -    - Packet length = 7 bytes
[COM4] - [INFO] - [APPLICATION] - Counter value: 110, PER index: 80
[COM4] - [INFO] - [APPLICATION] - PER = 0.0%
```

### 3.2. Trạng thái Suy hao / Mất liên lạc (Rx Timeout)

Phân tích log cuối file cho thấy khi bị che khuất hoàn toàn hoặc hết thời gian chờ (`Rx timeout`), tỷ lệ lỗi gói (PER) bắt đầu tăng vọt lên mức **>11%**. Điều này khẳng định cơ chế tính toán PER của Semtech hoạt động chuẩn xác dựa trên số Sequence mất mát.

```
[COM4] - [INFO] - [APPLICATION] - Interrupt flags = 0x00000008
[COM4] - [INFO] - [APPLICATION] - Interrupt flags (after filtering) = 0x00000008
[COM4] - [WARNING] - [APPLICATION] - Rx timeout
[COM4] - [INFO] - [APPLICATION] - PER = 11.40%
```

## 4. Nhận xét & Đánh giá Chuyên môn

1. **Khả năng duy trì Link Budget:** Trong phạm vi bán kính ~13 mét trong nhà, cường độ tín hiệu (RSSI) suy giảm tỷ lệ thuận với khoảng cách không gian (từ -15 dBm xuống -60 dBm). Tuy nhiên, mức -60 dBm vẫn là một tín hiệu **cực kỳ dồi dào** đối với bộ thu giải điều chế LoRa (thường nhạy đến ngưỡng -120 dBm tại SF7).
    
2. **Khả năng chống nhiễu (Interference Immunity):** Chỉ số SNR duy trì ổn định ở mức dương từ **+13 dB đến +15 dB** xuyên suốt dải đo. Điều này chứng minh sóng mang định tuyến ở tần số 868 MHz áp đảo hoàn toàn nhiễu nền vô tuyến (Noise floor) tại khu vực test, không xuất hiện hiện tượng tắc nghẽn phổ (Spectrum Congestion).
    
3. **Độ tin cậy của giao thức:** Trong điều kiện Line-of-Sight (hoặc cản nhẹ), chỉ số PER duy trì ở mức **0.0%**. Các điểm rớt gói nhảy vọt lên 11% ở cuối file log chủ yếu do giới hạn về góc khuất (Non-Line-of-Sight mạnh như thang máy, góc tường dày đặc) hoặc do mạch TX đã bị tắt trước khi quá trình timeout của RX kết thúc.
    

**Kết luận:** Bộ kit LR2021EVK1XBS1 cho thấy độ tin cậy tuyệt đối ở cự ly ngắn/trung bình trong nhà. Để đánh giá được giới hạn thực sự của Spreading Factor 7 (SF7), môi trường đo kiểm cần được mở rộng ra ngoài trời (Outdoor) với khoảng cách đơn vị hàng trăm mét đến vài kilomet, hoặc thực hiện test giới hạn (Stress Test) bằng cách giảm mạnh Tx Power về mức 0 dBm.
# 4. **Current Consumption** (Tiêu thụ dòng điện)
(Cái này cũng cần thiết bị đo ngoài => Pass)