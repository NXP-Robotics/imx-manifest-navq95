CTO Mobile Rotobics NavQ+ BSP
======

The [IMX yocto project users guide](https://www.nxp.com/docs/en/user-guide/IMX_YOCTO_PROJECT_USERS_GUIDE.pdf) contains 
detailed explanation on how to an build SD card image for the various iMX devices. For the NavQ+ there are a few exception that need to be taken 
care of.

See below table containing items that deviate from the manual:

| Item                   | Original                                    | New                                               |
| -----------------------| ------------------------------------------- | ------------------------------------------------- |
| imx-manifest repo URL  | https://github.com/nxp-imx/imx-manifest.git | https://github.com/NXPHoverGames/imx-manifest |
| Manifest file          | imx-6.6.52-2.2.0.xml                        | imx-6.6.52-2.2.0-navq.xml                         |
| Machine                | * (eg. imx8mpevk)              | imx8mp-navq                          |

For a NavQ+ specific explanation refer to the [Build SD card image](#build-sd-card-image) and the [Flash image to SD card](#flash-image-to-sd-card) paragraphs on this page.

Both SD card and NOR flash need the correct images for the NavQ+ to work properly.

<a name="build-sd-card-image"></a>

Build SD card image
-------------------

Sync repositories by manifest:
```bash
mkdir imx-yocto-bsp
cd imx-yocto-bsp
repo init -u https://github.com/NXPHoverGames/imx-manifest -b imx-linux-scarthgap -m imx-6.6.52-2.2.0-navq.xml
repo sync
```

Setup build:
```bash
MACHINE=imx8mp-navqdesktop DISTRO=imx-desktop-xwayland source imx-setup-release.sh -b build
```

Optionally add below lines to conf/local.conf in case the host should stay responsive
```bash
BB_NUMBER_THREADS = "6"
PARALLEL_MAKE = "-j 5"
```

Start build:
```bash
bitbake imx-image-mr
```
Or to start build and immediately detach the process from the console (may be convenient since this build may take a while)
```bash
nohup bitbake imx-image-mr &
```

To build an image with ROS2 preinstalled:
```bash
bitbake imx-image-ros
```

Flash image to SD card
----------------------

To flash the yocto image to an SD card use the command below. Make sure you update the output file ```of=/dev/sdX``` to the block device that belong to the SD card.
```bash
cd /path/to/imx-yocto-bsp/build
zstdcat tmp/deploy/images/imx8mpnavq/imx-image-mr-imx8mpnavq.rootfs.wic.zst | sudo dd of=/dev/sdX bs=1M conv=fsync
```