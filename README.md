NavQ95
======

The [IMX yocto project users guide](https://www.nxp.com/docs/en/user-guide/IMX_YOCTO_PROJECT_USERS_GUIDE.pdf) contains 
detailed explanation on how to an build SD card image for the various iMX devices. For the NavQ95 there are a few exception that need to be taken 
care of.

See below table containing items that deviate from the manual:

| Item                   | Original                                    | New                                               |
| -----------------------| ------------------------------------------- | ------------------------------------------------- |
| imx-manifest repo URL  | https://github.com/nxp-imx/imx-manifest.git | https://github.com/NXPHoverGames/imx-manifest-navq95-private.git |
| Manifest file          | imx-6.6.23-2.0.0.xml                        | imx-6.6.23-2.0.0-navq.xml                         |
| Machine                | * (eg. imx95-19x19-lpddr5-evk)              | imx95-19x19-navqdesktop                           |

For a NavQ95 specific explanation refer to the [Build SD card image](#build-sd-card-image) and the [Flash image to SD card](#flash-image-to-sd-card) paragraphs on this page.

There is no M7 software included in the image build by yocto and instead this is placed into NOR flash of the NavQ95. This should
be [built](#build-nor-flash-image) and [flashed](#flash-nor-flash-image) to the NOR flash manually.

Both SD card and NOR flash need the correct images for the NavQ95 to work properly.

<a name="build-sd-card-image"></a>

Build SD card image
-------------------

Sync repositories by manifest:
```bash
mkdir imx-yocto-bsp
cd imx-yocto-bsp
repo init -u https://github.com/NXPHoverGames/imx-manifest-navq95-private.git -b imx-linux-scarthgap -m imx-6.6.23-2.0.0-navq.xml
repo sync
```

Setup build:
```bash
MACHINE=imx95-19x19-navqdesktop DISTRO=imx-desktop-xwayland source imx-setup-release.sh -b build-95-full
```

Optionally add below lines to conf/local.conf in case the host should stay responsive
```bash
BB_NUMBER_THREADS = "6"
PARALLEL_MAKE = "-j 5"
```

Start build:
```bash
bitbake imx-image-desktop
```
Or to start build and immediately detach the process from the console (may be convenient since this build may take a while)
```bash
nohup bitbake imx-image-desktop &
```

<a name="flash-image-to-sd-card"></a>

Flash image to SD card
----------------------

To flash the yocto image to an SD card use the command below. Make sure you update the output file ```of=/dev/sdX``` to the block device that belong to the SD card.
```bash
cd /path/to/imx-yocto-bsp/build-95-full
zstdcat tmp/deploy/images/imx95-19x19-navq/imx-image-desktop-imx95-19x19-navq.rootfs.wic.zst | sudo dd of=/dev/sdX bs=1M conv=fsync
```

<a name="build-nor-flash-image"></a>

Build NOR flash image
---------------------

The M7 core runs the PX4 autopilot software and this need to be stored on the NOR flash.

To build the PX4 software first clone the PX4 software and checkout the imx95-m7 branch:
```bash
git clone https://github.com/NXPHoverGames/PX4-Autopilot-NXP.git --recursive
cd PX4-Autopilot
git checkout imx95-m7
```

Then build the nxp_imx95_default target:
```bash
make nxp_imx95_default
```

<a name="flash-nor-flash-image"></a>

Flash NOR flash image
---------------------

Install pyocd to flash the image through the on-board JTAG device.

Clone pyocd into a directory of your preference and checkout the imx95 branch:
```bash
git clone https://github.com/NXPHoverGames/pyocd-private.git
cd pyocd
git checkout imx95
```

Build pyocd:
```bash
python3 -m pip install .
```

Power up the NavQ95 without any SD card inserted and with the host connected to the Debug USB port (J2)

Run below command to flash the PX4 software to the NOR flash:
```bash
cd /path/to/PX4-Autopilot
pyocd flash -t mimx95_cm33 ./build/nxp_imx95_default/nxp_imx95_default.bin -f 10m
```

:warning: Writing to NOR flash is not completely stable yet. Retry the pyocd flash command until pyocd displays it had only programmed 0 pages.
