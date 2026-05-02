## Geek Zero

~ is a clone of the Flipper, probably developed somewhere around the Shenzhen area, and the PCB is built from scratch based on the open-sourced schematics available. Usually these devices manifest on AliExpress on seemingly non-salient storefronts. Search for 'Upgraded Clipper', 'Upgraded Clipper with Momentum System', 'Electronic Pet Toy', 'Electronic Dolphin', 'Clipper', 'Geek Zero Dolphin 2' and so on.

 Most of the Geek Zero is electronically identical to the original Flipper Zero, but there are a few differences though.

 So far, the identified ones are:

* The One-Time Programmable memory area (OTP) is NOT programmed, and the 'Security Enclave' along with the 'Factory Keys' are missing.
  * These are NOT open-sourced, so DIY-ers will not be able to use some applications and the universal two-factor (U2F) feature. On the plus side, if the main MCU needs to be replaced, it could be sourced from anywhere.
* The NFC/RFID antenna is smaller, and the 125 kHz part seems not to be tuned as well: it has reduced reading range and it is unable to read FDX-B (animal) tags that are operating at 134 kHz.
* The display is different: not only the pixel aspect ratio is off, but the display controller's preferred contrast value is different too.
* My unit's display backlight is not white: it's green below 25% and very distinctly saffron-esque above 50%, presumably because the poor LEDs are being over-biased.
  * This is fixed now. Poor LEDs had 150 mA blasted at them. Ouch.
