# Báo cáo Kỹ thuật 3.4: Tích hợp Demo Ranging và Chuyên nghiệp hóa Dự án

**Ngày:** 25 tháng 05, 2026
**Dự án:** Phát triển Firmware TDoA cho LR2021
**Nền tảng:** XIAO nRF54L15 + Semtech LR2021 (LoRa Plus)
**Mục tiêu:** Di trú demo đo khoảng cách độ chính xác cao, tái cấu trúc kho lưu trữ để đạt tính độc lập và chuẩn hóa dự án cho hợp tác quốc tế.

---

## 1. Tóm tắt điều hành

Giai đoạn này tập trung vào việc mở rộng khả năng của dự án từ giao tiếp cơ bản (PingPong) sang định vị nâng cao (Ranging/ToF). Các thành tựu chính bao gồm việc nội bộ hóa thành công logic Ranging của Semtech, triển khai giao diện dòng lệnh chuyên nghiệp hoàn toàn bằng tiếng Anh, và xử lý dứt điểm các lỗi hiển thị đặc thù trên board đánh giá LR2021EVK1XBS1. Dự án hiện đã được cấu trúc như một kho lưu trữ firmware chuyên nghiệp, có tính di động cao và tự động hóa hoàn toàn.

---

## 2. Khám phá và Phân tích Demo Ranging

### 2.1. Xác minh tính năng
Quá trình khảo sát không gian làm việc Semtech USP (`lora_usp_workspace`) đã xác định được một bản demo Ranging hoàn chỉnh tại `application/samples/usp/sdk/ranging_demo`. Khác với các bản mẫu trước đó, phiên bản này đã hoàn thiện về mặt chức năng:
*   **Quản lý Vai trò:** Hỗ trợ rõ ràng cho `RANGING_DEVICE_MODE_MANAGER` (Máy chủ) và `RANGING_DEVICE_MODE_SUBORDINATE` (Máy tớ).
*   **Hỗ trợ LR2021 bản địa:** Đã xác minh các trình điều khiển (drivers) và khai báo devicetree cho dòng LR20xx.
*   **Độ tin cậy thống kê:** Triển khai thuật toán lọc trung vị (median filtering) để giảm thiểu nhiễu đa đường (multipath interference).

### 2.2. Phân tích Thuật toán
Bộ máy Ranging sử dụng kỹ thuật **Thời gian bay khứ hồi (Round Trip Time of Flight - RTTOF)** kết hợp với **Nhảy tần (Frequency Hopping)**:
1.  **Bắt tay (Handshake):** Manager gửi gói tin LoRa để đồng bộ các tham số nhảy tần.
2.  **Giai đoạn Nhảy tần:** Bộ máy RTTOF phần cứng thực hiện các phép đo ở mức dưới nano giây trên hơn 30 kênh tần số.
3.  **Hiệu chỉnh:** Sử dụng các bảng tra cứu nội bộ (Delay Indicators) để trừ đi độ trễ lan truyền đặc thù của chip.
4.  **Tính toán cuối cùng:** $\text{Khoảng cách} = (\text{Thời gian đo} - \text{Độ lệch}) \times \frac{c}{2}$

---

## 3. Tái cấu trúc Kiến trúc

### 3.1. Nội bộ hóa Độc lập (Mô hình OOT)
Để đảm bảo kho lưu trữ có tính di động và không phụ thuộc vào các sửa đổi bên trong không gian làm việc của nhà cung cấp, các tệp sau đã được đưa về repo `lr2021-tdoa-firmware`:
*   **Mã nguồn:** Chuyển vào `src/ranging/` và `src/pingpong/`.
*   **Mã nhị phân Firmware:** Sao chép `LoRaStudio_...elf` vào `bin/` để phục vụ việc khôi phục trạng thái xuất xưởng tại chỗ.
*   **Tệp tiêu đề (Headers):** Tất cả các cấu hình ứng dụng đã được chuyển về địa phương.

