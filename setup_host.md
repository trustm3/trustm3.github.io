---
---
# Setup build host

This document describes the process of building, setting up, and starting of the trust\|me system.  
The instructions where tested on Debian Stretch (x86-64).  
You can either build trust\|me natively on your host or use the preconfigured, docker-based build environment. Choose the option that best fits your requirements.

## Docker-based build environment

Execute the following commands on your build host to setup your build environment
```
cd trustme/build/yocto/docker
docker build -t trust\|me-builder .
./run-docker.sh <path-to-ws-yocto>
```

## Setup host natively

The trust\|me build needs packages from main and contrib archive areas. If not already done so, enable contrib in your sources.list.

To setup your build host, install the following Packages:
```
apt-get install repo chrpath diffstat g++ gawk makeinfo
```
