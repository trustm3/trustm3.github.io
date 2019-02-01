---
---

# What is trust|me

trust\|me abbreviates "trusted mobile equipment" and is a OS-level virtualization solution. It is based on Linux specific features like namespaces, cgroups and capabilities to provide isolation of different guest Operating System Stacks on top of one Linux kernel. Devices are virtualized either directly in the OS kernel or with a proxy/server in the Android HAL. The picture below shows the System Architecture of trust\|me.
The trust\|me platform was originally developed to separate Android OS stacks. Afterwards it was extended to support arbitrary x86 Linux systems.

> **Note**: Android is supported on the Google Nexus 5X (bullhead) and Nexus 5 (hammerhead) devices only!

![trustme system architecture](https://github.com/trustm3/trustme_main/raw/master/doc/architecture.png)

A detailed introduction to the used concepts is provided in
[trustme.pdf](https://github.com/trustm3/trustme_main/raw/master/doc/trustme.pdf)

# IDS / Trusted Connector

The trust\|me code base is also used for our platform reference implementation for an IDS connector.
The [Industrial Data Space (IDS)](http://www.industrialdataspace.org/en/)
provides concepts for a generic shared data cloud in the IoT context.
The following figure shows the generic architecture of trust\|me

![trust\|me system architecture](https://github.com/trustm3/trustme_main/raw/master/doc/trust-x-ids.png)

We reuse only the container management layer of trust\|me (ramdisk). The whole Android high level user space stack is
replaced by a generic GNU user space stack. The "business logic" which is not part of this code release is implemented
on a generic JAVA stack inside the GNU containers.

Since on IoT Gateways no user interaction and thus no user authentication is needed, we do not need a
smartcard daemon. However platform authentication is required to allow remote connectors to communicate with each other.
For this purpose a TPM 2.0 chip is used. Currently the code base includes a tpm2simulator based on the work
[stwagnr/tpm2simulator] (https://github.com/stwagnr/tpm2simulator).

# Publications

|Year|Title|Authors|PDF|
|--|--------------------|----------|--------|
| 2018	| An Ecosystem and IoT Device Architecture for Building Trust in the Industrial Data Space | Brost, Gerd Huber, Manuel Weiß, Michael Protsenko, Mykolai Schütte, Julian Wessel, Sascha | [https://doi.org/10.1145/3198458.3198459](https://doi.org/10.1145/3198458.3198459)|
|2017 | Freeze & Crypt: Linux Kernel Support for Main Memory Encryption | Manuel Huber, Julian Horsch, Junaid Ali, Sascha Wessel, | [http://dx.doi.org/10.5220/0006378400170030](http://dx.doi.org/10.5220/0006378400170030) |
| 2015 | A Secure Architecture for Operating System-Level Virtualization on Mobile Devices | Manuel Huber, Julian Horsch, Michael Velten, Michael Weiß, Sascha Wessel | [http://dx.doi.org/10.1007/978-3-319-38898-4_25](http://dx.doi.org/10.1007/978-3-319-38898-4_25) |
| 2015 | Improving mobile device security with operating system-level virtualization (Jornal) | Sascha Wessel, Manuel Huber, Frederic Stumpf, Claudia Eckert | [https://doi.org/10.1016/j.cose.2015.02.005](https://doi.org/10.1016/j.cose.2015.02.005) |
| 2013 |	Improving Mobile Device Security with Operating System-Level Virtualization | Sascha Wessel, Frederic Stumpf, Ilja Herd, Claudia Eckert |	[https://doi.org/10.1007/978-3-642-39218-4_12](https://doi.org/10.1007/978-3-642-39218-4_12) |
