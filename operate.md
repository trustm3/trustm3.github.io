---
---

# Basic trust|me operation

trust|me is operated using a socket-based interface. This interface is used by the *control* command-line tool, which is installed together with trust|me. It is available from the priviledged container (c0) and the debugging shell.
The *control* tool allows administration and configuration of the trust|me platform, such as creating and starting containers, running a given command command inside a container, etc.  
The available actions are listed below.

* Start container:
```
   control start <container-uuid> --key=<token-key>
```

This command starts the container with the given UUID. In order to do so, the container has to be created beforehand by 'control create'

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
The configuration template file has to be given using protobuf syntax. It must have at least the following content:
```
name: "<container name>"
guest_os: "<guest os name>"
guestos_version: <guestos-version>
```

* Container current state:
```
   control state <container-uuid>
```
This returns the current state of the given container. Possible states are:
	- STOPPED: The container has either not be run, was explicitly stopped or the init process of the container exited
	- STARTING: The container was started using 'control start' and the container manager is preparing to launch the container.
	- BOOTING: The container manager finished preparing the container and the control was handed over to the container's init process
	- RUNNING: The container was initialized successfully and is now running.

* List all available container and their status:
```
   control list
```
   
* Run command inside the specified container:
```
control run <command> [<arg_1> ... <arg_n>] <container-uuid>
```
The given command is executed in the context of the given container. Before executing the command, the process is made session leader an get's a new PTY attached.

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

# trust\|me system components

## Containers and guest operating systems

Each container has a corresponding configuration file defining its key features.
Every container runs a certain operating system, which has also to be defined in a configuration file.
Note that several containers may run the same operating system.
Both container and OS configuration file formats are composed of several '<key>: <value>' lines.

## Container configuration

All containers currently available are defined in the configuration files of the form '<container-uuid>.conf' in the '/data/cml/containers/' folder. The <container-uuid> is a Universal Unique Identifier provided in five character groups separated by hyphens, e.g., 11111111-1111-1111-1111-111111111111. A minimal container configuration should define container name as well as guest OS name and version, for example:
```
name: "core0"
guest_os: "idsos"
guestos_version: 1
```

## OS configuration

An operating system configuration is defined in the '<os-name>-<os-version>.conf' in the '/data/cml/operatingsystems/' folder. The configuration includes OS name and version, supported hardware type, description of mounted images, init process path and parameters, build date, and others. Descriptions of mounted images include their hashes, size, and file system type.
