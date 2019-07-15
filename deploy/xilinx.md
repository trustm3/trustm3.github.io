---
---
- TOC
{:toc}

# Deploy trust\|me on Xilinx Zynq platforms
{:.no_toc}

This section describes how to deploy trust\|me on a Xilinx Zynq platform.

## Create bootable medium

### Requirements
* A successfully built trust\|me image file (trustmeimage.img), either downloaded from **tbd** or built following the instructions [here]({{ "/" | abolute_url }}build/build#build-trustme-image).
* The script **copy_image_to_disk.sh** which can be found [on GitHub](https://github.com/trustme_build/yocto/copy_image_to_disk.sh) or in your build folder at `trustme/build/yocto/copy_image_to_disk.sh`
* A MicroSD card compatible with your board
* The pre-built BOOT.BIN from your board's Board Support Package package (can be downloaded from the Xilinx website)

In first place, ensure the needed packages are installed on your system.
```
apt-get install util-linux btrfs-progs sgdisk parted
```

### Copy trust\|me image to disk
Now the trust\|me image can be copied to the MicroSD card.
The provided script takes care of expanding the partitions to usea all of the available disk space.

**WARNING: This operation will wipe all data on the target device**
```
sudo copy_image_to_disk.sh <trustme-image> </path/to/target/device>
```

If you have built from source in `ws-yocto` and your target device is `/dev/mmc0` the command would be:
```
cd ws-yocto # your yocto workspace directory
sudo copy_image_to_disk.sh trustmeimage.img /dev/mmc0
```


### Copy BOOT.BIN to the MicroSD card
The Zynq boards need a BOOT.BIN file to boot. Copy this file from your board's BSP to the partition 1 of the SD card

```
mount </path/to/target/device> <mount point>
# e.g. mount /dev/mmc0p1 <mount point
sudo cp <path/to/BOOT.BIN> <mount point>
sync
umount <mount point>
```

## Boot trust|me

Connect the Xilinx board to your host machine via the onboard serial-to-usb converter using e.g. Minicom. Then boot the board from the MicroSD card.
For instructions on how to do this please refer to the board's manual.
After boot, the trust\|me shell will become available on the serial console.
For instructions on how to operate trust\|me please refer to section [Operate](/operate).
