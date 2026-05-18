# Bản dịch LoRaStudio_UserGuide.pdf

Dưới đây là bản dịch toàn bộ tài liệu `LoRaStudio_UserGuide.pdf` từ tiếng Anh sang tiếng Việt:

---

**Tên tệp:** `LoRaStudio_UserGuide.pdf`

---

## ===== Trang 1 =====

### **Hướng dẫn sử dụng LoRa Studio**

Chào mừng bạn đến với **LoRa® Studio v1.5.2**!

LoRa Studio là một ứng dụng minh họa các khả năng và tính năng của chip radio Semtech®. Ứng dụng cho phép bạn cấu hình, kiểm tra và trực quan hóa hiệu suất của dòng Gen 4 trong nhiều kịch bản khác nhau. Tất cả điều này mà không cần viết dù chỉ một dòng mã!

Vui lòng đảm bảo bạn đã cập nhật phiên bản mới nhất bằng cách truy cập trang web LoRa PLUS!

### **Bắt đầu**

#### **Các bộ phát triển được hỗ trợ**

LoRa Studio hỗ trợ nhiều loại bộ phát triển khác nhau, một loại dựa trên vi điều khiển STM32 và hai loại khác dựa trên Nordic nRF54. Như bạn sẽ thấy trong các phần tiếp theo, tên bộ sản phẩm bao gồm mã sản phẩm của chip radio theo sau là loại bộ phát triển.

**LoRa Plus™ ARDUINO® LEGACY (STM32®)**
Dựa trên nền tảng STM32L4, bộ phát triển này bao gồm một Nucleo® STM32L476RG, một bo mạch chuyển đổi để xử lý chân cắm Arduino và bo mạch mở rộng radio. Bộ sản phẩm này có 3 mã sản phẩm khác nhau cho mỗi chip radio, tùy theo khu vực:
- `LR2021EVK1XBS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2021, 868MHz cho Châu Âu
- `LR2021EVK1XCS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2021, 915MHz cho Bắc Mỹ
- `LR2021EVK1XGS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2021, 490MHz cho Trung Quốc và Châu Á
- `LR2022EVK1XBS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2022, 868MHz cho Châu Âu
- `LR2022EVK1XCS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2022, 915MHz cho Bắc Mỹ
- `LR2022EVK1XGS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2022, 490MHz cho Trung Quốc và Châu Á
- `LR2012EVK1XBS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2012, 868MHz cho Châu Âu
- `LR2012EVK1XCS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2012, 915MHz cho Bắc Mỹ
- `LR2012EVK1XGS1_ARDUINO_LEGACY` : Bộ phát triển LoRa Plus™, LR2012, 490MHz cho Trung Quốc và Châu Á

**LoRa Plus™ ARDUINO nRF54L15 Dev Kit**
Dựa trên nền tảng nRF54L15, bộ phát triển này bao gồm một nRF54L15-DK, một bo mạch chuyển đổi để xử lý chân cắm Arduino và bo mạch mở rộng radio. Bộ sản phẩm này có 3 mã sản phẩm khác nhau cho mỗi chip radio, tùy theo khu vực:
- `LR2021EVK1XBS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2021, 868MHz cho Châu Âu
- `LR2021EVK1XCS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2021, 915MHz cho Bắc Mỹ
- `LR2021EVK1XGS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2021, 490MHz cho Trung Quốc và Châu Á
- `LR2022EVK1XBS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2022, 868MHz cho Châu Âu
- `LR2022EVK1XCS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2022, 915MHz cho Bắc Mỹ
- `LR2022EVK1XGS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2022, 490MHz cho Trung Quốc và Châu Á
- `LR2012EVK1XBS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2012, 868MHz cho Châu Âu
- `LR2012EVK1XCS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2012, 915MHz cho Bắc Mỹ
- `LR2012EVK1XGS1_ARDUINO` : Bộ phát triển LoRa Plus™, LR2012, 490MHz cho Trung Quốc và Châu Á

**LoRa Plus™ XIAO Dev Kit**

---

## ===== Trang 2 =====

