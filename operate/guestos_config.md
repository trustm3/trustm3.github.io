---
---

- TOC
{:toc}

## GuestOS configuration

An operating system configuration is defined in the `<os-name>-<os-version>.conf` in the `/data/cml/operatingsystems/` folder. The configuration includes OS name and version, supported hardware type, description of mounted images, init process path and parameters, build date, and others. Descriptions of mounted images include their hashes, size, and file system type.

The full GuestOS configuration contains following fields:

```protobuf
// The following three fields together should UNIQUELY identify the actual guestos image files and config:
required string name  // unique name of the operating system
required string hardware // target hardware; must match hardware_get_name(), e.g. "i9505", etc.
required uint64 version // version string for the guest OS

repeated GuestOSMount mounts // list of mounts inside the container
optional string init_path [default = "/init"];  // path to the init binary, e.g. "/init"
repeated string init_param // parameters (argv) for init
repeated string init_env  // environment variables
repeated GuestOSMount mounts_setup // list of mounts for setup mode of a container

optional bool feature_install_guest [default = false]

optional uint32 min_ram_limit [default = 128] // required minimum RAM size (MBytes)
optional uint32 def_ram_limit [default = 1024]  // default RAM size (MBytes)

// Descriptive information (for GUI):
required I18NString description // description/full name
optional I18NString description_long // help text
optional string upstream_version // upstream version
optional bytes icon // name of icon file
optional string build_date // build date

optional string	update_base_url // provide url to file server which hosts the actual image data (overwrites device.conf)


```

The original Protobuf formatted file can be found [here](https://github.com/trustm3/device_fraunhofer_common_cml/blob/trustx-master/daemon/guestos.proto).

Similar to container configuration also direct access to guestOS specific configuration files is not possible from core0 container.
Intended control is granted through the control interface.

The corresponding commands of the control tool are `ca_register`,
`list_guestos`, `push_guestos_config` and `remove_guestos`:

**ca_register** is used to install a new root certificate inside the CML.
Thus, it is possible to provide customer specific GuestOSes, signed by
a certificate from that CA.

**list_guestos** provides the configuration in protobuf-text format of
the already installed GuestOSes which have a valid signature.

**push_guestos_config**  is used to install new GuestOSes as well as
to update existing ones. The `cmld` checks if the config contains a valid
signature and matches the included image hashes, before accepting the new/updated GuestOS.

**remove_guestos**  is used to remove GuestOSes.

## Example: Using GuestOS debos
The GuestOS debos provides a basic integrity protected debian installer
base image. This GuestOS type provides the ability to setup a
confidential debian system inside a container.
In order to achieve that, we use the setup mode for running a container instance of debos.

### Prerequisites
* a running (deployed) trust\|me system.
* an installed debos image in that system   
  You can verify this by checking:
 ```
root@trustx-core:~# control list_guestos | grep debos
     name: "debos"
  ```
* configured internet access in core0 (default is dhcp on eth0)
* we assume a standard password for usertoken "trustme"


### Creating container and installing debian
create a new container config skeleton file `debian1.skel` for the container e.g.:
```
name: "debian1"
guest_os: "debos"
color: 0
vnet_configs {
    if_name: "eth0"
    configure: true
}
image_sizes {
    image_name: "enc_root"
    image_size: 8192
}
```
The images_size argument overwrites the standard size of the encrypted writable
data image which is mounted as rootfs in case of normal boot.

Create and start the container in setup mode:
```
control create debian1.skel
control start debian1 --key=trustme --setup
```
You can verify that the container has reached setup state by:
```
control state debian1
```
Now you can run commands inside the container to install
a standard Linux system to `/setup` inside the container.
The debos image contains a debootstrap util inside the container to
setup a debian system. However, you could also try to run any other
bootstrapping tool for another distribution, e.g., emerge. In that case,
you have to download the bootstrapper first, loosing integrity protection
of the installer. We proceed with `debootstrap`.
> Remember that control run is somewhat experimental and not all characters,
especially control characters like [Ctrl]+C, may work as expected.

```
control run /bin/bash debian1
``` 
Now we are inside the container "debian1".   
You can check if the encrypted root is mounted on setup by calling `mount`

Install debian as follows:
```
export PATH=$PATH
debootstrap stretch /setup
printf '%s\n' '#!/bin/bash' '/sbin/cservice' 'exit 0' > /setup/etc/rc.local
chmod a+x /setup/etc/rc.local
exit
```
Now stop the container and start it normally:
```
control stop debian1
control start debian1 --key=trustme
```

## Example: Using docker-convertos
> **Experimental**, the converter may not work for every image on dockerhub!

### Prerequisites
* a running (deployed) trust\|me system.
* an installed docker-convertos image in that system   
  You can verify this by checking:
  ```
root@trustx-cml:~# control list_guestos | grep docker-convertos
     name: "docker-convertos"
  ```
* configured internet access in core0 (default is dhcp on eth0)
* we assume a standard password for usertoken "trustme"

### Creating converter container
Create a new container config skeleton file `docker2.skel` for the container e.g.:
```
name: "docker2"
guest_os: "docker-convertos"
color: 0
vnet_configs {
  if_name: "eth0"
  configure: true
}
image_sizes {
  image_name: "tmp"
  image_size: 8192
}
```

```
control run /bin/bash docker2
```
Inside container:
```
export PATH=$PATH
/etc/init.d/lighttpd start
converter pull <dockerhub image> <tag>
/etc/init.d/lighttpd stop
exit
```

### Creating instance of converted image
```
control list_guestos
```
Should show new GuestOS created from Dockerhub image
with a name like "`<image>_<tag>_<timestamp>`"

Create a new config skeleton `/etc/templates/converted.skel` for the image:
```
name: "converted3"
guest_os: "<image>_<tag>_<timestamp>"
color: 0
vnet_configs {
  if_name: "eth0"
  configure: true
}
```

```
control create converted.skel
```
Run it.
```
control start converted3 --key=trustme
```

