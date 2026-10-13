# MyLuma 7 firmware dump

![MyLuma 7 PhotoShare Wi-Fi digital picture frame in Stardust Grey](https://i5.walmartimages.com/seo/MyLuma-7-PhotoShare-WiFi-Digital-Picture-Frame-with-built-in-LEDs-in-Stardust-Grey_11f56d14-caeb-48ec-adce-0eec412ef3ac.d5fc8040820884d49aa943c4cbf9cc1b.jpeg?odnHeight=573&odnWidth=573&odnBg=FFFFFF)

A read-only copy of the stock eMMC user area from a MyLuma 7 PhotoShare frame. The photo is the Stardust Grey frame from the Walmart listing.

## Device

| | |
| --- | --- |
| Model | FML07SD |
| Product | MyLuma 7 PhotoShare Wi-Fi digital picture frame |
| Maker | Simply Smart Home (Switchmate PhotoShare) |
| Firmware | v1.0.14 |
| Application | 4.041 (96) |
| FCC ID | [2AT7Q-MYLUMA](https://fccid.io/2AT7Q-MYLUMA) |
| SoC | Allwinner A133 |
| Board | A100-SWM0813F-V1.0 |

FML07WL is the same board in another color. The FCC filing lists both names.

## The image

`emmc-user.bin` is the eMMC user area, read on 13 October 2026. Two hardware boot areas, 4 MiB each, are not in this file.

| | |
| --- | --- |
| Size | 7,818,182,656 bytes (7,456 MiB) |
| Layout | 15,269,888 sectors of 512 bytes |
| SHA-256 | `954d4115db22f265a8fc709a15892e863276410cc10d06f789e75226cd4531a4` |

GitHub cannot take that file as one object. A file in the git repository has to stay under 100 MB, and a release asset has to stay under 2 GB. The image is split into four equal parts of 1,954,545,664 bytes (about 1.82 GiB). Each part is under the release limit.

The four parts and the smaller captures from the same read are published on release [fml07sd-v1.0.14](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/tag/fml07sd-v1.0.14). Joining the parts in order reproduces `emmc-user.bin`. The SHA-256 of that join matches the full-image hash above.

| Part | Size | Offset | SHA-256 |
| --- | ---: | ---: | --- |
| `emmc-user.part0.bin` | 1,954,545,664 bytes | 0 | `40a15d884d95bab78d92abb58fd94d5a324f1fa9f14339e6281e1f9ec76c9d67` |
| `emmc-user.part1.bin` | 1,954,545,664 bytes | 1,954,545,664 | `0ad8947390350aab97fe322f079dd05fd633bba858f758d7e8d6d5adc9d17da0` |
| `emmc-user.part2.bin` | 1,954,545,664 bytes | 3,909,091,328 | `2297affb7f05ad3e6acd5ceaf5cd9423dd8904cb151779fde6b03cb9ec2741fd` |
| `emmc-user.part3.bin` | 1,954,545,664 bytes | 5,863,636,992 | `72fe9e7436caac933c0e83e3955de231e4551e3a0e6a7c10633999f6284612a2` |

Rebuild and check the image with:

```bash
cat emmc-user.part0.bin emmc-user.part1.bin emmc-user.part2.bin emmc-user.part3.bin > emmc-user.bin
sha256sum -c SHA256SUMS
```

`SHA256SUMS` lists the four parts and the joined image. The same file is in this repository and on the release.

| File | What it is | Download |
| --- | --- | --- |
| `emmc-user.part0.bin` | Bytes 0 through 1,954,545,663 | [part 0](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part0.bin) |
| `emmc-user.part1.bin` | Offset 1,954,545,664 | [part 1](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part1.bin) |
| `emmc-user.part2.bin` | Offset 3,909,091,328 | [part 2](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part2.bin) |
| `emmc-user.part3.bin` | Offset 5,863,636,992 | [part 3](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-user.part3.bin) |
| `SHA256SUMS` | Hashes for the parts and the joined image | [SHA256SUMS](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/SHA256SUMS) |
| `bootrom.bin` | Boot ROM, 64 KiB from address 0 | [bootrom.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom.bin) |
| `bootrom-0x0000.bin` | First 32 KiB of that boot ROM | [bootrom-0x0000.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom-0x0000.bin) |
| `bootrom-0x8000.bin` | Second 32 KiB of that boot ROM | [bootrom-0x8000.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/bootrom-0x8000.bin) |
| `emmc-sector0.bin` | Protective MBR | [emmc-sector0.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-sector0.bin) |
| `emmc-sector1.bin` | Primary GPT header | [emmc-sector1.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-sector1.bin) |
| `gpt-entries.bin` | GPT entries from LBAs 2 through 6 | [gpt-entries.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/gpt-entries.bin) |
| `emmc-extcsd.bin` | 512-byte EXT_CSD | [emmc-extcsd.bin](https://github.com/josephburt/MyLuma-7-Firmware-Dump/releases/download/fml07sd-v1.0.14/emmc-extcsd.bin) |

## Partition map

The user area starts with a protective MBR. LBA 1 is a GPT header (`EFI PART`). The backup header is the last sector, LBA 15,269,887. The first 36 MiB, through LBA 73,727, holds the partition tables and the gap before the first entry.

| Name | Start LBA | End LBA | Size |
| --- | ---: | ---: | ---: |
| bootloader | 73728 | 139263 | 32 MiB |
| env | 139264 | 172031 | 16 MiB |
| boot | 172032 | 237567 | 32 MiB |
| super | 237568 | 4431871 | 2 GiB |
| misc | 4431872 | 4464639 | 16 MiB |
| recovery | 4464640 | 4530175 | 32 MiB |
| cache | 4530176 | 6103039 | 768 MiB |
| vbmeta | 6103040 | 6135807 | 16 MiB |
| vbmeta_system | 6135808 | 6168575 | 16 MiB |
| vbmeta_vendor | 6168576 | 6201343 | 16 MiB |
| metadata | 6201344 | 6234111 | 16 MiB |
| private | 6234112 | 6266879 | 16 MiB |
| frp | 6266880 | 6267903 | 512 KiB |
| empty | 6267904 | 6299647 | 15.5 MiB |
| dtbo | 6299648 | 6303743 | 2 MiB |
| media_data | 6303744 | 6336511 | 16 MiB |
| UDISK | 6336512 | 15269854 | 4,573,871,616 bytes |

`boot` and `recovery` start with an Android boot image (`ANDROID!`). `dtbo` starts with a device-tree overlay header. Every entry uses the Android partition type GUID `EBD0A0A2-B9E5-4433-87C0-68B6B72699C7`.

## How it was read

The user area was copied over USB while the frame was in Allwinner FEL mode. The copy was checked against the first sector, the GPT header, and the backup header at the last sector.