Dựa trên nền tảng Seeeduno XIAO, bộ phát triển này bao gồm sự tích hợp mới nhất của vi điều khiển nRF54L15 trong hệ số dạng XIAO nhỏ gọn và radio LR2021, dưới dạng mô-đun hoặc bo mạch mở rộng. Bộ sản phẩm này có 3 mã sản phẩm khác nhau cho mỗi chip radio, tùy theo khu vực:
- `LR2021EVK1XBS1_XIAO`: Bộ phát triển LoRa Plus™, LR2021, 868MHz cho Châu Âu
- `LR2021EVK1XCS1_XIAO`: Bộ phát triển LoRa Plus™, LR2021, 915MHz cho Bắc Mỹ
- `LR2021EVK1XGS1_XIAO`: Bộ phát triển LoRa Plus™, LR2021, 490MHz cho Trung Quốc và Châu Á
- `LR2022EVK1XBS1_XIAO`: Bộ phát triển LoRa Plus™, LR2022, 868MHz cho Châu Âu
- `LR2022EVK1XCS1_XIAO`: Bộ phát triển LoRa Plus™, LR2022, 915MHz cho Bắc Mỹ
- `LR2022EVK1XGS1_XIAO`: Bộ phát triển LoRa Plus™, LR2022, 490MHz cho Trung Quốc và Châu Á
- *(Lưu ý: Tài liệu gốc có lặp lại mã LR2021)*

#### **Cài đặt trình điều khiển**

**Bộ sản phẩm dựa trên STM32L4**

Trên Windows, để đảm bảo bạn có thể kết nối đúng với bộ sản phẩm này, bạn cần cài đặt trình điều khiển ST-LINK USB có sẵn tại đây: [ST-LINK USB driver].

Trên Linux, bạn có thể cài đặt bộ công cụ stlink theo hướng dẫn trên trang GitHub nhưng hãy đảm bảo phiên bản HĐH của bạn được hỗ trợ.

Nếu không, bạn cần tạo một quy tắc udev để cho phép truy cập thiết bị không cần quyền root:
Tạo một tệp có tên `/etc/udev/rules.d/49-stlinkv2-1.rules` với nội dung sau:
```bash
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374a", MODE=="0666", SYMLINK+="stlinkv2-1"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="374b", MODE=="0666", SYMLINK+="stlinkv2-1"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="3752", MODE=="0666", SYMLINK+="stlinkv2-1"
```
Sau đó tải lại quy tắc udev với lệnh:
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```
Sau đó, rút và cắm lại thiết bị. Bạn sẽ có thể truy cập thiết bị mà không cần quyền root.

Trong cả hai trường hợp, khi được cài đặt đúng, bạn sẽ thấy một thiết bị lưu trữ lớn thường có tên là `NUCLEO_L476RG` khi cắm bộ sản phẩm vào máy tính. Bạn cũng sẽ thấy một cổng nối tiếp liên kết với thiết bị.

**Bộ sản phẩm dựa trên nRF54L15-DK**

Để xử lý đúng bộ phát triển Nordic Semiconductor®, bạn cần cài đặt công cụ **nRF Connect for Desktop** từ trang web của Nordic.

Vì nRF54L15-DK dựa trên bộ gỡ lỗi J-Link, bạn cũng cần cài đặt trình điều khiển **J-Link** từ trang web của Segger.

Cả hai gói phần mềm đều có sẵn cho cả Windows và Linux.

Sau khi cài đặt cả hai gói phần mềm, hãy cắm nRF54-DK vào máy tính của bạn. Bạn sẽ thấy 2 cổng nối tiếp liên kết với thiết bị. Bạn cần hủy kích hoạt cổng đầu tiên (cổng có số thấp nhất) và đặt điện áp nguồn là 3.3V như hình dưới đây:

---

## ===== Trang 3 =====

[Hình ảnh: Cấu hình nRF54L15 DK]

Đảm bảo ghi lại thay đổi bằng cách nhấp vào nút **Write Config**. Bạn có thể đóng ứng dụng nRF Connect for Desktop.

Bây giờ bạn sẽ có thể sử dụng LoRa Studio với các bộ sản phẩm dựa trên nRF54L15-DK.

**Bộ sản phẩm dựa trên Seeduino XIAO**

Trên Windows, tin tốt là: không cần làm gì cả! Mô-đun hoạt động kiểu "cắm và chạy".

Trên Linux, bạn cần tạo một quy tắc udev để cho phép truy cập thiết bị không cần quyền root:
Tạo một tệp có tên `/etc/udev/rules.d/99-xiao.rules` với nội dung sau:
```bash
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2886", ATTRS{idProduct}=="0066", MODE=="0666", SYMLINK+="nrf54"
SUBSYSTEMS=="usb", ATTRS{idVendor}=="2886", ATTRS{idProduct}=="[08]02d", MODE=="0666", ENV{ID_MM_DEVICE_IGNORE}=="1", ENV{ID_MM_PORT_IGNORE}=="1"
```
Sau đó tải lại quy tắc udev với lệnh:
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```
Sau đó, rút và cắm lại thiết bị. Bạn sẽ có thể truy cập thiết bị mà không cần quyền root.

