---
title: "ARM: rpi"
category: Deploy
order: 3
---
- TOC
{:toc}

# Deploy trust\|me on Raspberry Pi platforms
{:.no_toc}

This section describes how to deploy trust\|me on Raspberry Pi platforms.

> **Current pre-built release image**: \\
[trustmeimage-{{site.release_tag}}_arm32_raspberrypi2.img.bz2]({{site.githuborg}}/{{site.repository}}/releases/download/{{site.release_tag}}/trustmeimage-{{site.release_tag}}_arm32_raspberrypi2.img.bz2)\\
[trustmeimage-{{site.release_tag}}_arm64_raspberrypi3-64.img.bz2]({{site.githuborg}}/{{site.repository}}/releases/download/{{site.release_tag}}/trustmeimage-{{site.release_tag}}_arm64_raspberrypi3-64.img.bz2)

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
- **Raspberry Pi2**
```
cd ws-yocto # your yocto workspace directory
sudo copy_image_to_disk_mbr.sh \
	out-yocto/tmp/deploy/images/raspberrypi2/trustme_image/trustmeimage.img \
	/dev/mmcblk0
```
- **Raspberry Pi3**
```
cd ws-yocto # your yocto workspace directory
sudo copy_image_to_disk_mbr.sh \
	out-yocto/tmp/deploy/images/raspberrypi3-64/trustme_image/trustmeimage.img \
	/dev/mmcblk0
```

## Boot trust|me

Connect a monitor to the HDMI port and a keyboard to the USB connector of your Raspberry Pi
 board.

After boot a shell in the management container (c0) will be available at tty1.
Also a debug shell into the CML will be available at tty12.
Further, the init log messages will appear on tty11.

> **Note**: On first boot several keys are generated, thus it may take a long
time untill login prompt may appear. You can accelerate the progress by generating
randomness with the connected keyboard.

For instructions on how to operate trust\|me please refer to section [Operate](/operate).
