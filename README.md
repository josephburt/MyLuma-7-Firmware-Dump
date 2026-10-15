# MyLuma 7" Wi-Fi Digital Photo Frame — Firmware Dump & Reverse Engineering

<p align="center">
  <img src="assets/device.jpg" alt="MyLuma 7 PhotoShare Wi-Fi Digital Picture Frame" width="650">
</p>

<p align="center">
  <a href="https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/tag/fml07sd-v1.0.14"><img src="https://img.shields.io/badge/Release-fml07sd--v1.0.14-blue?style=flat-square" alt="Release"></a>
  <img src="https://img.shields.io/badge/SoC-Allwinner%20A133-orange?style=flat-square" alt="SoC">
  <img src="https://img.shields.io/badge/OS-Android%2010%20(AOSP)-green?style=flat-square" alt="OS">
  <img src="https://img.shields.io/badge/Kernel-Linux%204.9.191-blueviolet?style=flat-square" alt="Kernel">
  <a href="https://fccid.io/2AT7Q-MYLUMA"><img src="https://img.shields.io/badge/FCC%20ID-2AT7Q--MYLUMA-informational?style=flat-square" alt="FCC ID"></a>
  <img src="https://img.shields.io/badge/eMMC%20Hash-Verified%20Bit--Exact-brightgreen?style=flat-square" alt="Hash Verified">
</p>

---

## Overview

This repository provides a complete, bit-exact backup of the factory eMMC firmware from the **MyLuma 7" PhotoShare Wi-Fi Digital Picture Frame**, read via Allwinner FEL mode.

In addition to raw partition images and verification hashes, this repository documents:
- **The Built-in OEM Engineering Menu ("Backdoor")**: How to enable native Android USB debugging without hardware modification.
- **Bootloader Cryptographic Verification**: Technical analysis of Allwinner A133 U-Boot TOC1 secure container and why direct `vbmeta` modification disables the backlight.
- **Hardware Architecture**: Complete GPT partition layout, Wi-Fi driver quirks, and wireless ADB control.

---

## Hardware Specifications