---

## ===== Trang 4 =====

[Hình ảnh: 2021EVEK1XBS1_XIAO]

### **Sử dụng LoRa Studio**

#### **Kết nối bộ phát triển của bạn**

Sau khi cài đặt trình điều khiển đúng cách, bạn có thể kết nối bộ sản phẩm của mình với máy tính. Nếu bộ sản phẩm được trang bị màn hình OLED nhỏ, bạn sẽ thấy logo Semtech cho biết bộ sản phẩm đã được cấp nguồn. Nếu logo không xuất hiện, vui lòng đặt lại bộ sản phẩm theo cách thủ công bằng cách nhấn nút reset.

[Hình ảnh]

<center>Khởi chạy LoRa Studio, đây là giao diện của nó:</center>

[Hình ảnh lớn hơn]

Khi khởi chạy, ba lần đầu tiên bạn sẽ được chuyển hướng đến trang tài liệu này. Sau đó, bạn sẽ thấy Trang Kết nối. Cho đến khi bạn kết nối một bộ sản phẩm, bạn chỉ có thể điều hướng đến trang tài liệu hoặc ở lại trang kết nối.

Nếu bạn có nhiều bộ sản phẩm được kết nối, bạn có thể xác định cổng nối tiếp nào được liên kết với bộ sản phẩm nào bằng cách nhấp vào nút **[Identify]**. Thao tác này sẽ làm cho đèn LED nhấp nháy trên bộ sản phẩm tương ứng.

Bạn cũng có thể đặt lại bộ sản phẩm của mình bằng nút **[Reset]**.

---

## ===== Trang 5 =====

[Hình ảnh]

Khi khởi động, LoRa Studio phát hiện (các) bộ phát triển được kết nối và hiển thị thông tin bộ sản phẩm như hình trên. Nếu không có thiết bị nào được phát hiện, hãy đảm bảo bộ sản phẩm được kết nối đúng cách, trình điều khiển được cài đặt chính xác hoặc xem phần xử lý sự cố bên dưới.

**Lưu ý quan trọng về Tên Bộ sản phẩm:** Loại bộ sản phẩm (nghĩa là MCU mà nó dựa trên) được tự động phát hiện NHƯNG mã sản phẩm phải được chọn theo cách thủ công. Nếu tham chiếu chính xác không được đặt đúng, LoRa Studio có thể không thể vận hành radio của bộ sản phẩm đúng cách dẫn đến hành vi không mong muốn. Vui lòng xác minh rằng tên bộ sản phẩm khớp với bộ phát triển bạn đang sử dụng trước khi tiến hành bất kỳ thao tác nào. Vui lòng tham khảo phần Bộ phát triển được hỗ trợ ở trên để biết tên bộ sản phẩm chính xác.

Sau khi kết nối, LoRa Studio tự động chuyển sang trang **Cấu hình Radio**.

[Hình ảnh]

Từ trang này, bạn có thể cấu hình các tham số radio của bộ sản phẩm. Bạn có thể chọn, trong số các tham số khác, tần số, công suất đầu ra, điều chế và các tham số của nó.

Bạn cũng có thể, giống như trên trang kết nối, xác định và đặt lại bộ sản phẩm đã kết nối bằng các nút **[Identify]** và **[Reset]**.

