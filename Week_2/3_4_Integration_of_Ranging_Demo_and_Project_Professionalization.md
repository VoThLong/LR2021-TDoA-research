# Technical Report 3.4: Ranging Demo Integration and Project Professionalization

**Date:** May 25, 2026
**Project:** LR2021 TDoA Firmware Development
**Platform:** XIAO nRF54L15 + Semtech LR2021 (LoRa Plus)
**Objective:** Migrate the high-precision ranging demo, restructure the repository for standalone independence, and standardize the project for international collaboration.

---

## 1. Executive Summary

This phase focused on expanding the project's capabilities from basic communication (PingPong) to advanced localization (Ranging/ToF). Key achievements include the successful localization of Semtech's Ranging logic, the implementation of a professional English-only command interface, and the resolution of hardware-specific display issues on the LR2021EVK1XBS1 evaluation board. The project is now structured as a professional, portable, and fully automated firmware repository.

---

## 2. Ranging Demo Discovery and Analysis

### 2.1. Feature Verification
The investigation of the Semtech USP workspace (`lora_usp_workspace`) identified a mature Ranging demo at `application/samples/usp/sdk/ranging_demo`. Unlike previous samples, this version was found to be functionally complete:
*   **Role Management:** Explicit support for `RANGING_DEVICE_MODE_MANAGER` and `RANGING_DEVICE_MODE_SUBORDINATE`.
*   **LR2021 Native Support:** Verified drivers and devicetree bindings for the LR20xx series.
*   **Statistical Robustness:** Implementation of median filtering to mitigate multipath interference.

### 2.2. Algorithmic Deep Dive
The Ranging engine utilizes **Round Trip Time of Flight (RTTOF)** combined with **Frequency Hopping**:
1.  **Handshake:** Manager sends a LoRa packet to synchronize hopping parameters.
2.  **Hopping Phase:** The hardware RTTOF engine performs sub-nanosecond measurements across 30+ channels.
3.  **Correction:** Uses internal lookup tables (Delay Indicators) to subtract chip-specific propagation delays.
4.  **Final Calculation:** $Distance = (Measured\_Time - Offset) \times \frac{c}{2}$

---

## 3. Architectural Restructuring

### 3.1. Standalone Localization (OOT Model)
To ensure the repository is portable and does not rely on modifications within the vendor's workspace, the following files were localized into the `lr2021-tdoa-firmware` repository:
*   **Source Code:** Moved to `src/ranging/` and `src/pingpong/`.
*   **Firmware Binaries:** Copied `LoRaStudio_...elf` to `bin/` for localized factory resets.
*   **Headers:** All application-specific configuration headers were moved locally.

### 3.2. Professional English Standardization
In accordance with international engineering standards:
*   **Removed Emojis/Icons:** All terminal prompts and script logs were converted to plain text.
*   **Language Migration:** Converted all documentation (README), script comments, and CMake status messages from Vietnamese to formal technical English.
*   **Clean Interface:** Standardized error reporting and success messages across all automation scripts.

---

## 4. Hardware Debugging and Optimization

### 4.1. OLED Display (SSD1306) Fix
Initial tests showed blank OLED screens. The issue was diagnosed as a combination of disabled drivers and missing devicetree entries:
*   **Software Activation:** Enabled `CONFIG_I2C`, `CONFIG_DISPLAY`, and `CONFIG_SSD1306` in `prj.conf`.
*   **Devicetree Overlay:** Updated `xiao_nrf54l15_nrf54l15_cpuapp.overlay` to:
    *   Enable `&i2c22` (standard I2C for XIAO).
    *   Define the `ssd1306@3c` node.
    *   Assign the `zephyr,display` chosen path.

### 4.2. Signal Verification (Continuous Mode)
To bypass the requirement of manual button presses during initial RF testing, the `CONTINUOUS_RANGING` macro was temporarily enabled. This allowed for immediate verification of:
*   **OLED Markers:** "M" (Manager) and "S" (Subordinate) indicators.
*   **RF Spectrum:** Consistent LoRa pulses in the 868MHz band via frequency hopping.

---

## 5. Technical Analysis of Measurement Error

A significant observation was made where devices placed side-by-side reported **-14 meters**. 
*   **Cause:** The firmware's internal calibration tables were tuned for Semtech's reference hardware. The XIAO + Shield combination exhibits a lower internal propagation delay than the subtracted constant.
*   **Solution:** A software `offset` variable was identified in `app_ranging_hopping.c` for future calibration to "Zero" the system at close range.

---

## 6. Automation Suite

The script architecture was "flattened" from a nested model to standalone, independent scripts for maximum transparency:
*   `flash_factory.sh`: Restores hardware to original modem state using local ELF.
*   `flash_ranging_manager.sh`: Compiles and flashes the Manager role.
*   `flash_ranging_sub.sh`: Compiles and flashes the Subordinate role.
*   `flash_pingpong_master.sh`: Compiles and flashes the PingPong Master.
*   `flash_pingpong_sub.sh`: Compiles and flashes the PingPong Subordinate.

---

## 7. Conclusion and Current Status
The firmware platform is now fully operational and professionally documented. Both Manager and Subordinate units are verified to boot, display status on OLED, and perform continuous ranging exchanges. The project is ready for handover or field testing.
