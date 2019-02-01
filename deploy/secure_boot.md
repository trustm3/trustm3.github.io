---
---
This page describes how to setup the Secure Boot keys on your system

## Setup Secure Boot on x86 systems
1. Create bootable device for replacing EFI keys:
```
   cd <trustme build directory>
   source init_ws.sh out-yocto
   bitbake trustx-keytool
   wic create -e trustx-keytool keytoolimage
```
> Note: The created image file is named keytoolimage-<timestamp>-sda.direct

2. Copy keytool image to USB drive
**WARNING: This will wipe all data on the target device**
```
   dd if=<keytoolimage.img> of=</path/to/target/device> status=progress
```

3. (Optional) Boot the just created USB drive and backup current platform keys (usually the Microsoft's Platform Keys ).
If you have Secure Boot already enabled you will have to disable it temporarily.   
After the KeyTool has booted, save your Secure Boot keys using KeyTool -> Save Keys

4. Put your UEFI into Setup Mode (BIOS -> Secure Boot -> Erase platform key / Setup mode)
**WARNING: This will erase your Secure Boot platform keys**

5. Boot in Setup Mode 
```
   KeyTool -> Edit Keys
   Replace db with keys/DB.esl
   Replace KEK with keys/KEK.esl
   Replace PK with keys/PK.auth
```

   The PK replacement will switch to Secure Boot User Mode

   see [https://www.rodsbooks.com/efi-bootloaders/controlling-sb.html](https://www.rodsbooks.com/efi-bootloaders/controlling-sb.html) for more information
