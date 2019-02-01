---
---
# Run trust\|me image in QEMU/KVM (x86-64)
Before following these steps you need to create the partitioned trust\|me image as described [here]({{ "/" | absolute_url }}build)  
In first place, the QEMU/KVM and the OVMF UEFI have to be installed.
```
   apt-get install qemu-kvm ovmf
```
Also, an image holding the created containers is needed. Choose the image size large enough to hold the containers you will create.
```
   dd if=/dev/zero of=containers.btrfs bs=1M count=<space to be available for containers>
   mkfs.btrfs -L containers containers.btrfs
```

   Now the trustme image can be booted as follows:   
```
   kvm -m 4096 -bios OVMF.fd -serial mon:stdio \
	-device virtio-scsi-pci,id=scsi \
	-device scsi-hd,drive=hd0 -drive if=none,id=hd0,file=$(ls trustmeimage-* | tail -n1),format=raw \
	-device scsi-hd,drive=hd1 -drive if=none,id=hd1,file=containers.btrfs,format=raw
```

# Run trust\|me in QEMU/KVM with TPM emulation

This section describes the process of running trust\|me in KVM/QEMU connected to an emulated TPM2.0 which runs inside a Docker container.
( [swtpm] (https://github.com/stefanberger/swtpm) ).

## Build SW-TPM Docker


```
cd ws-yocto/trustme/cml/tpm2d/swtpm-docker
docker build -t swtpm-docker .
```

## Run TPM 2.0 emulator in a Docker container

```
./run-swtpm-docker.sh
```

This creates a UNIX socket in the host's directory '/tmp/swtpmqemu/', which QEMU can connect to.

## Run KVM/QEMU

The following example runs a trust\|me image using the TPM emulator and enabling host-guest port forwarding in KVM:

```
sudo kvm -m 4096 -bios OVMF.fd -serial mon:stdio \
	-device virtio-scsi-pci,id=scsi \
	-device scsi-hd,drive=hd0 \
	-drive if=none,id=hd0,file=$(ls trustmeimage-* | tail -n1),format=raw \
	-device scsi-hd,drive=hd1 -drive if=none,id=hd1,file=containers.btrfs,format=raw \
	-net nic -net user,hostfwd=tcp::8181-:8181,hostfwd=tcp::2323-:22 \
	-chardev socket,id=chrtpm,path=/tmp/swtpmqemu/swtpm-sock \
	-tpmdev emulator,id=tpm0,chardev=chrtpm \
	-device tpm-tis,tpmdev=tpm0
```