Khi bạn hài lòng với cấu hình của mình, bạn có thể chuyển đến trang **Ứng dụng** bằng cách nhấp vào biểu tượng **[Application]** trong menu bên trái.

---

## ===== Trang 6 =====

[Hình ảnh]

Từ trang này, bạn có thể chọn, đặt tham số và chạy các ứng dụng trình diễn.

Bạn cũng có thể, giống như trên các trang trước, xác định và đặt lại bộ sản phẩm đã kết nối bằng các nút **[Identify]** và **[Reset]**.

Cuối cùng, trên cả hai trang Cấu hình Radio và Ứng dụng, một thanh công cụ cho phép bạn:
- **[Save]**: Lưu cấu hình vào tệp JSON. Thao tác này sẽ mở một hộp thoại để chọn vị trí và tên tệp.
- **[Load]**: Tải cấu hình từ một tệp.
- **[Copy]**: Sao chép cấu hình vào bộ nhớ tạm: Nội dung JSON có thể được dán bất cứ đâu bạn muốn hoặc...
- **[Paste]**: ... dán nó lên trang của một bộ sản phẩm đã kết nối khác để đảm bảo cả hai sử dụng cùng một tham số chính xác.
- **[Firmware Update]**: Cập nhật chương trình cơ sở của bộ sản phẩm đã kết nối. Thao tác này sẽ mở một hộp thoại để chọn tệp chương trình cơ sở. Tệp chương trình cơ sở phải là tệp .bin và phù hợp với bộ sản phẩm của bạn.
- **[NVM Reset]**: Nút "NVM Reset" cho phép bạn đặt lại NVM của bộ sản phẩm. Thao tác này sẽ xóa tất cả cấu hình và nhật ký được lưu trữ in NVM và đặt lại về trạng thái mặc định. Điều này hữu ích trong trường hợp có bất kỳ sự cố nào với bộ sản phẩm hoặc nếu bạn muốn bắt đầu lại. Vui lòng đảm bảo dừng đúng cách bất kỳ ứng dụng nào đang chạy trước khi sử dụng nút này để tránh bất kỳ sự cố nào. Ngoài ra, hãy sử dụng nó một cách thận trọng vì nó sẽ xóa tất cả dữ liệu được lưu trữ trong NVM mà không cần xác nhận.
- **[Standalone Mode]**: Một nút chế độ độc lập để chuyển đổi giữa chế độ kết nối và chế độ độc lập khi chạy các ứng dụng trình diễn ở chế độ độc lập. Thông tin chi tiết hơn về nút này và chế độ độc lập trong phần Chạy ứng dụng trình diễn bên dưới.

### **Chạy ứng dụng trình diễn**

Từ trang Ứng dụng, bạn có thể chọn và chạy các ứng dụng trình diễn khác nhau. Các ứng dụng có sẵn là:
- Sóng liên tục (Continuous Wave)
- Ping Pong
- Kiểm tra PER (Tỷ lệ lỗi gói tin)
- Chế độ tĩnh để đo mức tiêu thụ dòng điện

Mỗi ứng dụng có các tham số riêng được đặt khi bắt đầu. Bạn có thể chạy ứng dụng khi kết nối với máy tính hoặc ở chế độ độc lập bằng cách cấp nguồn cho (các) bộ sản phẩm bằng sạc dự phòng hoặc pin.

**Chế độ kết nối (Connected mode)**

Trong chế độ kết nối, bộ sản phẩm được cấp nguồn qua kết nối USB với máy tính của bạn và ứng dụng chạy trên máy tính điều khiển bộ sản phẩm. Nhật ký của

---

## ===== Trang 7 =====

1. Chọn (các) bộ sản phẩm của bạn trên trang kết nối và nhấp vào nút **Connect**, ứng dụng sẽ tải và tự động mở trang Cấu hình Radio.
2. Cấu hình các tham số radio theo ý muốn.
3. Chuyển đến trang **Ứng dụng**.
4. Cấu hình các tham số ứng dụng theo ý muốn.
5. Nhấp vào nút **Start** để bắt đầu ứng dụng.
6. Dừng ứng dụng bằng cách nhấp vào nút **Stop**.

Bạn cũng có thể lưu nhật ký vào tệp văn bản bằng cách nhấp vào nút phía trên khu vực nhật ký.

