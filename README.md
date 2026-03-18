# NavQ95

The [IMX yocto project users guide](https://www.nxp.com/docs/en/user-guide/UG10164.pdf) contains
detailed explanation on how to an build SD card image for the various iMX devices. For the NavQ95 there are a few exception that need to be taken
care of.

See below table containing items that deviate from the manual:

| Item                   | Original                                    | New                                               |
| -----------------------| ------------------------------------------- | ------------------------------------------------- |
| imx-manifest repo URL  | https://github.com/nxp-imx/imx-manifest.git | https://github.com/NXP-Robotics/imx-manifest-navq95-private.git |
| Manifest file          | imx-6.12.20-2.0.0.xml                       | imx-6.12.20-2.0.0-navq.xml                        |
| Machine                | * (eg. imx95-19x19-lpddr5-evk)              | imx95-navqbdesktop                                |

For a NavQ95 specific explanation refer to the [Build SD card image](#build-sd-card-image) and the [Flash image to SD card](#flash-image-to-sd-card) paragraphs on this page.

<a name="build-sd-card-image"></a>

## Build SD card image

Sync repositories by manifest:
```bash
mkdir imx-yocto-bsp
cd imx-yocto-bsp
repo init -u git@github.com:NXP-Robotics/imx-manifest-navq95-private.git -b imx-linux-walnascar -m imx-6.12.20-2.0.0-navq.xml
repo sync
```

Setup build:
```bash
MACHINE=imx95-navqbdesktop DISTRO=imx-desktop-xwayland source imx-setup-release.sh -b build-95-full
```

Optionally add below lines to conf/local.conf in case the host should stay responsive
```bash
BB_NUMBER_THREADS = "6"
PARALLEL_MAKE = "-j 5"
```

Start build:
```bash
bitbake mc:imx95-navqdesktop:imx-image-mr
```
Or to start build and immediately detach the process from the console (may be convenient since this build may take a while)
```bash
nohup bitbake mc:imx95-navqdesktop:imx-image-mr &
```

To build an image with ROS2 preinstalled:
```bash
bitbake mc:imx95-navqdesktop:imx-image-ros
```

<a name="flash-image-to-sd-card"></a>

## Flash image to SD card

To flash the yocto image to an SD card use the command below. Make sure you update the output file ```of=/dev/sdX``` to the block device that belong to the SD card.
```bash
cd /path/to/imx-yocto-bsp/build-95-full

# Deploy mc:imx95-navqdesktop:imx-image-mr on /dev/sdX
zstdcat tmp-imx95-navq/deploy/images/imx95-navq/imx-image-mr-imx95-navq.rootfs.wic.zst | sudo dd of=/dev/sdX bs=1M conv=fsync

# Deploy mc:imx95-navqdesktop:imx-image-ros on /dev/sdX
zstdcat tmp-imx95-navq/deploy/images/imx95-navq/imx-image-ros-imx95-navq.rootfs.wic.zst | sudo dd of=/dev/sdX bs=1M conv=fsync
```

<a name="flash-nor-flash-image"></a>

## Flash NOR flash image

Install pyocd to flash the image through the on-board JTAG device.
This is tested with python 3.10 but python 3.9 should suffice.

Clone pyocd into a directory of your preference and checkout the `pr-imx95` branch:
```bash
git clone https://github.com/NXP-Robotics/pyOCD -b pr-imx95
```

Build pyocd:
```bash
cd pyocd-private
python3 -m pip install .
```

Make sure the DIP switches are correctly configured as described in [Power up](#power-up-navq95).
Remove any SD card and connect the Debug USB port (J2) to your host. Then apply 12V to the J15 connector to power up the board. Make sure to do this in the given order.

Run below command to flash the built RTOS software to the NOR flash:
```bash

pyocd flash -t mimx95_cm33_mx25um path/to/built/file.bin -f 10m
```

> [!WARNING]
> Writing to NOR flash is not completely stable yet. Retry the pyocd flash command until pyocd displays it had only programmed 0 pages.

<a name="power-up-navq95"></a>

# Power up NavQ95

Before powerering up the NavQ95 make sure the DIP switches have the correct settings. They must be configured like the image below to boot from the SD card.

<img src="dip-switches.png" alt="navq95 ports" style="width:20%;"/>

To change the boot device see table below for different boot modes

| **BOOT_MODE[3:0]** | **SW1** | **SW2** | **SW3** | **SW4** |          **Boot Device**          |
|:------------------:|---------|---------|---------|---------|:---------------------------------:|
| `1001`             | ON      | OFF     | OFF     | ON      | Serial Downloader on USB3.0 (J13) |
| `1010`             | OFF     | ON      | OFF     | ON      | Boot from eMMC                    |
| `1011`             | ON      | ON      | OFF     | ON      | Boot from SD Card                 |
| `1100`             | OFF     | OFF     | ON      | ON      | Boot from Octal Flash             |



Insert the SD card with the image installed. Then apply 9-52V to the J19 connector to power up the board.

<img src="mr_navq95-ports.png" alt="navq95 ports" style="width:50%;"/>

The USB port gives access to the tty's of linux and RTOS (if flashed to the NOR flash).

The default linux user is 'user' (password: 'user')

# Optional flashing the eMMC

Set the boot switches into "Serial Downloader on USB3.0 (J13)" and insert a USB-C cable from your PC to J13 and power up the NavQ95.

Then download/install UUU from https://github.com/nxp-imx/mfgtools

> [!NOTE]
> The bootloader for flashing is located on this repo under as `MR-NAVQ95B-SERIAL-DOWNLOAD.bin`

To flash your image/wic use the following command

```
 ./uuu.exe -b emmc_all MR-NAVQ95B-SERIAL-DOWNLOAD.bin <patch.to.wic.file>
```

After flashing unpower the board.

Set the boot switches to "Boot from eMMC" and power up the NavQ95.