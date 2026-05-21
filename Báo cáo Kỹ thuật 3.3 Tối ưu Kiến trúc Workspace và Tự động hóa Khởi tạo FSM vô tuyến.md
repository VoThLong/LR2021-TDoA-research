**Đề tài:** Nghiên cứu mã nguồn mở Semtech USP trên nền tảng Zephyr RTOS **Hệ thống mục tiêu:** Seeed Studio XIAO nRF54L15 + Semtech LR2021 **Môi trường thực thi:** Fedora 44 (x86_64) **Giai đoạn:** 3.3 - Hoàn thiện cấu trúc biên dịch độc lập và kịch bản nạp phân quyền (Master/Slave) **Kỹ sư R&D:** VO THANH LONG

## 1. Tối ưu hóa Luồng Biên dịch cho Dự án Độc lập (Out-of-Tree Build)

### 1.1. Phân tích Xung đột Hệ thống

Trong quá trình tổ chức lại mã nguồn thành một kho lưu trữ Git độc lập (`lr2021-tdoa-firmware`), trình điều phối `west` đã từ chối thực thi lệnh `build` với lỗi `unknown command "build"`.

Sự cố này bắt nguồn từ cơ chế quản lý Meta-repository của Zephyr. Công cụ `west` yêu cầu ngữ cảnh hệ thống định tuyến (thư mục ẩn `.west`) để nhận diện Toolchain và nhân Zephyr (`ZEPHYR_BASE`). Việc đứng tại thư mục dự án độc lập nằm ngoài không gian Workspace đã cắt đứt liên kết này.

### 1.2. Giải pháp: Điều hướng Luồng Biên dịch (Build Redirection)

Thay vì sao chép toàn bộ SDK khổng lồ vào dự án, kỹ thuật "Mượn lực Workspace" thông qua tham số điều hướng đã được áp dụng:

```
cd /home/dashtrad/lora_usp_workspace
west build \
    -b xiao_nrf54l15/nrf54l15/cpuapp \
    -d /home/dashtrad/Documents/lr2021-tdoa-firmware/build \
    /home/dashtrad/Documents/lr2021-tdoa-firmware \
    --shield semtech_loraplus_expansion_board \
    --shield semtech_wio_lr2021
```

**Bản chất kỹ thuật:**

- Hệ thống chuyển vùng hoạt động về thư mục Workspace để tận dụng bộ cấu hình hãng.
    
- Cờ chỉ định `-d` ép buộc bộ phân tích CMake xuất toàn bộ tệp nhị phân mục tiêu (`.elf`, `.hex`) về lại thư mục `build/` của dự án độc lập, duy trì nguyên tắc cách ly mã nguồn nghiêm ngặt.
    

## 2. Giải quyết Nghẽn luồng FSM và Tự động hóa Cấu hình Thiết bị

### 2.1. Phân tích Nguyên nhân Treo logic vô tuyến

Thực nghiệm giám sát UART Log trước đó cho thấy cả hai thiết bị đều rơi vào trạng thái `Subordinate` (chế độ chờ thu). Do thiếu định danh Master chủ động kích hoạt bộ khuếch đại công suất (PA) để phát gói tin đầu tiên, hệ thống vĩnh viễn kẹt trong vòng lặp chờ timeout, dẫn đến việc phổ vô tuyến hoàn toàn trống rỗng.

### 2.2. Xây dựng Script Phân quyền và Vượt rào cản Trình nạp (Memory Diffing Bypass)

Để khắc phục, một kịch bản nạp tự động (`flash_master.sh`) đã được phát triển để phân định rõ ràng Machine State (Master/Slave) cho từng nút mạng. Kịch bản này phải xử lý các ràng buộc khắt khe của Toolchain:

```
# 1. Biên dịch phân định (Pristine Build)
west build -p always -b xiao_nrf54l15/nrf54l15/cpuapp ...

# 2. Xóa toàn bộ NVM và Nạp mã cưỡng bức
west flash -i $1 -- --erase
```

**Các can thiệp cấp thấp:**

1. **Dọn dẹp Cache Biên dịch (`-p always`):** Đảm bảo CMake không tái sử dụng các tệp tin cấu hình bộ nhớ và Device Tree cũ, một thao tác sống còn khi thay đổi logic Master/Slave. Bắt buộc phải ánh xạ lại đầy đủ các chân SPI SCK/MOSI từ tệp Overlay.
    
2. **Cưỡng chế Xóa Flash (`-- --erase`):** Vô hiệu hóa thuật toán tối ưu chu kỳ ghi của trình nạp `pyocd`. Bằng cách ra lệnh Mass Erase, chip nRF54L15 bị ép buộc ghi lại toàn bộ sector dữ liệu mới, ngăn chặn hiện tượng `skipped` do Toolchain lầm tưởng mã mới quá giống mã cũ.
    

## 3. Kết luận

Các giải pháp trên đã hoàn thiện khung làm việc chuẩn công nghiệp cho dự án. Thiết bị giờ đây đã được tách biệt mã nguồn an toàn trên một Repo độc lập, đồng thời luồng khởi tạo vô tuyến đã được kích hoạt thành công bằng cách cấu hình phân định tĩnh Master/Slave trực tiếp qua script tự động hóa. Hệ thống sẵn sàng bước vào giai đoạn đo kiểm sóng vô tuyến thực tế.