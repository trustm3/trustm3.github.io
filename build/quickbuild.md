---
---
* TOC
{:toc}

# Quick build instructions

This section sums up the necessary steps to build trust\|me on different architectures.
For a detailed explanation of each build step please refer to the [build section](/build/build)
and assure [prerequisites](/build/build#prerequisites).


## x86 platforms
```
mkdir ws-yocto
cd ws-yocto
repo init -u https://github.com/trustm3/trustme_main.git -b master \
    -m yocto-x86-trustx-corei7-64.xml
repo sync -j8
source init_ws.sh out-yocto x86 trustx-corei7-64
bitbake multiconfig:container:trustx-core
bitbake trustx-cml
bitbake trustx-keytool
```

To create a bootable installation medium to install trustme to an internal disk,
also execute the following command.
```
bitbake multiconfig:installer:trustx-installer
```

## Xilinx Zynq ZCU104 platform

```
mkdir ws-yocto
cd ws-yocto
repo init -u https://github.com/trustm3/trustme_main.git -b master \
     -m yocto-arm64-zcu104-zynqmp.xml
repo sync -j8
source init_ws.sh out-yocto arm64 zcu104-zynqmp
bitbake multiconfig:pmu:pmu-firmware
bitbake multiconfig:container:trustx-core
bitbake trustx-cml
```
