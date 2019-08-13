---
---

# Roll-out and Provisioning

For secure roll-out and provisioning, several steps in a secure environment are required.

### Set secure boot keys

Before the deployment of the base system, set the trust anchors of the software signing PKI, e.g., in UEFI variables.
Set other required keys, e.g., the KEK and PK.

### Boot into provisioning mode

At the first boot, trust\|me automatically boots into a provisioning mode.
In this mode, the virtualization layer exposes additional functionality to the core container via the control
interface.
At that time, the deployed trust\|me base system is equipped with the software signing trust anchor and is
thus capable of starting the core container. To allow running other GuestOSes, you can add the GuestOS signing trust
anchors to the certificate store in the virtualization layer (via the control interface). This allows the
virtualization layer to verify any GuestOS that can be added to the system via the core container at any
time.

Further, the virtualization layer initializes the TPM and creates a device CSR corresponding to a TPM-bound
signing keypair. The core container can request the CSR from the virtualization layer.
This allows to sign the device CSR using the Device Sub CA and to deploy the device certificate back to the
target. During this step, the trust anchor for device certificates can be also be deployed in order to verify
other device certificates, such as a backend, which may have a device identity in the same PKI hierarchy.

As a last step, the virtualization layer creates a softtoken for wrapping container encryption keys.
This token can be replaced during provisioning with an own token, its password can be changed,
or the token can be removed to use a Secure Element in future builds.  

### Complete provisioning

Reboot the system to complete the provisioning. During the next boot, the trust\|me base system no longer
boots into provisioning mode and can use the deployed certificates and tokens during operation.
This menas that the control interface available in the core container no longer exposes the provisioning
functionality.
Future builds will come up with functionality to replace relevant key material during the operation to
complete the device lifecycle management.

## Self-Provisioning
Current development builds contain a self-provisioning mechansim.
The self provisioning first generates self-signed device certificates for testing.
The pre-deployed software signing trust anchors is used on development builds to verify all containers, i.e.
core containers and other deployed containers.

Further, an initial softtoken with password `trustme` is generated.
The token is used to wrap the encryption keys for container data and emulates a Secure
Element (or other tamper-resistant key store).
In future builds, trust\|me will allow storing these  keys in Secure Elements.
At every container start, the password for the softtoken must be provided via the core
container to the virtualization layer, which unlocks the softtoken or Secure Element, respectively.

### Complete self-provisioning
Just reboot the system after finishing initial provisioning with shell

    reboot

## Change the softtoken password
There are two options to change the passphrase of the softtoken.

### 1. Control Interface
You can just use the control tool to change the token's passhprase
at any time, either during provisioning or later on during normal operation by:
```
control change_pin
```

### 2. Oppenssl tool
You can change the password in the virtualization layer by running `openssl` during
provisioning mode or at the debug shell in development builds.
```
# unwrap existing token
openssl pkcs12 -in /data/cml/tokens/testuser.p12 -out /tmp/mycert.pem -nodes
# rewrap existing token, set new password
openssl pkcs12 -export -out /data/cml/tokens/testuser.p12 -in tmp/mycert.pem
# remove temp file
rm /tmp/mycert.pem
```
