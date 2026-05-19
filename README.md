# Next-Gen Asset Tracking using LR2021 Multi-band TDoA/RTToF

Dự án nghiên cứu về định vị tài sản dựa trên công nghệ LoRa thế hệ mới (**Semtech LR2021 - LoRa Plus™**) được thực hiện bởi nhóm nghiên cứu **The Things Lab (TTLab)** – Khoa KTMT, UIT.

## 1. Giới thiệu (Introduction)

Dự án tập trung khai phá khả năng định vị dựa trên **TDoA (Time Difference of Arrival)** và **RTToF (Round Trip Time of Flight)** sử dụng chip Semtech LR2021. Mục tiêu là tạo ra một hệ thống định vị nội khu có tầm phủ sóng rộng, tiêu thụ năng lượng thấp nhưng vẫn đạt độ chính xác ở cấp độ mét.

## 2. Công nghệ cốt lõi

- **Semtech LR2021 (LoRa Plus™):** Chip thế hệ thứ 4 hỗ trợ đa băng tần (Sub-GHz và 2.4 GHz) với tích hợp Ranging Engine tiên tiến.
- **RTToF (Round Trip Time of Flight):** Xác định khoảng cách bằng cách đo thời gian khứ hồi của tín hiệu.
- **TDoA (Time Difference of Arrival):** Xác định vị trí dựa trên sự chênh lệch thời gian tín hiệu đến các Gateway khác nhau.

## 3. Phạm vi dự án (Project Scope)

1.  **Đo đạc môi trường LOS & NLOS:** Đánh giá sai số lý tưởng và khả năng xuyên vật cản tại hành lang, phòng Lab E6.1.
2.  **So sánh đối chứng (Benchmarking):**
    - So sánh với các thế hệ trước: **SX1280** và **LR1110**.
    - So sánh với công nghệ **UWB** (ví dụ: Apple AirTag).

## 4. Cấu trúc thư mục

- `Docs/`: Chứa các tài liệu hướng dẫn sử dụng, proposal và tài liệu kỹ thuật liên quan.
- `Week_1/`: Quy trình cài đặt môi trường (LoRa Studio, Zephyr) và các nghiên cứu ban đầu.
- `Attachments/`: Các hình ảnh minh họa.
- `To-Do List.md`: Lộ trình và các nhiệm vụ cụ thể theo từng tuần.

## 5. Hướng dẫn bắt đầu

Để bắt đầu với dự án, vui lòng tham khảo các tài liệu hướng dẫn trong thư mục `Week_1/`:
- [Quy trình Setup LoRa Studio](Week_1/1_Quy_Trinh_Set_Up_(LoRa_Studio).md)
- [Quy trình Setup USP Zephyr](Week_1/3_1%20Quy%20trinh%20Set%20Up%20(USP%20Zephyr).md)

## 6. Lộ trình (Roadmap)

Chi tiết công việc được cập nhật thường xuyên tại [To-Do List.md](To-Do List.md).

---
**The Things Lab (TTLab)**
Khoa Kỹ thuật Máy tính - Đại học Công nghệ Thông tin (UIT).
