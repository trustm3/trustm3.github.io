---
---

# Build trust|me for x86 systems

The steps to build trust\|me are the same for each flavour of the platform such as core, IDS, etc. They only differ in the available guest operating systems and the installed containers.   
To build the trust\|me flavour you're interested in, select the appropriate containers in [the corresponding step](#include-containers-to-trustme-image). (If you just want to try out trust\|me the core flavour is the best option for you.)

## Prerequisites

To make the build process work flawlessly, please assure your build host meets the following prerequisites:
   * Build host configuration as described in section [Setup Host]({{ "/" | absolute_url }}setup_host)
   * Enough hard disk space, at least TODO GB
   * Enough RAM. We tested the build on a VM having 4 GB RAM. However a build host with less RAM should also work.

## Create a workspace directory
```
   mkdir ws-yocto
   cd ws-yocto
```

## Fetch yocto meta repos and trustme/build repo
```
   repo init -u https://github.com/trustm3/trustme_main.git -b master -m ids-x86-yocto.xml
   repo sync -j8
```

## Setup yocto environment (poky)
```
   export DEVICE=x86 # (is set by default)
   source init_ws.sh out-yocto
```
   This automatically switches to out-yocto and extends ws-yocto/out-yocto/conf/local.conf to configure merged kernel+initramfs binary

## Include containers to trust\|me image
These commands install guest operating systems and containers (e.g. the IDS container) to your trust\|me platform.
It is recommended to use exactly one of the following commands.
> Experienced users may choose to include all containers. However this will require manual configuration of the platform.

Build and include the minimal core container
```
bitbake multiconfig:container:trustx-core
```

Build and include the IDS Trusted Connector container
```
bitbake trustx-cml-initramfs multiconfig:container:ids
```

## Build own poky-tiny distro
This step builds a basic Linux distro based on Yocto's Poky distro. This distro contains the trust|me container management layer.
If you want to make changes to the distro or use your own PKI you can do so by editing the following files:
   * distro config: ws-yocto/meta-trustx/conf/distro/cml-tiny.conf
   * initramfs bitbake file: ws-yocto/meta-trustx/image/trustx-cml-initramfs.bb
   * PKI symlink/directory: ws-yocto/out-yocto/test_pki

```
   bitbake trustx-cml-initramfs
```

## Build trustme image
Finally, a pre-partitioned disk image is created using wic. The resulting image file is called *trustmeimage-\<timestamp\>-sda.direct*.
This image can be deployed as described on the [Deploy](/deploy/x86) pages.

```
wic create -e trustx-cml-initramfs --no-fstab-update trustmeimage
```
  
# Legacy Android Build

Currently there is a Android 7.1.2 based branch and the old beta Android 5.1.1 based branch
available of trust|me. You can find a howto for getting and compiling the code of trust|me for Android in the
main build repository:

1. beta [trustme_build (trustme-7.1.2_r33-github)](https://github.com/trustm3/trustme_build/tree/trustme-7.1.2_r33-github)
2. old-beta [trustme_build (trustme-5.1.1_r38-github)](https://github.com/trustm3/trustme_build/tree/trustme-5.1.1_r38-github)




# Build tips
## Change kernel config
Temporarily
```
     bitbake -f -c menuconfig virtual/kernel
     bitbake -f virtual/kernel
     bitbake -f trustx-cml-initramfs
```

Persistently
* Add file to ws-yocto/meta-trustx/recipes-kernel/linux/files
* Register new file in .bbappend files inside ws-yocto/meta-trustx/recipes-kernel/linux/

## Build test PKI manually
```
   remove test PKI symlink/directory, which is located at ws-yocto/out-yocto/test_certificates
   bash ws-yocto/trustme/build/device_provisioning/gen_dev_certs.sh
```

## Signing kernel+initramfs binary manually
```
   sbsign --key test_certificates/ssig_subca.key \
      --cert test_certificates/ssig_subca.cert \
      --output linux.sigend.efi \
      ws-yocto/out-yocto/tmp/deploy/images/intel-corei7-64/bzImage-initramfs-intel-corei7-64.bin
```

   To boot the signed kernel using UEFI, it should be placed as /EFI/BOOT/BOOTX64.EFI on the UEFI system partition.