### 3.2. Chuẩn hóa Tiếng Anh Chuyên nghiệp
Theo tiêu chuẩn kỹ thuật quốc tế:
*   **Loại bỏ biểu tượng:** Tất cả các nhắc lệnh terminal và nhật ký script đã được chuyển sang dạng văn bản thuần túy.
*   **Chuyển đổi ngôn ngữ:** Biên dịch toàn bộ tài liệu (README), chú thích script và thông báo trạng thái CMake từ tiếng Việt sang tiếng Anh kỹ thuật trang trọng.
*   **Giao diện sạch:** Chuẩn hóa báo lỗi và thông báo thành công trên tất cả các script tự động hóa.

---

## 4. Gỡ lỗi và Tối ưu hóa Phần cứng

### 4.1. Khắc phục hiển thị OLED (SSD1306)
Các thử nghiệm ban đầu cho thấy màn hình OLED bị tối đen. Vấn đề được chẩn đoán là do sự kết hợp của việc tắt trình điều khiển trong phần mềm và thiếu khai báo trong devicetree:
*   **Kích hoạt phần mềm:** Bật `CONFIG_I2C`, `CONFIG_DISPLAY`, và `CONFIG_SSD1306` trong `prj.conf`.
*   **Cấu hình Devicetree Overlay:** Cập nhật `xiao_nrf54l15_nrf54l15_cpuapp.overlay` để:
    *   Bật `&i2c22` (cổng I2C tiêu chuẩn của XIAO).
    *   Định nghĩa node `ssd1306@3c`.
    *   Gán đường dẫn hiển thị mặc định cho `zephyr,display`.

### 4.2. Xác minh tín hiệu (Chế độ liên tục)
Để bỏ qua yêu cầu nhấn nút thủ công trong quá trình thử nghiệm RF ban đầu, macro `CONTINUOUS_RANGING` đã được bật tạm thời. Điều này cho phép xác minh ngay lập tức:
*   **Chỉ báo OLED:** Các ký hiệu "M" (Manager) và "S" (Subordinate).
*   **Phổ RF:** Các xung LoRa xuất hiện liên tục trong dải băng tần 868MHz thông qua việc nhảy tần.

---

## 5. Phân tích Kỹ thuật về Sai số Phép đo

Một quan sát quan trọng đã được ghi nhận khi các thiết bị đặt cạnh nhau báo kết quả **-14 mét**.
*   **Nguyên nhân:** Các bảng hiệu chuẩn nội bộ của firmware được tinh chỉnh cho phần cứng tham chiếu của Semtech. Sự kết hợp giữa XIAO và Shield thực tế có độ trễ lan truyền nội bộ thấp hơn hằng số bị trừ đi trong phần mềm.
*   **Giải pháp:** Một biến `offset` phần mềm đã được xác định trong `app_ranging_hopping.c` để phục vụ việc hiệu chuẩn "đưa về mức 0" cho hệ thống trong tương lai.

---

## 6. Bộ công cụ Tự động hóa

Kiến trúc script đã được "làm phẳng" từ mô hình lồng nhau sang các script độc lập để đạt được sự minh bạch tối đa:
*   `flash_factory.sh`: Khôi phục phần cứng về trạng thái modem gốc bằng tệp ELF nội bộ.
*   `flash_ranging_manager.sh`: Biên dịch và nạp vai trò Manager.
*   `flash_ranging_sub.sh`: Biên dịch và nạp vai trò Subordinate.
*   `flash_pingpong_master.sh`: Biên dịch và nạp PingPong Master.
*   `flash_pingpong_sub.sh`: Biên dịch và nạp PingPong Subordinate.

---

## 7. Kết luận và Trạng thái Hiện tại
Nền tảng firmware hiện đã hoạt động hoàn toàn và được tài liệu hóa chuyên nghiệp. Cả hai đơn vị Manager và Subordinate đã được xác nhận là khởi động đúng, hiển thị trạng thái trên OLED và thực hiện trao đổi dữ liệu ranging liên tục. Dự án đã sẵn sàng cho việc bàn giao hoặc thử nghiệm thực địa.
