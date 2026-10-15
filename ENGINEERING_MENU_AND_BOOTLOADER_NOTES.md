# MyLuma 7" Digital Photo Frame — Reverse Engineering & Backdoor Documentation

**Target Device:** MyLuma 7" Smart Digital Photo Frame  
**SoC:** Allwinner A133 (`sun50iw10p1`, Quad-Core Cortex-A53)  
**OS:** Android 10 (Kernel 4.9.191, AOSP build)  
**Wi-Fi Module:** Allwinner XR829 (2.4 GHz 802.11b/g/n only, WPA/WPA2-PSK)  
**Stock Application:** `PSFFrameAPK.apk` (`com.photoshareframe.frameapp`)  

---

## 1. Built-in OEM Engineering Menu ("Backdoor")

The stock photo frame software runs as a restricted kiosk launcher that hides Android system settings. However, the OEM developers included a built-in engineering backdoor inside `PSFFrameAPK.apk` to enable ADB USB debugging without needing FEL mode or hardware flashing.

### How to Access the Engineering Menu:
1. Turn on the device and let it boot into the stock frame interface.
2. Open **Settings** from the main menu.
3. Scroll down and select **Network Diagnostic**.
4. Locate the **Frame ID** text field on the screen (`tvFrameId`).
5. **Tap "Frame ID" 5 times rapidly (within 8 seconds)**.
6. The screen will immediately navigate to the hidden **Engineering Menu** (`EngMenuFragment`).
7. In the Engineering Menu, locate the **USB Debugging** toggle (`switchFrameSettings`) and switch it to **ON**.

### What This Does Under the Hood (Decompiled Smali/Kotlin):
In `NetworkDiagnosticFragment` (`Lh/d/a/q/a/u/y/n/z;`), a rapid click detector listens on `tvFrameId`:

```kotlin
// Time-window click counter on Frame ID:
val now = System.currentTimeMillis()
if (now - lastClickTime < 8000) {
    clickCount++
} else {
    clickCount = 1
}
lastClickTime = now

if (clickCount == 5) {
    Timber.f("Open Eng Menu Screen")
    activity.openEngMenu() // Launches EngMenuFragment
}
```

In `EngMenuFragment` (`Lh/d/a/q/a/u/y/k/j;`), toggling the USB Debugging switch invokes:

```java
// OnUsbDebuggingClick handler:
Settings.Global.putInt(contentResolver, "development_settings_enabled", 1);
Settings.Global.putInt(contentResolver, "adb_enabled", 1);
```

This immediately triggers Android's native `adbd` daemon to enumerate over the USB-C port.

---

## 2. Bootloader Verification & Why Direct `vbmeta` Patching Failed

Attempting to disable AVB (dm-verity) by patching flags inside the `vbmeta` partition causes the frame to display the splash logo and go black after 30 seconds.

### Root Cause:
Allwinner A133 U-Boot (version `2018.05`) packages firmware components inside a proprietary **TOC1** (Table of Contents 1) secure container at LBA `32800`:

```
TOC1 Items:
  Item  0: rootkey              offset=0x00001800 size=0x000011dc
  Item  1: monitor              offset=0x00002c00 size=0x0000036d
  Item  2: monitor              offset=0x00003000 size=0x0000f30c
  Item  3: optee                offset=0x00012400 size=0x0000036b
  Item  4: optee                offset=0x00012800 size=0x0002e100
  Item  5: u-boot               offset=0x00040c00 size=0x0000036c
  Item  6: u-boot               offset=0x00041000 size=0x000c8000
  Item  7: scp                  offset=0x00109000 size=0x00000369
  Item  8: scp                  offset=0x00109400 size=0x00014008
  Item  9: boot                 offset=0x0011d800 size=0x0000036a
  Item 10: vbmeta               offset=0x0011dc00 size=0x0000036c
  Item 11: recovery             offset=0x0011e000 size=0x0000036e
  Item 12: dtb                  offset=0x0011e400 size=0x00024a00
  Item 13: board-cfg            offset=0x00143000 size=0x00000400
```

`Item 10` is an X.509 certificate containing an extension that hardcodes the exact SHA256 hash of the factory `vbmeta` partition (first 8192 bytes):
```
0F41FDDE8C3B938C36D0C37D53AA4785F0BD09E74F93AB25036A258DFAA8169C
```

When U-Boot boots:
1. It reads `vbmeta` from eMMC partition 7 (LBA `6103040`).
2. It computes `SHA256(vbmeta[0:8192])` and compares it to the digest in `Item 10`.
3. If the bytes are modified (e.g. setting AVB flags to `3`), the hash check fails with `vbmeta self verify failed`.
4. U-Boot executes `onoff:DISP_LCD_BACKLIGHT_DISABLE`, cuts power to the panel backlight, and halts.

**Conclusion:** The stock `vbmeta` partition must remain untouched. ADB should be enabled through the OEM backdoor in user space rather than by patching partition images.

---

## 3. Wi-Fi Module & Driver Quirks

- **Chipset:** Allwinner XR829 SDIO Wi-Fi / Bluetooth combo chip.
- **Band Support:** 2.4 GHz only (Channels 1–11 US). Does not support 5 GHz networks.
- **Stock App Bug:** In `PSFFrameAPK`, if a Wi-Fi password fails once or times out, the app stores the stale configuration in Android's `WifiManager` without overwriting `preSharedKey` on retry. The frame appears stuck and unable to authenticate.
- **Workaround:** Once ADB is enabled, standard Android Wi-Fi settings can be launched directly:
  ```bash
  adb shell am start -a android.settings.WIFI_SETTINGS
  ```
  Connecting through the native Android Settings dialog bypasses the stock app bug completely.

---

## 4. Enabling Persistent ADB over Wi-Fi (Port 5555)

Once USB ADB is active:
```bash
adb shell setprop service.adb.tcp.port 5555
adb shell stop adbd
adb shell start adbd
```
You can then disconnect the USB cable and manage the device wirelessly:
```bash
adb connect <FRAME_IP_ADDRESS>:5555
```
