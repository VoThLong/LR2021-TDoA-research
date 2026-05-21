# 1. Nhảy về đầu não Zephyr
cd /home/dashtrad/lora_usp_workspace

# 2. Kích hoạt môi trường ảo
source .venv/bin/activate

# 3. Ra lệnh build và trỏ đường dẫn tuyệt đối đến thư mục chứa code của ông
west build -b xiao_nrf54l15/nrf54l15/cpuapp --shield semtech_loraplus_expansion_board --shield semtech_wio_lr2021 /home/dashtrad/Documents/lr2021-tdoa-firmware/src
