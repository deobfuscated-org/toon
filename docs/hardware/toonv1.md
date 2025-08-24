# Toon v1 - hardware specifications

## SoC
[NXP MCIMX27LMOP4A](https://www.nxp.com/products/processors-and-microcontrollers/arm-processors/i-mx-applications-processors/i-mx-mature-processors/multimedia-applications-processors-arm9-core-robust-security-for-mobile:i.MX27L)

* CPU: ARM926EJ-S
* Speed: 400 MHz

[Datasheet](./datasheets/NXP-MCIMX27LMOP4A.pdf)

## Memory
NANYA NT6DM32M32BC-T1

* Size: 128 MiB
* Type: DDR
* Speed: 200MHz

[Datasheet](./datasheets/NANYA-NT6DM32M32BC-T1.pdf)

## Storage
Samsung K9F1G08U0D

* Size: 128 MiB
* Type: NAND Flash

[Datasheet](./datasheets/Samsung-K9F1G08U0D.pdf)

### Partition table
#   | size    | label           | notes
--- | ---     | ---             | ---
0   | 1MiB    | `u-boot`        | Bootloader; not mapped in Linux
1   | 512KiB  | `u-boot-env`    | U-Boot environment variables
2   | 1536KiB | `splash-image`  | Boot splash screen
3   | 3MiB    | `kernel`        | Linux kernel
4   | 3MiB    | `kernel-backup` | Backup Linux kernel
5   | 119MiB  | `rootfs`        | Main root filesystem

## Display
Tianma TM070RDH03

* Size: 7 inch
* Resolution: 800x480 (WVGA), 134PPI, 16:9 aspect ratio
* Type: TFT LCD
* Touch: 4-wire resistive touch screen (see below for controller details)

No datasheet found.

## Interfaces

### Ethernet
* 10/100 Mbps Ethernet port
* Embedded, on-SoC PHY (Freescale Fast Ethernet Controller)

See the SoC's datasheet for more information.

### WiFi
AzureWave AW-NU168H

* Mini PCIe module
* Ralink RT5390 802.11b/g/n chip
* Connected via internal USB2 interface (see USB2 section below).

[Datasheet](./datasheets/AzureWave-AW-NU168H.pdf)

### Z-wave
Sigma designs ZW0301 Z-Wave controller

[Datasheet](./datasheets/Sigma-Designs-ZW0301.pdf)

The Z-Wave controller is connected to the SoC via UART2 (ttymxc2), which is used for communication with the Z-Wave controller.
The Z-Wave controller also has a serial EEPROM connected to it, which is used for data storage.

### Serial EEPROM
Atmel `ATMLH402` (as marked on chip)

* SPI serial EEPROM
* 256 Kbit (32 KiB x 8 bits)
* Address: spi0.0
* Linux driver: `at25`

This EEPROM is likely not supposed to be accessed directly, but rather through the Z-Wave controller's interface.
However, the SPI bus is accessible from Linux, and the EEPROM can be accessed directly.

[Datasheet](./datasheets/Fujitsu-MB85RS256TY.pdf) of a compatible part.

### USB2
Microchip USB3370 (x2)

* Enhanced Single Supply Hi-Speed USB ULPI Transceiver (USB PHY)
* Used to connect the USB port and the Wi-Fi module
* Connected to the SoC's USB2 ULPI interface

[Datasheet](./datasheets/Microchip-USB3370.pdf)


### OpenTherm / power port
Toon v1 has a 2-pin port on its back which is used to connect the Toon to a central heating system or boiler, as well as to provide power.

The pins match up with a connector embedded in the wall mount.

OpenTherm communications are done through UART1 (ttymxc1), at 4800,8n1.

#### Ketelmodule
Commonly, the wall mount connector is connected to a Ketelmodule (boiler module).
This module is a separate device, and allows the Toon to communicate with the boiler as well as provide adequate power to the Toon.

This module is not required for the Toon to function.
You can also connect the wall mount connector directly to 24V DC power, and use the Toon without OpenTherm functionality.

The Toon will sort out polarity fully automatically, so it does not matter which pin is connected to which wire.

### JTAG header
The Toon v1 PCB has a 14-pin JTAG header.
The kind folks at Quby have even populated the header with pins, which makes it super easy to connect a JTAG debugger or Raspberry Pi.

Its pinout is as follows:

Pin | Signal | Description
--- | ---    | ---
1   | RTCK   | Return clock signal
2   | TRST   | Test reset
3   | GND    | Ground
4   | TCK    | Test clock
5   | GND    | Ground
6   | TMS    | Test mode select
7   | SRST   | System reset
8   | TDI    | Test data in
9   | Vt     | ?
10  | TDO    | Test data out
11  | RxD    | UART Receive data
12  | (NC)   | Not connected
13  | TxD    | UART Transmit data
14  | GND    | Ground

* Pin 1 is the top left pin when orienting the PCB with the ethernet port at the bottom left.
* Pin 2 is the top right pin.
* The TxD and RxD pins are connected to UART0 (ttymxc0), which is used for console output by U-Boot and Linux.

```plaintext
          1___
RTCK 01 - |.|.| - 02 TRST
 GND 03 - |.|.| - 04 TCK
 GND 05 - |.|.| - 06 TMS
SRST 07 - |.|.| - 08 TDI
  Vt 09 - |.|.| - 10 TDO
 RxD 11 - |.|.| - 12 NC
 TxD 13 - |.|.| - 14 GND
```
## Sensors

### temperature sensor
Analog Devices ADT7410

* I2C temperature sensor
* Accuracy: ±0.5°C from -40°C to +105°C
* 16-bit resolution (~0.0078°C)
* Address: 0x49 @ i2c-0
* Linux driver: `adt7410`

Note: the ATD7410 would normally reside on 0x48, and while there is a driver configured for this address, it does not appear to work.

Potentially, the device was (planned to be) moved to 0x48 on later hardware revisions, or perhaps dual sensors were planned.

The software on Toon v1 also appears to apply some offset to the measured temperature value.

[Datasheet](./datasheets/Analog-Devices-ADT7410.pdf)

### (defunct?) temperature sensor
Unknown device

* I2C temperature sensor
* Address: 0x4c @ i2c-0
* Linux driver: `tmp431`

Does not appear to work, and is likely a leftover from an earlier hardware revision.
The driver is configured for this address, but the device does not respond to any commands.

## Other ICs of interest

### Real-time clock
Renesas ISL1208

* I2C real-time clock
* Address: 0x6f @ i2c-1
* Linux driver: `isl1208`

[Datasheet](./datasheets/Renesas-ISL1208.pdf)

### Touch controller
Texas Instruments TSC2007

* I2C interface
* 4-wire resistive touch
* Address: 0x48 @ i2c-1
* Linux driver: `tsc2007`

[Datasheet](./datasheets/TI-TSC2007.pdf)

### Audio codec + DSP + amplifier
Texas Instruments TLV320AIC3111

* Stereo audio codec
* I2C interface
* Address: 0x18 @ i2c-0
* Linux driver: `tlv320aic3111`

[Datasheet](./datasheets/TI-TLV320AIC3111.pdf)


# Mechanical aspects

Toon v1 has 5 main components:

1. The PCB, which contains the SoC, memory, storage, etc.
2. The display, which is a 7-inch touchscreen.
3. The front bezel, which holds the display in place.
4. The enclosure, which houses the PCB and display.
5. The wall mount, which provides power and connectivity to the Toon.

None of the components are screwed or glued; instead they are all clipped together.

## Enclosure
The enclosure is made of plastic and has a matte finish.
It has cutouts for the display, ethernet port, USB port, reset button and power / OpenTherm port, as well as small slots for ventilation in the top back.

The front bezel can easily be removed by gently prying it off the main body.
There are two clips per side.
Unfortunately, these clips are somewhat fragile, and not designed to be removed and reattached multiple times.

## Display
The display is connected to the PCB via a 40-pin FPC connector as well as a small connector for the touchscreen.

These connectors can be accessed by carefully tilting the display downward.

Both connectors can be unplugged by gently prying open the connector's locking mechanism.

## PCB
The PCB is clipped into the enclosure with 6 clips; one in each corner and two surrounding the OpenTherm port.
These clips are quite fragile, and break off easily.
Do not use anything thicker than a piece of paper to pry the clips off, as bending them too far will break the clips.

The PCB is a multi-layer board that contains all the electronic components of the Toon.
It was designed to be compact and efficient, with a focus on low power consumption.

The PCB has several connectors for the various interfaces, including Ethernet, USB, and OpenTherm.
It also has a JTAG header for debugging and programming.

The single button on the PCB is used for (hard) rebooting the Toon.