| Component | Details |
| :--- | :--- |
| **Model** | FML07SD (Stardust Grey) / FML07WL (White) |
| **Product** | MyLuma 7" PhotoShare Wi-Fi Digital Picture Frame with LED ambient backlight |
| **Manufacturer** | Simply Smart Home / Switchmate Home LLC |
| **FCC ID** | [2AT7Q-MYLUMA](https://fccid.io/2AT7Q-MYLUMA) |
| **Mainboard** | `A100-SWM0813F-V1.0` |
| **SoC** | Allwinner A133 (`sun50iw10p1`) — Quad-Core ARM Cortex-A53 @ 1.5 GHz |
| **GPU** | PowerVR GE8300 |
| **RAM** | 1 GB DDR3/DDR4 |
| **Internal Storage** | 8 GB eMMC 5.1 (User area: 7,818,182,656 bytes / 15,269,888 sectors) |
| **Display** | 7.0" TFT LCD, 1024 × 600 resolution, 5-point capacitive multi-touch |
| **Wi-Fi / BT** | Allwinner XR829 SDIO Wi-Fi (802.11b/g/n 2.4 GHz) + Bluetooth 4.2 |
| **Ports** | USB-C (OTG, charging, ADB), MicroSD slot, 5V DC barrel jack, reset pinhole |
| **Operating System** | Android 10 (API 29), Linux Kernel `4.9.191-rt130` |
| **Stock Launcher** | `PSFFrameAPK.apk` (`com.photoshareframe.frameapp`) v4.0.41 (build 96) |

---

## Quick Reference: OEM Engineering Menu Backdoor

The stock photo frame software runs as a restricted kiosk launcher. However, the OEM firmware embeds a hidden Engineering Menu to unlock native Android Developer Settings and ADB without hardware tools:

```
[Settings] ──► [Network Diagnostic] ──► Tap "Frame ID" 5 times rapidly ──► [Engineering Menu] ──► Toggle USB Debugging ON
```

1. Power on the frame and open **Settings** from the main menu.
2. Select **Network Diagnostic**.
3. Tap the **Frame ID** text field **5 times rapidly (within 8 seconds)**.
4. The hidden **`EngMenuFragment`** will open immediately.
5. Switch **USB Debugging** to **ON**.
6. Connect a USB-C cable to your PC; standard `adb devices` will enumerate immediately.

> [!TIP]
> For smali reverse engineering notes, decompiled click-detector source code, and persistent network ADB setup, see **[ENGINEERING_MENU_AND_BOOTLOADER_NOTES.md](ENGINEERING_MENU_AND_BOOTLOADER_NOTES.md)**.

---

## Firmware Dump Details

The complete eMMC user area is **7,818,182,656 bytes** (7,456 MiB / 15,269,888 sectors of 512 bytes).

- **Full User Area SHA-256:**
  ```text
  5fda167b44c361b5400cf0ab10c50584bd77b47596dd8986c586f9f99532ed9b
  ```

Because GitHub enforces a 2 GB file upload limit on release assets, the user area is split into four equal parts of 1,954,545,664 bytes each. All files are hosted on release **[fml07sd-v1.0.14](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/tag/fml07sd-v1.0.14)**.

### Release Files & Checksums

| File | Offset Range | Size | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| [`emmc-user.part0.bin`](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part0.bin) | Bytes 0 – 1,954,545,663 | 1.82 GiB | `d01b2bdd3b5916fb9ae725e7a0a0657716f5091130e82f3879f319b1e00ccb9d` |
| [`emmc-user.part1.bin`](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part1.bin) | Offset 1,954,545,664 | 1.82 GiB | `55c8eb951d240d25de42cd33b9285651dd7aea732193c1c4c82f47e060c358d4` |
| [`emmc-user.part2.bin`](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part2.bin) | Offset 3,909,091,328 | 1.82 GiB | `eca1e032f14a560dfe8ada90d4a3091a057ac2a2b300c1f5064ab9ebd6420766` |
| [`emmc-user.part3.bin`](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part3.bin) | Offset 5,863,636,992 | 1.82 GiB | `3f17aabeef95b8ba331dae887f5a24151c63ce7443e3725675dc7e1eaa5215cf` |
| [`SHA256SUMS`](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/SHA256SUMS) | — | 424 B | `b6e3f28cf674843be665e7d5668e27c1d76378ef8a57ec394ca09b1f23c72635` |

### Auxiliary Captures

| File | Description | Download |
| :--- | :--- | :--- |
| `bootrom.bin` | Allwinner A133 internal Boot ROM (64 KiB from address `0x00000000`) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom.bin) |
| `bootrom-0x0000.bin` | First 32 KiB half of the Boot ROM | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom-0x0000.bin) |
| `bootrom-0x8000.bin` | Second 32 KiB half of the Boot ROM | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom-0x8000.bin) |
| `emmc-sector0.bin` | Protective Master Boot Record (LBA 0) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-sector0.bin) |
| `emmc-sector1.bin` | Primary GUID Partition Table header (LBA 1, `EFI PART`) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-sector1.bin) |
| `gpt-entries.bin` | Partition entries array (LBAs 2 through 6) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/gpt-entries.bin) |
| `emmc-extcsd.bin` | 512-byte eMMC Extended CSD register dump | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-extcsd.bin) |
| `emmc-boot1.bin` | Hardware boot partition 1 (4 MiB, factory all-zeros) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-boot1.bin) |
| `emmc-boot2.bin` | Hardware boot partition 2 (4 MiB, factory all-zeros) | [Download](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-boot2.bin) |

---

## Reassembling the Image

To reconstitute the single, verified `emmc-user.bin` disk image:

```bash
# 1. Download all four split parts and SHA256SUMS
wget https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part{0,1,2,3}.bin
wget https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/SHA256SUMS

# 2. Verify split parts before concatenation
sha256sum --ignore-missing -c SHA256SUMS

# 3. Concatenate parts in numerical order
cat emmc-user.part0.bin emmc-user.part1.bin emmc-user.part2.bin emmc-user.part3.bin > emmc-user.bin

# 4. Verify complete assembled image matches factory checksum
sha256sum -c SHA256SUMS
```

---

## Partition Map

The storage is partitioned using standard GUID Partition Table (GPT) format with a protective MBR at LBA 0. The backup GPT header is located at the final sector (LBA 15,269,887).

All partitions use the Android standard partition type GUID `EBD0A0A2-B9E5-4433-87C0-68B6B72699C7`.

