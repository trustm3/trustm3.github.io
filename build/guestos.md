---
---

# Create new Guest Operating Systems

This page describes the process of creating a new guest OS to be used in trust\|me containers.
It is possible to build a new guest OS using Yocto or by hand using a rootfs image. Both possibilities are described here.

## Using Yocto
In order to create a new guest OS using Yocto, a new bitbake file has to be created. As a starting point the bitbake file for the 'trustx-coreos' guest OS can be used. This file is located at *\<yocto workspace directory\>/meta-trustx/images/trustx-core.bb*. In order to add a new guest OS copy this file into the same folder and rename it.
```
cp <yocto workspace directory>/meta-trustx/images/trustx-core.bb <yocto workspace directory>/meta-trustx/images/trustx-custom.bb
```
> Note: Although this approach is sufficient for testing, the Yocto way of adding your own .bb files would be to create and apply a new Yocto layer as described [here](https://www.yoctoproject.org/docs/current/dev-manual/dev-manual.html#understanding-and-creating-layers)

Before building the new guest OS, a config file has to be created.
Again you may use the trustx-coreos config file as a starting point.
```
cp <yocto workspace directory>/trustme/build/config_overlay/x86/trustx-coreos.conf <yocto workspace directory>/trustme/build/config_overlay/x86/trustx-customos.conf
```
The new config file has to be adapted to suit your needs. At least the 'name' field has to be changed to your OS name, e.g. 'custom'.

At this point the new guest OS is ready for build and you can customize by adding and removing features, tailor the build process to your needs, etc. For information on how to do this, please refer to the [Yocto manual](https://www.yoctoproject.org/docs/current/mega-manual/mega-manual.html).  
After editing the guest OS' bitbake file, rebuild it by running

```
bitbake multiconfig:container:trustx-custom
```
In order to include the guest OS to your trust|\me distro, also rebuild the initramfs:
```
bitbake trustx-cml-initramfs
```
Now you can use the new guest OS when creating containers as described [here]({{ "/" | absolute_url }}/operate).

## Manually / Using a pre-built docker image
Prerequisites:
* The software signing certificate and key. If you use the auto-generated PKI the files needed are 'ssig.key' and 'ssig.cert'
* The signing scripts located at <yocto workspace directory>/trustme/build/device_provisioning/oss_enrollment/config_creator/
* A root file system image, in the following called root.img (e.g. created using docker build + docker export)
* An example guest OS config, for example the config located at "<yocto workspace directory>/trustme/build/config_overlay/x86/trustx-coreos.conf"

Steps to manually create a guest OS

## Create rootfs using docker
1. Adapt the example guest OS config
	* Change the 'name' field to a name of your choice
	* (Optional) Adapt the 'mounts' entries to your needs.
	* Set the 'image_size'. 'image_sha1' and 'image_sha2_256' values to match your root.img
2. Sign the adapted guest OS config using the sign_config.sh scrip as follows:
```
./sign_config.sh <guest OS config> ssig.key ssig.cert
```
3. Copy the created files to to /data/cml/operatingsystems, e.g. the files trustx-customos-1.conf, trustx-customos-1.cert and trustx-customos-1.sig
4. Create a directory for your guest OS root.img, e.g. /data/cml/operatingsystems/trustx-customos-1
5. Copy your root.img to the newly created directory
6. Create a new container using your guest OS as described [here]({{ "/" | absolute_url }}operate)

## Using Docker converter from local registry
