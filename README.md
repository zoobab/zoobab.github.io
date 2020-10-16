[![noswpatv3](http://zoobab.wdfiles.com/local--files/start/noupcv3.jpg)](https://ffii.org/donate-now-to-save-europe-from-software-patents-says-ffii/)
# DirtyJTAG

DirtyJTAG is a JTAG adapter firmware for $2 ST-Link clones and generic STM32 development boards ("blue pill"/"black pill" STM32F101 and STM32F103 based ARM boards). The DirtyJTAG project was created to find an alternative to the obsolete (but cheap) LPT Wiggler cables, and expensive USB JTAG probes.

DirtyJTAG is dirty and dirt cheap, but is not fast nor a perfect implementation of the JTAG protocol. Yet it is around 500 sloccount lines, therefore it is easily understandable and hackable.

DirtyJTAG is supported in mainline UrJTAG, see the [Installing UrJTAG with DirtyJTAG support](#installing-urjtag-with-dirtyjtag-support) section.

If you prefer OpenOCD to UrJTAG, I suggest using Zoobab's fork of Versaloon firmware, which is available [on his GitHub repository](https://github.com/zoobab/versaloon).

## Complete instructions for installing DirtyJTAG on a $2 chinese ST-Link clone

The $2 chinese ST-Link clones that you can find on Aliexpress/eBay are based on a STM32F101 chip. They can be repurposed into a JTAG adapter using the DirtyJTAG firmware. However, due to the limited number of GPIO available to the user, only the bare minimum JTAG pins are available : `TDI/TDO/TMS/TCK` (no `SRST` or `TRST`).

### Taking apart the adapter

Press firmly the USB connector on a flat surface while holding the outer aluminium casing. This will release the PCB from its casing.

![Tearing down an ST-Link programmer](docs/img/stlinkv2-teardown.gif)

### Install DirtyJTAG firmware

In order to install a newer firmware, you will need a SWD programmer. I chose to use another $2 ST-Link programmer. Connect them together like this :

![Two ST-Link clones connected together](docs/img/stlinkv2-programming.png)

Do not plug your ST-Link (target) in a USB port yet. Install [stlink](https://github.com/texane/stlink) on your computer, and download a compiled release of DirtyJTAG for ST-Link adapters ([available here](https://github.com/jeanthom/dirtyjtag/releases)). Enter this command to program the STM32 :

```
st-flash write /path/to/dirtyjtag-stlink.bin 0x8000000
```

If for some reason the flashing process fails because the reported flash size is 0, [there is a fix](https://github.com/texane/stlink/issues/172#issuecomment-347887271).

If you have pogopins around, you could make your own flashing jig by soldering 4 of those on 2 pieces of stripboard, and hold them with a clothespin:

![Flashing jig with 4 pogopins and a clothespin](docs/img/stlinkv2-jig-1.jpg)
![Flashing jig with 4 pogopins and a clothespin bis](docs/img/stlinkv2-jig-2.jpg)

Otherwise some male to female header cable works great, it fits the pad hole snugly and doesn't make a loose connection.

![ST-Link connected together with header cable](docs/img/stlinkv2-header-cable.jpg)

### Using your *brand new* DirtyJTAG dongle with a JTAG target

This is the new pinout for your DirtyJTAG dongle. You may want to print it and glue it to the case for practical reasons.

![ST-Link v2 pinout with DirtyJTAG firmware](docs/img/stlinkv2-pinout.svg)

Install [UrJTAG with DirtyJTAG support](#installing-urjtag-with-dirtyjtag-support). Plug your DirtyJTAG dongle, and check with the `lsusb` command that a new device with `0x1209/0xC0CA` as its USB VID/PID has appeared.

```
$ jtag
UrJTAG 0.10 #
Copyright (C) 2002, 2003 ETC s.r.o.
Copyright (C) 2007, 2008, 2009 Kolja Waschk and the respective authors

UrJTAG is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
There is absolutely no warranty for UrJTAG.

warning: UrJTAG may damage your hardware!
Type "quit" to exit, "help" for help.

jtag> cable dirtyjtag
jtag> detect

```

## Installing DirtyJTAG on a standard STM32 developement board

The procedure is the same as above : download a pre-built version of the
firmware (available [on the release
page](https://github.com/jeanthom/dirtyjtag/releases)) or build the firmware
yourself (instructions provided below).

### Via an ST-Link-v2 dongle

Install [stlink](https://github.com/texane/stlink), then use this command :

```
st-flash write /path/to/dirtyjtag.bin 0x8000000
```

### Via a USB-serial adaptor

You can flash DirtyJTAG with a usb-serial dongle (such as based on chips such
as wch340g, pl2303, cp2102, ft232r, etc...). In the following example, we use a
cheap ch340g usb-serial dongle with can be bought on Aliexpress for 0.67USD.

Before flashing, you have to put the yellow jumpers `BOOT0` to 1, and keep `BOOT1`
at 0, like on the following picture:

![Set the yellow jumpers BOOT0=1 and BOOT1=0](docs/img/bluepill-boot0-boot1-flashmode.png)

If you don't want to solder the yellow 2 rows of headers on the side of the
board, you can prepare 2 bended pins at the end of 2 female-to-female wires,
and connect them to A9 and A10, while pressing the head againt the jumpers
block to maintain stability.

![Flashing DirtyJTAG with USB-serial adaptor](docs/img/bluepill-stm32flash-squaredpins-nosoldering.jpg)

You have to connect:

| USB-SERIAL | BLUEPILL |
|-------|------|
| 5V(or 3.3V)    | SWD_3.3  |
| GND   | SWD_GND  |
| TX   | A10  |
| RX   | A9   |

Note that the flashing works with either 5V or 3V3 as supplied voltage from
the USB-serial adaptor.

Then, you have to install `stm32flash`, which can be obtained in your
distribution (`apt-get install stm32flash` for debian/ubuntu), and flash the
firmware with this command:

```
$ stm32flash -w dirtyjtag.bluepill.bin -v -g 0x8000000 /dev/ttyUSB0
stm32flash 0.5

http://stm32flash.sourceforge.net/

Using Parser : Raw BINARY
Interface serial_posix: 57600 8E1
Version      : 0x22
Option 1     : 0x00
Option 2     : 0x00
Device ID    : 0x0410 (STM32F10xxx Medium-density)
- RAM        : 20KiB  (512b reserved by bootloader)
- Flash      : 128KiB (size first sector: 4x1024)
- Option RAM : 16b
- System RAM : 2KiB
Write to memory
Erasing memory
Wrote and verified address 0x08002028 (100.00%) Done.

Starting execution at address 0x08000000... done.
$
```

Then unplug it, put back the `BOOT0` jumper to 0, and plug it in USB, you should
see the "1209:c0ca InterBiometrics" in the output of `lsusb`:

```
$ lsusb
Bus 001 Device 004: ID 2232:1024 Silicon Motion 
Bus 001 Device 003: ID 8087:07da Intel Corp. 
Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 004: ID 1209:c0ca InterBiometrics 
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

### Pinout

The `bluepill` build of DirtyJTAG has the following pinout :

| STM32 | JTAG |
|-------|------|
| PA0   | TDI  |
| PA1   | TDO  |
| PA2   | TCK  |
| PA3   | TMS  |
| PA4   | TRST |
| PA5   | SRST |

If needed, pin definition can be modified in `src/jtag.c`.

## USB VID and PID

Once the bluepill dongle has been flashed, you should see the following USB VID=1209
and PID=c0ca belonging to "InterBiometrics" once plugged in on a linux computer:

```
$ lsusb
[...]
Bus 002 Device 003: ID 1209:c0ca InterBiometrics
```

The VID/PID was obtained through http://pid.codes, which is a registry of USB
PID codes for open source hardware projects.

More infos: http://pid.codes/1209/C0CA/

## Installing UrJTAG with DirtyJTAG support

Mainline UrJTAG has DirtyJTAG support since [763bbc](https://sourceforge.net/p/urjtag/git/ci/763bbce1213f5759e2925773d0dd5f3b537368f6/tree/) revision. However, most package sources still use 2011 UrJTAG sources, so you'll need to compile UrJTAG :

```bash
cd /tmp
git clone https://git.code.sf.net/p/urjtag/git urjtag-git
cd urjtag-git/urjtag/
./autogen.sh
make
sudo make install
```

Then, you can use DirtyJTAG like any other JTAG cable in UrJTAG :

```text
$ jtag
UrJTAG 0.10 #
Copyright (C) 2002, 2003 ETC s.r.o.
Copyright (C) 2007, 2008, 2009 Kolja Waschk and the respective authors

UrJTAG is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
There is absolutely no warranty for UrJTAG.

warning: UrJTAG may damage your hardware!
Type "quit" to exit, "help" for help.

jtag> cable dirtyjtag
jtag>
```

## DirtyJTAG compilation

### Manual build

In order to compile DirtyJTAG, you will need the following software :

 * git
 * ARM toolchain (I'm using Fedora's `arm-none-eabi-gcc-cs`/`arm-none-eabi-newline` packages)
 * make

Clone this repository :

```
git clone https://github.com/jeanthom/dirtyjtag
cd dirtyjtag
```

Then download unicore-mx :

```
git submodule init
git submodule update
```

Then you can build the firmware :

```
make bin PLATFORM=bluepill
```

Currently there are two platforms :

 * bluepill : Default build setting
 * stlinkv2 : Chinese ST-Link clone with specific pinout

Once the build is completed, your freshly compiled firmware will be available in `src/` as a binary file.

### Docker build

If you have [Docker](https://www.docker.com/) installed on your computer, you can run the `./build.sh` script that will automatically build DirtyJTAG and copy the firmware files out of the container.

## JTAG Validation

You can validate that your DirtyJTAG cable works by using a target, in this example we take another bluepill board which has JTAG pins exposed (yellow pins on the bluepill pinout):

![Bluepill pinout](docs/img/bluepill-pinout.gif)

![Validate that JTAG is working by connecting another bluepill](docs/img/bluepill-flash-another-bluepill-with-jtag.jpg)

You should be able to detect something:

```
jtag> cable DirtyJTAG
jtag> detect
IR length: 9
Chain length: 2
Device Id: 00111011101000000000010001110111 (0x3BA00477)
  Unknown manufacturer! (01000111011) (/usr/share/urjtag/MANUFACTURERS)
Device Id: 00010110010000010000000001000001 (0x16410041)
  Unknown manufacturer! (00000100000) (/usr/share/urjtag/MANUFACTURERS)
```

The MANUFACTURERS file or urjtag need to be updated with "ARM Ltd" and "SGS/Thomson":

https://raw.githubusercontent.com/pmaupin/playtag/master/playtag/bsdl/data/manufacturers.txt

```
$ grep 01000111011 manufacturers.txt
01000111011   ARM Ltd.

$ grep 00000100000 manufacturers.txt
00000100000   SGS/Thomson
```

## JTAG validation on a Linksys WRT54G router

You can test the cable with a Linksys WRT54G router, which has a JTAG header where you need to solder the pins. The pinout is the following for a WRT54Gv22:

![Linksys WRT54Gv22 JTAG pinout](docs/img/wrt54g-jtag-pinout.png)

Here is a picture of the setup:

![DirtyJTAG connected to a WRT54G router JTAG header](docs/img/bluepill-wrt54g-jtag.jpg)

You should then be able to detect the chip:

```
jtag> cable DirtyJTAG
jtag> detect
IR length: 8
Chain length: 1
Device Id: 00100100011100010010000101111111 (0x2471217F)
  Manufacturer: Broadcom (0x17F)
  Part(0):      BCM4712 (0x4712)
  Unknown stepping! (0010) (/usr/share/urjtag/broadcom/bcm4712/STEPPINGS)
jtag>
```

Further documentation is needed on how to dump the flash. Some interesting webpage about a similar router:

https://irgendwasistjaimmer.in-kiel.de/index.php?/archives/23-How-to-flash-a-new-CFE-into-your-bricked-Netgear-WGT634U.html

Which made urjtag segfaults:

```
jtag> discovery
[...]
Detecting DR length for IR 11111011 ... 1
Detecting DR length for IR 11111100 ... 1
Detecting DR length for IR 11111101 ... 1
Detecting DR length for IR 11111110 ... 1
jtag> initbus ejtag_dma
Initialized bus 5, active bus 0
jtag> detectflash 0x1fc00000
Segmentation fault (core dumped)
```

## Inspiration

 * [opendous-jtag](https://github.com/vfonov/opendous-jtag)
 * [neroJtag](https://github.com/makestuff/neroJtag)
 * [clujtag-avr](https://github.com/ClusterM/clujtag-avr)
 * [CuVoodoo page on JTAG and ST-Link v2 adapters](https://wiki.cuvoodoo.info/doku.php?id=jtag)
 * [versaloon jtag](https://github.com/zoobab/versaloon)
 * [blackmagic probe on st-link-v2 clones](https://madnessinthedarkness.transsys.com/blog:2017:0122_black_magic_probe_bmp_on_st-link_v2_clones)
