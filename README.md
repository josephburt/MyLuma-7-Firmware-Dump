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

`emmc-user.bin` is the whole eMMC user area, read on 13 October 2026. The card was not written. Two hardware boot areas, 4 MiB each, are not in the file.

| | |
| --- | --- |
| Size | 7,818,182,656 bytes (7,456 MiB) |
| Layout | 15,269,888 sectors of 512 bytes |
| SHA-256 | `954d4115db22f265a8fc709a15892e863276410cc10d06f789e75226cd4531a4` |

GitHub will not store this file in the repository. A file in git has to stay under 100 MB, and a release asset has to stay under 2 GB. The hash identifies the image.

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

The frame was held in Allwinner FEL mode and the user area was copied over USB. No erase and no write was sent to the eMMC. The copy was checked against the first sector, the GPT header, and the backup header at the last sector.
