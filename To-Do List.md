# To-Do List

## Tuần 4 + 5

**Mô tả**:

- Đo đạt thực tế
- Port code ranging_demo lên NiceRF LR2021 DevKit

## Tuần 3

**Mô tả**:
- Lên kế hoạch kiểm thử độ chính xác của ranging_demo trên các đời transceiver khác nhau.
- Làm quen với board NiceRF LR2021 DevKit của TTLab - [nguyenmanhthao996tn/NiceRF_LR2021_DevKit]([https://www.semtech.com/products/wireless-rf/lora-plus/lr2021evk1xcs1](https://github.com/nguyenmanhthao996tn/NiceRF_LR2021_DevKit))

### Sub-tasks:

- [ ] Lên kết hoạch đo, chuẩn bị phần cứng và so sánh khoảng cách thực tế của ranging_demo ở các mức: 1m, 10m, 100m, 1km, 3km, etc. (Ví dụ: Kịch bản đo board nào trước/sau? Cấp nguồn cho các board như thế nào? Làm sao kích hoạt một lần đo? Làm sao lấy kết quả?)
- [ ] Setup môi trường làm việc và bộ thư viện chuẩn cho NiceRF LR2021 DevKit → Chuẩn bị 1 reposistory hoặc 1 README.md hướng dẫn cách setup

---

## Tuần 1 + 2

**Mô tả**:
- Làm quen với board Dev Kit LR2021 của Semtech - [LR2021EVK1XBS1](https://www.semtech.com/products/wireless-rf/lora-plus/lr2021evk1xcs1)
- Chuẩn bị code ranging_demo demo trên các dev kit sử dụng transceiver đời trước
  - LR1110 - [LR1110DVK1TBKS](https://www.semtech.com/products/wireless-rf/lora-edge/lr1110dvk1tbks)
  - SX1280 - [SX1280DVK1ZHP](https://www.semtech.com/products/wireless-rf/lora-connect/sx1280dvk1zhp)

### Sub-tasks:

- [x] LoRaStudio: Setup và thử nghiệm các tính năng, demo có sẵn dành cho Dev Kit LR2021 của Semtech
- [x] **Tìm hiểu*** reposistory code [Lora-net/usp](https://github.com/Lora-net/usp) cho LR2021
- [x] **Tìm hiểu*** reposistory code [Lora-net/usp_zephyr]([https://github.com/Lora-net/usp](https://github.com/Lora-net/usp_zephyr)) cho LR2021
- [ ] Triển khai example **main/examples/main_examples/ranging_demo** trong [Lora-net/usp](https://github.com/Lora-net/usp) và [Lora-net/usp_zephyr]([https://github.com/Lora-net/usp](https://github.com/Lora-net/usp_zephyr))
- [x] 🔴 **QUAN TRỌNG - Viết README.md cho repo này**. (Phải có giới thiệu project chứ, có thể sử dụng [LR2021_TDoA_Project_Proposal.md](https://gist.github.com/nguyenmanhthao996tn/09e5b96183fd209d3801287c66844442))

* Các câu hỏi: Code này là gì? Làm sao sử dụng? etc.

---
