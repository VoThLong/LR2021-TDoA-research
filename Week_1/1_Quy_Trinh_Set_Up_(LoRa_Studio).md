# SetUp nhận diện board 
1. Chọn Board LR2021EVK1XBS1_XIAO
2. Nạp driver
3. Kết nối cổng com
4. Hình ảnh kết nối thành công: 
	![[Pasted image 20260518091332.png]]
# Tìm hiểu các yếu tố có trong danh mục configuration

### 1. Thanh công cụ trên cùng (Toolbar)

- **Blink and Reset:** Các nút ngoài cùng bên trái dùng để nháy LED nhận diện board hoặc reset vi điều khiển từ xa.
    
- **Quản lý cấu hình (Folder/Save/Copy/Paste):** Thay vì phải gõ lại thông số bằng tay, bạn có thể thiết lập chuẩn cho `COM3`, ấn nút **Copy** (biểu tượng hai tờ giấy) và sang tab `COM4` ấn **Paste** (biểu tượng bìa kẹp) để đồng bộ 100% cấu hình.
    
- **NVM Reset:** Nút này rất mạnh. Nó xóa sạch bộ nhớ Non-Volatile (cấu hình, log) trên chip đưa về trạng thái xuất xưởng. Chỉ dùng khi mạch bị treo cứng thuật toán hoặc lỗi tràn RAM không rõ nguyên nhân.
    
- **Standalone mode:** Khi bạn gạt công tắc này sang phải, cấu hình hiện tại sẽ được nạp cứng vào board. Sau đó, bạn có thể rút cáp USB, cắm mạch vào sạc dự phòng mang ra ngoài sân test khoảng cách mà không cần laptop.
    

### 2. Common radio settings (Tầng điều khiển phần cứng)

Khu vực này can thiệp vào cách chip tiêu thụ năng lượng và phát sóng:

- **Radio Frequency:** `868000000` Hz (868 MHz). Đây là tần số trung tâm của sóng mang (Carrier wave) chuẩn châu Âu.
    
- **Transmission Power:** `22.0` dBm. Mức công suất phát khá cao (khoảng 158 mW), đủ sức đánh xuyên qua nhiều lớp tường bê tông hầm xe.
    
- **SIMO regulator mode:** Đang cấu hình là `Simo normal mode (DCDC)`. Đây là điểm ăn tiền của các dòng chip LPWAN. Dùng DCDC (băm xung) sẽ tối ưu điện năng hơn rất nhiều so với dùng LDO (sụt áp tuyến tính), đặc biệt quan trọng nếu cấp nguồn bằng pin đồng xu.
    
- **Fallback Mode:** `Standby RC`. Định nghĩa trạng thái mà bộ thu phát vô tuyến (transceiver) sẽ tự động quay về sau khi phát (Tx) hoặc thu (Rx) xong một gói tin để tiết kiệm điện.
    

### 3. LoRa Parameters (Linh hồn của điều chế LoRa)

Đây là các tham số quyết định sự đánh đổi giữa **Tốc độ (Data rate)**, **Khoảng cách (Range)** và **Thời gian phát sóng (Time on Air - ToA)**:

- **LoRa Spreading Factor (SF):** Đang đặt `SF7`. SF là hệ số trải phổ (từ 7 đến 12). Tưởng tượng SF là "độ ngân dài" của một nốt nhạc. Ngân càng dài (SF cao) thì người nghe ở xa càng dễ nhận ra trong môi trường ồn ào (tăng độ nhạy thu, truyền xa hơn), nhưng sẽ gửi được ít dữ liệu hơn trong cùng một khoảng thời gian.
    
- **LoRa Bandwidth (BW):** `125 kHz`. Độ rộng băng tần. Băng thông càng rộng thì chèn được nhiều dữ liệu (tốc độ cao), nhưng tín hiệu dễ bị nhiễu hơn.
    
- **LoRa Coding Rate (CR):** `4/5`. Cơ chế sửa lỗi tiến (Forward Error Correction). Thay vì chỉ gửi 4 bit dữ liệu gốc, hệ thống đính kèm thêm 1 bit dư thừa (redundant bit) để bên thu có thể tự khôi phục dữ liệu nếu bị nhiễu mất mảng tín hiệu.
    
- **LoRa sync word:** `0x12`. Mã định danh mạng phần cứng. Thường `0x34` dùng cho mạng công cộng (LoRaWAN), còn `0x12` dùng cho mạng nội bộ (Private). Nếu 2 thiết bị khác Sync Word, phần cứng sẽ tự vứt gói tin đi từ vòng gửi xe chứ không cần CPU phải xử lý.
    