**Chế độ độc lập (Standalone mode)**

Trong chế độ độc lập, (các) bộ sản phẩm được cấp nguồn bởi một nguồn điện bên ngoài (sạc dự phòng, pin, v.v.) và ứng dụng chạy trên bộ sản phẩm điều khiển bản trình diễn. Thông tin về trạng thái của ứng dụng được hiển thị trên màn hình OLED của (các) bộ sản phẩm (nếu có) và nhật ký được lưu trữ trong NVM của (các) bộ sản phẩm.

Để chạy một ứng dụng ở chế độ độc lập, bạn cần:
1. Cắm bộ sản phẩm vào máy tính của bạn.
2. Chọn bộ sản phẩm của bạn trên trang kết nối và nhấp vào nút **Connect**, ứng dụng sẽ tải và tự động mở trang Cấu hình Radio.
3. Cấu hình các tham số radio và các tham số ứng dụng theo ý muốn.
4. Nhấp vào nút **Start** để bắt đầu ứng dụng.
5. Dừng ứng dụng bằng cách nhấp vào nút **Stop**. Bước này là bắt buộc để lưu đúng cấu hình và các tham số ứng dụng vào NVM của bộ sản phẩm.
6. Đặt thiết bị ở chế độ độc lập bằng cách trượt nút standalone **[Standalone]** sang phải. Nút bên trong sẽ chuyển sang màu xanh lá cây và các trường cấu hình sẽ bị vô hiệu hóa (chuyển sang màu xám) để cho biết bộ sản phẩm hiện đang ở chế độ độc lập.
7. Rút bộ sản phẩm khỏi máy tính và cấp nguồn cho nó bằng nguồn điện bên ngoài và đóng LoRaStudio.
8. Để bắt đầu ứng dụng ở chế độ độc lập, nhấn nút người dùng trên bộ sản phẩm.
9. Đảm bảo dừng ứng dụng ở chế độ độc lập bằng cách nhấn lại nút người dùng trên bộ sản phẩm.
10. Kết nối lại bộ sản phẩm với máy tính của bạn.
11. Mở lại LoRa Studio, quay lại trang kết nối, tìm kiếm thiết bị, chọn và kết nối bộ sản phẩm của bạn.
12. Để truy xuất nhật ký của ứng dụng, hãy chuyển đến trang Ứng dụng và hủy kích hoạt chế độ độc lập. Nút bên trong sẽ chuyển sang màu xám và các trường cấu hình sẽ được kích hoạt lại để cho biết bộ sản phẩm hiện đã trở lại chế độ kết nối. Một thông báo (toast) cũng sẽ xuất hiện để cho biết rằng nhật ký đã được truy xuất và lưu vào một tệp văn bản với tên của nó trong thư mục LoRa Studio:
   [Hình ảnh thông báo]
   Nó cũng tự động mở tệp đó cho bạn bằng trình soạn thảo văn bản mặc định của bạn.

### **Chi tiết ứng dụng**

#### **Bản trình diễn Sóng liên tục (Continuous Wave Demo)**

Bản trình diễn Sóng liên tục cho phép bạn truyền tín hiệu sóng liên tục cho mục đích thử nghiệm.
Ứng dụng này có thể được sử dụng để kiểm tra phạm vi của radio, độ nhạy của máy thu hoặc để thực hiện các thử nghiệm quy định.
Nó không có bất kỳ tham số cụ thể nào của ứng dụng. Bạn chỉ cần cấu hình các tham số radio. Chỉ cần sử dụng nút **Start** để bắt đầu truyền và nút **Stop** để dừng nó.

#### **Bản trình diễn Ping Pong**

Bản trình diễn Ping Pong minh họa giao tiếp hai chiều giữa hai thiết bị. Ứng dụng này rõ ràng cần hai bộ sản phẩm để hoạt động cùng nhau. MẸO: Chỉ cấu hình đúng một bộ sản phẩm, sau đó sử dụng chức năng sao chép/dán cấu hình để đảm bảo bạn chạy cùng một tham số trên cả hai bộ sản phẩm.

