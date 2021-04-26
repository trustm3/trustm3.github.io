---
title: Build Environment
category: General
order: 3
---
- TOC
{:toc}

# Setup build environment
{:.no_toc}

This page describes how to setup the build environment for the trust\|me system.
The instructions were tested on Debian Stretch (x86-64).
You can either build trust\|me natively on your host or use a preconfigured, Docker-based build environment.
Choose the option that best fits your requirements.

## Docker-based build environment
1. Install repo tool:
```
sudo apt-get install repo
```

2. Create and initialize workspace on host (for further information and available manifests see [build/initialize workspace](build/build#initialize-workspace))
```
mkdir ~/ws-yocto
cd ~/ws-yocto
repo init -u https://github.com/trustm3/trustme_main.git -b master -m <manifest file>.xml
repo sync -j8
```
3. Build Docker image
```
cd ~/ws-yocto/trustme/build/yocto/docker
docker build -t trustx-builder .
```
4. Start Docker
Please ensure you are logged in as a non-root user. Otherwise, bitbake will refuse to run. A tutorial on how to run docker as a normal user can be found [here](https://docs.docker.com/install/linux/linux-postinstall/)
```
cd ~/ws-yocto/trustme/build/yocto/docker
./run-docker.sh ~/ws-yocto
```
5. Follow build instruction from [Setup Yocto environment](/build/build#setup-yocto-environment) inside the Docker container


## Setup host natively

The trust\|me build needs packages from main and contrib archive areas. If not already done so, enable contrib in your sources.list.

1. To setup your build host, install the following packages required for Yocto/Poky (see
[Yocto reference manual](https://www.yoctoproject.org/docs/2.6.2/ref-manual/ref-manual.html#required-packages-for-the-build-host))
```
apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib \
     build-essential chrpath socat cpio python python3 python3-pip python3-pexpect \
     xz-utils debianutils iputils-ping
```
2. Install additional required packages for repo tool and image signing
```
apt-get install repo python-protobuf python3-protobuf
```

### IDS trusted connector specific requirements (optionally)
Install the following software packages for a Yarn-based
build of the [Trusted Connector](/index#use-cases) core compartment.
Note that this step is already included in the Docker-based environment above.


1. Install Java 8 JDK and additional dependencies
```
apt-get update
apt-get install kmod procps curl apt-get install openjdk-8-jdk-headless openjdk-8-jre-headless
```
2. Install Nodejs 11
```
curl -sL https://deb.nodesource.com/setup_11.x | bash -
apt-get install nodejs
```
3. Install Yarn
```
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add -
echo "deb http://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list
apt-get update
apt-get install yarn
```
