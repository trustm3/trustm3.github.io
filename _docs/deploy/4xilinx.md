---
title: "ARM: Xilinx"
category: Deploy
---
- TOC
{:toc}

# Deploy trust\|me on Xilinx Zynq platforms
{:.no_toc}

This section describes how to deploy trust\|me on a Xilinx Zynq platform.

> **Current pre-built release image**: \\
[trustmeimage-{{site.release_tag}}_arm64_zcu104-zynqmp.img.bz2]({{site.githuborg}}/{{site.repository}}/releases/download/{{site.release_tag}}/trustmeimage-{{site.release_tag}}_arm64_zcu104-zynqmp.img.bz2
)

## Create bootable medium



### Requirements
* A successfully built trust\|me image file (trustmeimage.img), either downloaded from [Github Release]({{site.githuborg}}/{{site.repository}}/releases/tag/{{site.release_tag}}) or built following the instructions [here]({{ "/" | abolute_url }}build/build#build-trustme-image).
* The script **copy_image_to_disk_mbr.sh** which can be found [on GitHub](https://github.com/trustm3/trustme_build/raw/master/yocto/copy_image_to_disk_mbr.sh) or in your build folder at `trustme/build/yocto/copy_image_to_disk.sh`
* A MicroSD card compatible with your board

First, ensure the needed packages are installed on your system.
```
apt-get install util-linux btrfs-progs sgdisk parted
```

### Copy trust\|me image to disk
Now the trust\|me image can be copied to the MicroSD card.
The provided script takes care of expanding the partitions to use all of the available disk space.

**WARNING: This operation will wipe all data on the target device**
```
sudo copy_image_to_disk_mbr.sh <trustme-image> </path/to/target/device>
```

If you have built from source in `ws-yocto` and your target device is `/dev/mmcblk0` the command would be:
```
cd ws-yocto # your yocto workspace directory
sudo copy_image_to_disk_mbr.sh out-yocto/tmp/deploy/images/zcu104-zynqmp/trustme_image/trustmeimage.img /dev/mmcblk0
```

<!--
### Copy BOOT.BIN to the MicroSD card
The Zynq boards need a BOOT.BIN file to boot. Copy this file from your board's BSP to the partition 1 of the SD card

```
mount </path/to/target/device> <mount point>
# e.g. mount /dev/mmc0p1 <mount point
sudo cp <path/to/BOOT.BIN> <mount point>
sync
umount <mount point>
```
-->

## Boot trust|me

Connect a monitor to the display port and a keyboard to the USB connector of your
Xilinx board. For an early boot debug shell, also connect the Xilinx board to your host machine via the onboard serial-to-usb converter using e.g. Minicom. Then boot the board from the MicroSD card.
For instructions on how to do this, please refer to the board's manual.

After boot a shell in the management container (c0) will be available at tty1.
Also a debug shell into the CML will be available at tty12.
Further, the init log messages will appear on tty11.

If you have setup the serial connector, the early boot messages of arm-trusted-firmware, OpTEE,
u-boot and the Linux kernel will be printed on serial console. After boot also a CML shell will
become available on the serial console.

For instructions on how to operate trust\|me please refer to section [Operate](/operate).