Bản trình diễn Ping Pong yêu cầu các tham số ứng dụng sau:
- **Độ trễ trước khi truyền**: đặt độ trễ trước khi gói tin đầu tiên được gửi (ms)
- **Độ trễ giữa các gói tin**: đặt độ trễ (ms) giữa việc nhận và truyền lại gói tin
- **Ngưỡng đồng bộ hóa gói tin**: Số lượng gói tin để truyền mà không có phản hồi để tạo đủ biên độ cho sự đồng bộ hóa
- **Kích thước tiền tố**: Số byte được sử dụng làm tiền tố trong gói tin
- **Thời gian chờ nhận**: Thời gian (ms) để chờ phản hồi trước khi coi gói tin bị mất

Khi bắt đầu ứng dụng, bộ sản phẩm sẽ bắt đầu truyền gói tin và chờ phản hồi. Nếu nhận được phản hồi, nó sẽ lấy nội dung của nó (một bộ đếm) và tăng giá trị của nó lên trước khi truyền lại gói tin sau độ trễ đã cấu hình. Nếu không nhận được phản hồi trong thời gian chờ, nó sẽ coi gói tin bị mất và truyền một gói tin mới sau độ trễ đã cấu hình và đặt lại chính nó ở chế độ nhận.

Vì không có sự đồng bộ hóa thực tế nào giữa hai bộ sản phẩm, nên có thể cả hai bộ sản phẩm truyền cùng một lúc và do đó bỏ lỡ gói tin của nhau. Ngưỡng đồng bộ hóa cho phép giảm thiểu vấn đề này bằng cách truyền một số lượng gói tin nhất định mà không cần chờ phản hồi. Điều này làm tăng cơ hội rằng một trong các gói tin sẽ được bộ sản phẩm kia nhận và giao tiếp có thể bắt đầu.

---

## ===== Trang 8 =====

1. Sau khi kết nối với bộ sản phẩm của bạn, ứng dụng sẽ tải và tự động mở trang Cấu hình Radio.
2. Cấu hình các tham số radio (tần số, công suất, điều chế...).
3. Chuyển đến trang **Ứng dụng**.
4. Chọn **Chế độ tĩnh** và thiết lập các tham số ứng dụng:
   - **Chế độ**: Chọn chế độ tĩnh mà bạn muốn đặt bộ sản phẩm.
5. Nhấp vào nút **Start** để đặt bộ sản phẩm ở chế độ tĩnh đã chọn.
6. Bộ sản phẩm sẽ chuyển sang chế độ tĩnh đã chọn.
7. Nhấp vào nút **Stop** để thoát chế độ tĩnh và trở lại hoạt động bình thường.

#### **Cập nhật chương trình cơ sở (Firmware Update)**

Để cập nhật chương trình cơ sở của bộ sản phẩm, bạn cần kết nối nó với máy tính và mở LoRa Studio. Sau đó, trên trang cấu hình radio hoặc trang ứng dụng, hãy nhấp vào nút **[Firmware Update]** trên thanh công cụ. Thao tác này sẽ mở một hộp thoại tệp để chọn tệp chương trình cơ sở. Ứng dụng sẽ tự động mở thư mục nơi có thể tìm thấy chương trình cơ sở. Chọn tệp phù hợp với bộ sản phẩm của bạn.

Khi tệp chương trình cơ sở được chọn, quá trình cập nhật sẽ tự động bắt đầu. Một thanh tiến trình sẽ hiển thị tiến trình cập nhật. Không ngắt kết nối hoặc tắt bộ sản phẩm trong quá trình cập nhật.

Khi quá trình cập nhật hoàn tất, một thông báo sẽ cho biết rằng quá trình cập nhật thành công. Bạn có thể sử dụng chương trình cơ sở mới.

### **Xử lý sự cố (Troubleshooting)**

#### **Các vấn đề kết nối phổ biến**

Trang chính hiển thị thiết bị **"Đã phát hiện nhưng không phản hồi"**:
[Hình ảnh]

