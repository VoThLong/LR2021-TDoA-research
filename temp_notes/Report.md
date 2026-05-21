# Báo cáo Kỹ thuật: Phân tích Lỗi Cấu trúc Workspace và Giải pháp Biên dịch

## 1. Hiện tượng Lỗi (Symptom)
Khi thực hiện lệnh biên dịch `west build` trực tiếp tại thư mục dự án độc lập (`/home/dashtrad/Documents/lr2021-tdoa-firmware`), hệ thống trả về thông báo lỗi:
> `west: unknown command "build"; do you need to run this inside a workspace?`

## 2. Nguyên nhân Gốc rễ (Root Cause)
Lỗi này phát sinh từ cơ chế quản lý của công cụ **West** (trình điều phối của Zephyr):

*   **Thiếu ngữ cảnh (Context):** `west` không đơn thuần là một trình biên dịch. Nó là một trình quản lý meta-repository. Để thực hiện được lệnh `build`, nó cần tìm thấy thư mục ẩn `.west` ở thư mục hiện tại hoặc các thư mục cha. Thư mục này định nghĩa đường dẫn tới nhân Zephyr (`ZEPHYR_BASE`).
*   **Dự án Độc lập (Freestanding Application):** Dự án mới nằm trong thư mục `Documents`, hoàn toàn nằm ngoài "vùng phủ sóng" của Workspace gốc mà hãng cung cấp. Do đó, `west` bị mất liên kết với bộ công cụ (Toolchain) và các lệnh mở rộng của Zephyr.

## 3. Giải pháp Khắc phục (Solution - "Mượn lực Workspace")
Thay vì phải copy toàn bộ mã nguồn Zephyr (hàng GB) vào dự án mới, chúng ta sử dụng kỹ thuật **Điều hướng Luồng Biên dịch**:

### Lệnh thực thi chuẩn:
```bash
# 1. Quay về thư mục có quyền điều phối (nơi chứa .west)
cd /home/dashtrad/lora_usp_workspace

# 2. Thực hiện lệnh build với các tham số điều hướng
west build \
    -b xiao_nrf54l15/nrf54l15/cpuapp \
    -d /home/dashtrad/Documents/lr2021-tdoa-firmware/build \
    /home/dashtrad/Documents/lr2021-tdoa-firmware \
    --shield semtech_loraplus_expansion_board \
    --shield semtech_wio_lr2021
```

### Cơ chế hoạt động:
1.  **`cd .../lora_usp_workspace`**: Kích hoạt bộ não của West và bộ Toolchain đã được hãng cấu hình sẵn.
2.  **`-d .../build`**: Ép buộc West phải xuất toàn bộ file trung gian và file đích (`zephyr.elf`, `zephyr.hex`) vào thư mục dự án riêng, giữ cho thư mục gốc luôn sạch sẽ.
3.  **Tham số cuối cùng (`.../lr2021-tdoa-firmware`)**: Chỉ thị cho West rằng ứng dụng cần biên dịch nằm ở địa chỉ này, thay vì ứng dụng mặc định của hãng.

## 4. Kết luận
Phương pháp này giúp duy trì một dự án **gọn nhẹ**, dễ quản lý bằng Git riêng, nhưng vẫn tận dụng được toàn bộ sức mạnh và thư viện của bộ SDK gốc mà không làm thay đổi hay hỏng hóc mã nguồn của nhà sản xuất.
