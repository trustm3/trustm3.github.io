---
title: PKI
category: General
order: 4
---

- TOC
{:toc}

# Overview of the trust\|me PKI solution
<img alt="trustme pki architecture" src="img/pki_trustm3.png" width="55%">

In order to sign all trust\|me software artifacts and to provide device identities (e.g., for secure remote
management), trust\|me proposes a PKI solution that can be tailored to the requirements of a spefic
deployment. This allows the secure operation of trust\|me regarding device and software lifecycle when
operating the PKI properly - an aspect that you need to handle yourself and for which trust\|me implements the
necessary foundations.

The illustration gives an overview of our proposed PKI solution, which comes with to hierarchies.
The _Software Signing PKI_ is designated for signing the base system, i.e., kernel, virtualization layer,
GuestOS for the core container.
The _Customer PKI_ has two purposes: signing the other GuestOSes and issuing device certificates.
The term customer PKI expresses aspects the customer wants to control while the software
signing PKI provides the core components a customer usually does not want to deal with.
In other terms, you might want to build the trust\|me software stack with your own PKI and have the necessary
trust anchors deployed on the platforms to be enrolled. This ensures that only the trust\|me base system may
boot. Before bringing the devices in the field, the party that actually utilizes the devices in its ecosystem,
creates its own customer PKI to sign device certificates and to sign its own, custom containers that may
only ever run on the platform.
In the following, we will take a closer look at the two hierarchies.

### Software Signing PKI

The _Software Signing Root CA_ serves as root of trust for the trust\|me base system software.
This CA can create different _Software Signing Sub CAs_ which can be used to sign the kernel, virtualization
layer, or core container when building the trust\|me base system.
On a secured platform, the base system needs to be started first and verified during the boot of a device.
The illustration exemplifies an UEFI Secure Boot case where the public keys of the Software Signing Sub CA can
be stored in the UEFI _Signature Database_. Note that the UEFI Secure Boot requires to set a platform-individual
_Key Exchange Key_ and _Platform Key_, which is out of scope of this PKI hierarchy.

### Customer PKI

The _General Root CA_ serves as root of trust for customer-specific (in other words, use case specific) trust
relationships, which are currently GuestOS signing and providing device identities.
The _GuestOS Signing Sub CA_ can be used to sign the different GuestOSes that are running as containes on the
trust\|me base system. Each GuestOS thus gets its own certificate which can be verified by the trust\|me base
system.
The _Device Sub CA_ can be used to issue device certificates. This, for instance, allows a secure
communication with a backend.

# Generating the PKI

This part covers the building of the PKI. The deployment of trust anchors and signing of CSRs is covered on
the [roll-out page](/provisioning).

### Build-time generation 

When building trust\|me for the first time, our PKI solution is automatically generated
from source, if not yet generated. Leaving the solution unmodified, automatic generation
results in a _development_ PKI instantiation (possibly not yet applicable for productive use).
Especially, this means that private keys are stored locally on the filesystem.

### Manually generate development PKI from source

You can also build our PKI solution manually and tailor it to your specific requirements.
For instance, you might want to generate/store root CA keys in HSMs, or adapt signing algorithms and
certificate attributes to employ the PKI solution in practice. Once signing keys are no longer stored inside
the build system directories, you will need to adapt the software artifact signing procedure to use keys,
e.g., from HSMs.

```
bash ws-yocto/trustme/build/device_provisioning/gen_dev_certs.sh
```