* The '5V' pin is labelled as 'VSYS', and is only 5V when the USB is plugged in. Otherwise, it's the lithium battery's voltage. So probably there is no boost converter in it.
* Speaking of the built-in battery, it is very tiny (and optimistically labelled: 760 mAh) and its thermistor wire is not connected at all.
* It has some additional hardware too, allegedly. These are:
  * CC1101 antenna is not only on the PCB with traces, but is also routed out via an MCX connector in the back.
  * [Bosch BMI160](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bmi160-ds000.pdf) (the listing calls it 'BM160') inertial sensor.
    * I think it's safe to say that it's missing from mine. The 'BMI Air Mouse' app doesn't work, and there is no Bosch IC on any of the PCBs :)
  * There supposed to be a Hall-sensor on top of the display, but no information about what it is or how it is connected.
    * The white version's backplate shows that it may be read via `PB2` but my black one doesn't say it.
    * I believe it is the missing U23 component. The pinout would match with an [AH1806](https://www.diodes.com/assets/Datasheets/AH1806.pdf), and a small decoupling capacitor is missing too.
    * There is a thin trace that does seem to go to PB2 from the hall sensor's output through a resistor.
  * There is a magnet built in at the back, so you can play with various sensors and switches.

## [Photos](/documentation/geekzero_photos)

There is a scan of the PCBs in the link above.

![](/documentation/geekzero_photos/firmware_reflashed_too_much_contrast_2.jpg)
![](/documentation/geekzero_photos/firmware_reflashed_too_much_contrast_1.jpg)
![](/documentation/geekzero_photos/split_open_2.jpg)
![](/documentation/geekzero_photos/split_open_3.jpg)


## Modified original firmware for the Geek Zero flipper clone

There are a few, minimal modifications, but they do not break compatibility. These are:

* The display is different, and it needed the default contrast value changed.
In `lib/u8g2/u8g2_glue.c`, `CONTRAST_ERC` was reduced to `10`.
* The backlight LED current was excessive.
In `targets/f7/furi_hal/furi_hal_light.c`, `LED_CURRENT_WHITE` was set to  `(20u)`. The default 150 mA for AlGaInP green LEDs were a tad too much, they turned yellow screaming for help. Everything feels much happier at 20 mA.
* The `qflipper` refused to communicate, because the device didn't have valid name.
In `targets/f7/furi_hal/furi_hal_version.c`, in function `furi_hal_version_get_name_ptr()` around line 269-271, `return *furi_hal_version.name == 0x00 ? "geekzero" : furi_hal_version.name;` was added to prevent returning `NULL` when the OTP is not set. Instead, it now returns `geekzero`.
* The battery gauge display shows garbage
In `targets/f7/furi_hal_power_config.c`, `GEEKZERO_ACTUAL_BATTERY_CAPACITY_MAH` is now defined as 760, and `furi_hal_power_gauge_data_memory` structure array's `Q27220DMAddressGasGaugingCEDVProfile1FullChargeCapacity` and `BQ27220DMAddressGasGaugingCEDVProfile1DesignCapacity` values were set to this define.

## Why

It's an open-source project, and someone went through the trouble of building the device from scratch. Wagner had to exist first, in order to get all the other composers responding to Wagner's work. This is just yet another example of this and [is not the only one](https://www.hackster.io/zst123/fcfz-fully-compatible-flipper-zero-e686ba).

This whole thing started when I re-flashed the Momentum firmware and got a completely black screen. The seller was of course not responsive, so I was on my own. Luckily, the code was relatively easy to navigate and is well-documented.

Sadly, these devices have a certain (bad) reputation because some idiots on social media are (mostly pretending) misusing them; in reality, these devices are in fact nothing but an implementation of a microcontroller ecosystem, just like an Arduino. It just happened to be STM32-based and runs a modified version of RTOS, with some quasi-standardised hardware, and has enough developer community around it so it's above critical mass. As a plus, after the first few years of teething problems, the core developers seemingly stopped randomly introducing breaking changes, so I can actually work with it. Collingridge dilemma and the likes.

For scientific research, I am using my own custom hardware for it that I developed on my own, and for this purpose, I am more than happy to use the stock firmware. Until I hit a snag, that is. Theoretically all other firmware versions may be customised, so far the differences are very little.

## Installing

Clone this repo to your favourite happy place on your computer, format the micro SD card inside the Geek Zero, and then execute:
```
./fbt flash_usb_full
```

The OTP is not programmed. The name of the device is set within the firmware.. U2F will throw a 'Certificate error', but otherwise the device is usable.


<picture>
    <source media="(prefers-color-scheme: dark)" srcset="/.github/assets/dark_theme_banner.png">
    <source media="(prefers-color-scheme: light)" srcset="/.github/assets/light_theme_banner.png">
    <img
        alt="A pixel art of a Dophin with text: Flipper Zero Official Repo"
        src="/.github/assets/light_theme_banner.png">
</picture>

# Flipper Zero Firmware

- [Flipper Zero Official Website](https://flipperzero.one). A simple way to explain to your friends what Flipper Zero can do.
- [Flipper Zero Firmware Update](https://flipperzero.one/update). Improvements for your dolphin: latest firmware releases, upgrade tools for PC and mobile devices.
- [User Documentation](https://docs.flipper.net). Learn more about your dolphin: specs, usage guides, and anything you want to ask.
- [Developer Documentation](https://developer.flipper.net/flipperzero/doxygen). Dive into the Flipper Zero Firmware source code: build system, firmware structure, and more.

# Contributing

Our main goal is to build a healthy and sustainable community around Flipper, so we're open to any new ideas and contributions. We also have some rules and taboos here, so please read this page and our [Code of Conduct](/CODE_OF_CONDUCT.md) carefully.

## I need help

The best place to search for answers is our [User Documentation](https://docs.flipper.net). If you can't find the answer there, check our [Discord Server](https://flipp.dev/discord) or our [Forum](https://forum.flipperzero.one/). If you want to contribute to the firmware development or modify it for your own needs, you can also check our [Developer Documentation](https://developer.flipper.net/flipperzero/doxygen).

## I want to report an issue

If you've found an issue and want to report it, please check our [Issues](https://github.com/flipperdevices/flipperzero-firmware/issues) page. Make sure the description contains information about the firmware version you're using, your platform, and a clear explanation of the steps to reproduce the issue.

## I want to contribute code

Before opening a PR, please confirm that your changes must be contained in the firmware. Many ideas can easily be implemented as external applications and published in the [Flipper Application Catalog](https://github.com/flipperdevices/flipper-application-catalog). If you are unsure, reach out to us on the [Discord Server](https://flipp.dev/discord) or the [Issues](https://github.com/flipperdevices/flipperzero-firmware/issues) page, and we'll help you find the right place for your code.

Also, please read our [Contribution Guide](/CONTRIBUTING.md) and our [Coding Style](/CODING_STYLE.md), and make sure your code is compatible with our [Project License](/LICENSE).

Finally, open a [Pull Request](https://github.com/flipperdevices/flipperzero-firmware/pulls) and make sure that CI/CD statuses are all green.

# Development

Flipper Zero Firmware is written in C, with some bits and pieces written in C++ and armv7m assembly languages. An intermediate level of C knowledge is recommended for comfortable programming. C, C++, and armv7m assembly languages are supported for Flipper applications.

# Firmware RoadMap

[Firmware RoadMap Miro Board](https://miro.com/app/board/uXjVO_3D6xU=/)

## Requirements

Supported development platforms:

- Windows 10+ with PowerShell and Git (x86_64)
- macOS 12+ with Command Line tools (x86_64, arm64)
- Ubuntu 20.04+ with build-essential and Git (x86_64)

Supported in-circuit debuggers (optional but highly recommended):

- [Flipper Zero Wi-Fi Development Board](https://shop.flipperzero.one/products/wifi-devboard)
- CMSIS-DAP compatible: Raspberry Pi Debug Probe and etc...
- ST-Link (v2, v3, v3mods)
- J-Link

Flipper Build System will take care of all the other dependencies.

## Cloning source code

Make sure you have enough space and clone the source code:

```shell
git clone --recursive https://github.com/flipperdevices/flipperzero-firmware.git
```

## Building

Build firmware using Flipper Build Tool:

```shell
./fbt
```

## Flashing firmware using an in-circuit debugger

Connect your in-circuit debugger to your Flipper and flash firmware using Flipper Build Tool:

```shell
./fbt flash
```

## Flashing firmware using USB

Make sure your Flipper is on, and your firmware is functioning. Connect your Flipper with a USB cable and flash firmware using Flipper Build Tool:

```shell
./fbt flash_usb
```

## Documentation

- [Flipper Build Tool](/documentation/fbt.md) - building, flashing, and debugging Flipper software
- [Applications](/documentation/AppsOnSDCard.md), [Application Manifest](/documentation/AppManifests.md) - developing, building, deploying, and debugging Flipper applications
- [Hardware combos and Un-bricking](/documentation/KeyCombo.md) - recovering your Flipper from the most nasty situations
- [Flipper File Formats](/documentation/file_formats) - everything about how Flipper stores your data and how you can work with it
- [Universal Remotes](/documentation/UniversalRemotes.md) - contributing your infrared remote to the universal remote database
- [Firmware Roadmap](https://miro.com/app/board/uXjVO_3D6xU=/)
- And much more in the [Developer Documentation](https://developer.flipper.net/flipperzero/doxygen)

# Project structure

- `applications`        - Applications and services used in firmware
- `applications_users`  - Place for your additional applications and services
- `assets`              - Assets used by applications and services
- `documentation`       - Documentation generation system configs and input files
- `furi`                - Furi Core: OS-level primitives and helpers
- `lib`                 - Our and 3rd party libraries, drivers, tools and etc...
- `site_scons`          - Build system configuration and modules
- `scripts`             - Supplementary scripts and various python libraries
- `targets`             - Firmware targets: platform specific code

Also, see `ReadMe.md` files inside those directories for further details.

# Links

- Discord: [flipp.dev/discord](https://flipp.dev/discord)
- Website: [flipperzero.one](https://flipperzero.one)
- Forum: [forum.flipperzero.one](https://forum.flipperzero.one/)
- Kickstarter: [kickstarter.com](https://www.kickstarter.com/projects/flipper-devices/flipper-zero-tamagochi-for-hackers)

## SAST Tools

- [PVS-Studio](https://pvs-studio.com/pvs-studio/?utm_source=website&utm_medium=github&utm_campaign=open_source) - static analyzer for C, C++, C#, and Java code.
