---
---

# Roll-out and Provisioning

After first boot the CML starts in provisioning mode, where
initial device specific certificates could be deployed.
Current development builds contain a self provisioning mechansim.

## Self-Provisioning
The self provisioning just generates self-signed certificates for testing.
For instance the device csr for a TPM-owned private key is signed here.
Further, an initial user token with password `trustme` is generated.
The token is used to wrap the encryption keys for container data.
Thus, you need to provide this at every container start.

### Change the password
You can change the password in CML by running `openssl` during
provisioning mode or at the debug shell in development builds.

    # unwrap existing token
    openssl pkcs12 -in /data/cml/tokens/testuser.p12 -out /tmp/mycert.pem -nodes
    # rewrap existing token
    openssl pkcs12 -export -out /data/cml/tokens/testuser.p12 -in tmp/mycert.pem
    # remove temp file
    rm /tmp/mycert.pem

### Complete self-provisioning
Just reboot system after initial provisioning shell appears

    reboot