| # | Partition Name | Start LBA | End LBA | Sector Count | Size | Purpose / Contents |
| :---: | :--- | ---: | ---: | ---: | ---: | :--- |
| — | *(MBR + GPT + Gap)* | 0 | 73,727 | 73,728 | 36 MiB | Protective MBR, Primary GPT, TOC1 boot container |
| 1 | `bootloader` | 73,728 | 139,263 | 65,536 | 32 MiB | Secondary bootloader staging |
| 2 | `env` | 139,264 | 172,031 | 32,768 | 16 MiB | U-Boot environment variables |
| 3 | `boot` | 172,032 | 237,567 | 65,536 | 32 MiB | Android Boot Image (`ANDROID!`, Kernel + Ramdisk) |
| 4 | `super` | 237,568 | 4,431,871 | 4,194,304 | 2,048 MiB | Dynamic partitions (`system`, `vendor`, `product`) |
| 5 | `misc` | 4,431,872 | 4,464,639 | 32,768 | 16 MiB | Android bootloader control block (BCB) |
| 6 | `recovery` | 4,464,640 | 4,530,175 | 65,536 | 32 MiB | Android Recovery image (`ANDROID!`) |
| 7 | `cache` | 4,530,176 | 6,103,039 | 1,572,864 | 768 MiB | Ext4 cache partition |
| 8 | `vbmeta` | 6,103,040 | 6,135,807 | 32,768 | 16 MiB | Android Verified Boot metadata (Root AVB) |
| 9 | `vbmeta_system` | 6,135,808 | 6,168,575 | 32,768 | 16 MiB | AVB descriptors for system |
| 10 | `vbmeta_vendor` | 6,168,576 | 6,201,343 | 32,768 | 16 MiB | AVB descriptors for vendor |
| 11 | `metadata` | 6,201,344 | 6,234,111 | 32,768 | 16 MiB | Encryption and metadata store |
| 12 | `private` | 6,234,112 | 6,266,879 | 32,768 | 16 MiB | Factory calibration and serial data |
| 13 | `frp` | 6,266,880 | 6,267,903 | 1,024 | 512 KiB | Factory Reset Protection storage |
| 14 | `empty` | 6,267,904 | 6,299,647 | 31,744 | 15.5 MiB | Unformatted reservation |
| 15 | `dtbo` | 6,299,648 | 6,303,743 | 4,096 | 2 MiB | Device Tree Blob Overlay (`dtbo`) |
| 16 | `media_data` | 6,303,744 | 6,336,511 | 32,768 | 16 MiB | Preloaded OEM demo photos / assets |
| 17 | `UDISK` | 6,336,512 | 15,269,854 | 8,933,343 | 4,362 MiB | `/data` user data partition (Ext4 / F2FS) |

---

## Technical Notes & Acquisition Methodology

- **Acquisition Protocol:** Raw eMMC acquisition performed via USB using Allwinner FEL mode.
- **Verification History:** The initial reader pass introduced an artifact (a 12-byte status record at byte 32,768 of every 64 KiB block). Every affected sector was re-read directly from physical eMMC and corrected. The final 4-part release image is verified bit-exact against physical flash.
- **Hardware Boot Areas:** eMMC hardware boot partitions (`boot1` and `boot2`, 4 MiB each) are unwritten (all zeros, SHA-256 `bb9f8df61474d25e71fa00722318cd387396ca1736605e1248821cc0de3d3af8`). All SPL and TOC1 bootloaders execute entirely out of the eMMC user area.

---

## Repurposing Ideas

Because this device features a quad-core 64-bit ARM processor, 1 GB RAM, Wi-Fi, and a crisp 7" 1024×600 IPS touch display with USB-C power, it makes an ideal platform for:
- **Smart Desk Display / Clock**: Weather, live radar loop, calendar, and news widgets.
- **RTSP IP Camera Monitor**: ExoPlayer low-latency multi-camera surveillance viewer.
- **Home Assistant Kiosk**: Fully Kiosk or custom lightweight webview dashboard.
- **Media Player / Internet Radio**: Squeezelite, MPD, or local UPnP/DLNA streamer.

---

## References & Documentation

- **[ENGINEERING_MENU_AND_BOOTLOADER_NOTES.md](ENGINEERING_MENU_AND_BOOTLOADER_NOTES.md)** — In-depth reverse engineering report on the stock photo frame APK, decompiled smali backdoor, U-Boot TOC1 cryptographic verification, and persistent network ADB configuration.
- **FCC ID Report:** [FCC ID 2AT7Q-MYLUMA](https://fccid.io/2AT7Q-MYLUMA) (Schematics, internal PCB photos, SAR reports).
- **Allwinner Technology:** [Allwinner A133 User Manual & Datasheet](https://linux-sunxi.org/A133) on linux-sunxi.