Điều này thường có nghĩa là ứng dụng có thể nhìn thấy một cổng nối tiếp để giao tiếp with thiết bị, nhưng thiết bị không phản hồi các lệnh. Có thể thiết bị này không phải là bộ phát triển được hỗ trợ hoặc chương trình cơ sở trên bộ sản phẩm không tương thích với LoRa Studio. Nó cũng có thể là bộ sản phẩm gặp một số vấn đề về UART. Bạn có thể thử các bước sau để giải quyết vấn đề:
- Nhấp vào nút **Search for devices**.
- Nhấp vào nút **"Reset"** liên kết với cổng nối tiếp mà bạn tin rằng bo mạch của mình được kết nối, sau đó nhấp lại nút **"Search"**.
- Đặt lại bộ sản phẩm bằng cách nhấn nút reset và sau đó nhấp lại nút **"Search for devices"**.

*Lưu ý: Đối với các bộ sản phẩm dựa trên nRF54L15-DK, mặc dù bạn đã đảm bảo rằng mình đã hủy kích hoạt cổng nối tiếp đầu tiên trong nRF Connect, hai cổng nối tiếp sẽ được phát hiện và một trong số chúng sẽ luôn không phản hồi. Điều này không ảnh hưởng đến chức năng của LoRa Studio và có thể bỏ qua.*

#### **Không có thiết bị nào hiển thị**
- **Xác minh cài đặt trình điều khiển**: Kiểm tra kỹ rằng tất cả các trình điều khiển cần thiết đã được cài đặt và cập nhật.
- **Khởi động lại ứng dụng**: Đôi khi, chỉ cần khởi động lại LoRa Studio có thể giải quyết các sự cố kết nối.
- **Thử một cổng USB hoặc cáp khác**: Cổng USB hoặc cáp bị lỗi có thể gây ra sự cố kết nối. Chuyển sang một cổng khác hoặc sử dụng cáp khác để xem

---

## ===== Trang 9 =====

điều đó có giải quyết được vấn đề không.
- **Kiểm tra nguồn điện của thiết bị**: Đảm bảo rằng bộ phát triển đã được cấp nguồn và hoạt động bình thường.
- **Kiểm tra xung đột**: Đảm bảo rằng không có ứng dụng nào khác đang sử dụng cùng cổng nối tiếp mà LoRa Studio đang cố gắng truy cập.

#### **Không có kết nối máy chủ**
[Hình ảnh]

Có thể xảy ra trường hợp phần phụ trợ (backend) không khởi động đúng cách và do đó không có thiết bị nào có thể kết nối. Bạn có thể xác minh và khắc phục sự cố này bằng cách nhấp vào thông tin máy chủ ở dưới cùng bên trái của trang kết nối.
[Hình ảnh]

Một hộp thoại sẽ mở ra, nhấp vào nút **Stop local backend**. Một số thông báo (toast) sẽ cho biết việc dừng phần phụ trợ, nút sẽ chuyển thành **Start local backend**. Nhấp lại vào nút này, thao tác này sẽ khởi động lại phần phụ trợ một cách chính xác.
[Hình ảnh]

Cũng có thể một máy chủ RPC khác đang chạy trên máy của bạn và sử dụng cùng cổng với phần phụ trợ LoRa Studio, điều này ngăn nó khởi động đúng cách. Trong trường hợp này, bạn có thể thay đổi cổng được sử dụng bởi phần phụ trợ LoRa Studio bằng cách dừng máy chủ như đã giải thích ở trên và sau đó thay đổi số cổng trong trường nhập liệu trước khi khởi động lại nó.
[Hình ảnh]

#### **Bộ sản phẩm không phản hồi sau một số lệnh**

Nếu bộ sản phẩm ngừng phản hồi sau một số lệnh, điều đó có thể là do chính sách tiết kiệm năng lượng của hệ điều hành đặt cổng USB ở chế độ ngủ. Bạn có thể thử tắt tính năng này trong cài đặt HĐH của mình. Điều này đã được biết là xảy ra trên Windows 11. Chỉ cần bỏ chọn tùy chọn **"Cho phép máy tính tắt thiết bị này để tiết kiệm năng lượng"** cho Bộ điều khiển trung tâm USB (USB Root Hubs) trong Trình quản lý Thiết bị (Device Manager) như hình dưới đây:

[Hình ảnh - không hiển thị trong văn bản]

---
*Bản dịch tài liệu LoRa Studio User Guide.*
