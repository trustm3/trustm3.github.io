---
---

- TOC
{:toc}

# Containers and GuestOSes
{:.no_toc}

## Basic trust|me operation

trust|me is operated using a socket-based interface. This interface is used by the *control*
command-line tool, which is installed together with trust|me. It is available from the
privileged container (core0) and the debugging shell of CML.
This tool is just for basic usage and demonstration of the control interface.
For productive use cases an implementation of the protobuf-based control interface should be
used, for instance in a web-based UI.

The *control* tool allows administration and configuration of the trust|me platform, such as creating and starting containers, running a given command inside a container, etc.  
The available actions are listed below.

Usually, container specific commands use the container-uuid as parameter to identify the
corresponding container. However, for convenience also the container name could be used.
However, if you have several containers with the same name, always the first matching
one is used. In that case, you have to specify the UUID to address all containers.

* Start container:
```
control start <container-uuid> --key=<token-key> [ --setup ]
```
This command starts the container with the given UUID. In order to do so, the container has to be created beforehand by 'control create'.
The optional `--setup` parameter allows you to manipulate 
the containers root file system tree including all encrypted overlays mounted at `/setup`.
This could be used to bootstrap an encrypted container, see [example: Using GuestOs debos](#example-using-guestos-debos).
`--key` specifies the Passphrase/PIN for the softtoken used for container en/decryption,
use e.g. for default self-provisioned device `--key=trustme`.

* Stop container:
```
control stop <container-uuid>
```
This command stops the currently running container with the given UUID

* Create container configuration from the given template config file:
```
control create <config-template-file>
```
This generates a new container configuration with a random UUID.
The configuration template file has to be given using protobuf-text syntax. It must contain at least the following:
```
name: "<container name>"
guest_os: "<guest os name>"
color: <color for UI as argb e.g. 0xff00ffff>
```
You can use `control list_guestos` to get the name for available GuestOSes.

* Remove container:
```
control remove <container-uuid>
```
Deletes the given container completely including configuration, data images and corresponding wrapped keys.

* Container current state:
```
control state <container-uuid>
```
This returns the current state of the given container. Possible states are:
	- `STOPPED`: The container has either not been run, was explicitly stopped or the init process of the container exited
	- `STARTING`: The container was started using 'control start' and the container manager is preparing to launch the container.
	- `BOOTING`: The container manager finished preparing the container and the control was handed over to the container's init process
	- `RUNNING`: The container was initialized successfully and is now running.
	- `SETUP`: The container was initialized successfully and is now running in setup mode (target rootfs mounted at /setup)

* List all available containers and their state:
```
control list
```

* Container config:
```
control config <container-uuid>
```
Prints the container config to stdout. This could be used for editing the current config.

* Update a container configuration file
```
control update_config <container-uuid> --file=<container.conf>
```
Sends a new config file `container.conf` to the `cmld`. It is not applied to running
containers directly. You have to stop the container and trigger a reload.
* Reload container configuration
```
control reload
```
This command reloads all container configs of all conatiners in state `STOPPED`
into the running `cmld`.
   
* Run command inside the specified container:
```
control run <command> [<arg_1> ... <arg_n>] <container-uuid>
```
**Experimental**, some characters especially control characters may not work as expected.    
The given command is executed in the context of the given container. Before executing the command, the process is made session leader and get's a new PTY attached. 

* Assign a network interface to the container with the specified UUID:
```
control assign_iface --iface <interface-name> <container-uuid> [--persistent]
```
The 'persistent' option also applies the change to the container's config file, such that the assignment persists container reboot

* Un-assign a network interface to the container with the specified UUID:
```
control unassign_iface --iface <interface-name> <container-uuid> [--persistent]
```
The 'persistent' option also applies the change to the container's config file, such that the assignment persists container reboot.

* List all network interfaces currently assigned to the container with the specified UUID:
```
control ifaces <container-uuid>
```
* Freeze container:
```
control freeze <container-uuid>
```
Currently not supported

* Unfreeze container:
```
control unfreeze <container-uuid>
```
Currently not supported

* List GuestOS configuration:
```
control list_guestos
```
Lists the configuration of all installed GuestOSes.

* Register a new CA for GuestOS signature verification
```
control ca_register <ca.cert>
```
This command registers a new certificate in trusted CA store for allowed GuestOS signatures.

* Install new GuestOS or update GuestOS
```
control push_guestos_config <guestos.conf> <guestos.sig> <guestos.pem>
```
This command can be used to push the specified GuestOS config, signature, and certificate files
of a corresponding GuestOS. This would trigger a download of the corresponding GuestOS image
files from either the download URL provided in the `guestos.conf` or the default
download url in the CML's `device.conf`. If an older version is already installed an
update would be triggered after a reboot of the whole system.

* Remove/Delete a GuestOS
```
control remove_guestos <guestos name>
```
Deletes the GuestOS by the given `guestos name`. All associated files such as
configuration, signature and shared images files are removed permanently.
If the GuestOS is still in use by any container, this command will do nothing.
In this case you have to delete all remaining containers using the GuestOS
with `control remove`, see above.

* Get Certificate request of Device
```
control pull_csr <device.csr>
```
During provisioning mode (first boot of device) it is posible to
sign a device private key with a customer CA. This command pulls
the corresponding csr from the virtualization layer and
stores the csr in the provided file name `<device.csr>`.

* Store the signed device certificate
```
control push_cert <device.cert>
```
This command pushes back a device certificate provided in file `<device.cert>`.
Remember, you have to provide your own mechanism to transfer and sign
the certification request to your CA. Further, you have to store the
corresponding Customer CA root certificate in the trusted CA store, see `control ca_register`.



* Change the softtoken's PIN/Passphrase
```
control change_pin
```
This command provides an interactive prompt for old and new
Pin/Passphrase for the softtoken. Remember, the softtoken is used for
wrapping the container encryption keys and must be provided to container
start. (default passphrase after self-provisioning is `trustme`


## Containers and guest operating systems

Each container has a corresponding configuration file defining its key features.
Every container runs a certain operating system, which has also to be defined in a configuration file.
Note that several containers may run the same operating system.
Both container and OS configuration file formats are composed of several '<key>: <value>' lines.

## Container configuration

All containers currently available are defined in the configuration files in the form `<container-uuid>.conf` in the '/data/cml/containers/' folder. The `<container-uuid>` is a Universal Unique Identifier provided in five character groups separated by hyphens, e.g., 11111111-1111-1111-1111-111111111111. A minimal container configuration should define container name as well as guest OS name and version, for example:
```
name: "core0"
guest_os: "idsos"
guestos_version: 1
```

Accessing those files directly is not possible from core0 container.
Only in development builds with enabled `'debug-tweeks'`, you can access this
through CML shell on tty12.

Intended control of container configuration is gained through the control interface from within
core0 container.
The corresponding commands of the control tool are `create`, `config` and `update_config`


**create** uses a container.config skeleton as shown above, and writes it to corresponding
config files in `/data/cml/containers/` with a random uuid. The corresponding created config
is also responded back to core0 through the control interface.

**config** requests the configuration from an existing container. Similar to
the answer of create. You can write it to some location inside core0 to edit it.

**update_config**  writes back an edited config or just overwrites the existing
config by the provided skeleton.


## GuestOS configuration

An operating system configuration is defined in the `<os-name>-<os-version>.conf` in the `/data/cml/operatingsystems/` folder. The configuration includes OS name and version, supported hardware type, description of mounted images, init process path and parameters, build date, and others. Descriptions of mounted images include their hashes, size, and file system type.

Similar to container configuration also direct access to guestOS specific configuration
files is not possible from core0 container.
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

