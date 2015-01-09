Kernel Config
=============

https://community.cloud.online.net/t/official-linux-kernel-new-modules-optimizations-hacks/226

The kernel is built with the official mainline kernel, here are the .config files used.

Modifications
-------------

The only things we added are:

- a kernel module to simulate some virtualization features: handling serial console, being able to soft reset a node from the API
- a dts file, so the kernel can understand our hardware

---

Build a custom kernel using Docker
==================================

Run a `make menuconfig` for `.config-3.18-std`

    make menuconfig CONFIG=.config-3.18-std

Build a kernel with `.config-3.17-std` file

    make build CONFIG=.config-3.17-std

Create a new `.config-3.10-new` file from scratch for kernel `3.10`

    make defconfig CONFIG=.config-3.10-new

Debug the build container / run custom command

    make run CONFIG=.config-3.17-std

Test a kernel with QEMU
=======================

You should use a config file made for `versatile`.

Build a `3.18` kernel for `versatile`:

    make build CONFIG=.config-3.18-defconfig_versatile

Run the kernel in qemu

    make qemu CONFIG=.config-3.18-defconfig_versatile

---

Build a custom kernel from scratch (without Docker)
===================================================

Prerequisites:
--------------

- An arm(hf) compiler:
  - a cross-compiler on non-armhf host, ie `gcc-arm-linux-gnueabihf`
  - a standard compiler from an armhf host (you can build a kernel from your C1)
- Theses packages: git, wget, make


Steps:
------

- Configure environment
  ```bash
export VERSION=3.17
export ARCH=arm
export ARTIFACTS=artifacts
```

- Download archive via web
  ```bash
wget https://kernel.org/pub/linux/kernel/v3.x/linux-$VERSION.tar.xz && tar xf linux-$VERSION.tar.xz
  ```
  or via git
  ```bash
git clone -b v$VERSION --single-branch git://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git linux-$VERSION
```

- Generate a base `.config` file by building it
  ```
make ARCH=arm mvebu_v7_defconfig
```
  or by fetching our one
  ```
wget -O .config https://raw.githubusercontent.com/online-labs/kernel-config/master/.config-$VERSION-std
```

- Tune the .config
  ```bash
make ARCH=arm menuconfig
# ... configure using console interface
```

- Building kernel and modules
  ```bash
make -j $(echo `nproc` ' * 2' | bc) uImage modules LOADADDR=0x8000
```

- Export the artifacts (kernel, header, modules) to `$ARTIFACTS` directory
  ```bash
mkdir -p $ARTIFACTS
cp arch/arm/boot/uImage $ARTIFACTS/
cp System.map $ARTIFACTS/
cp .config $ARTIFACTS/
make headers_install INSTALL_HDR_PATH=$ARTIFACTS/ > /dev/null
find $ARTIFACTS/include -name ".install" -or -name "..install.cmd" -delete
make modules_install INSTALL_MOD_PATH=$ARTIFACTS/ > /dev/null
rm -rf $ARTIFACTS/modules && \
   mv $ARTIFACTS/lib/modules $ARTIFACTS && \
   rmdir $ARTIFACTS/lib && \
   rm $ARTIFACTS/modules/*/source $ARTIFACTS/modules/*/build
```

Minimal configuration for C1 servers
------------------------------------

```gherkin
- Networking support
  - Networking options
    - 802.1Q/802.1ad VLAN Support -> **YES**
    - Packet socket -> **YES**
    - Unix domain sockets -> **YES**
- Device Drivers
  - Network device support
    - PHY Device support and infrastructure
      - Driver for MDIO Bus/PHY emulation with fixed speed/link PHYs -> **YES**
  - Block devices
    - Network block device support -> **YES**
- Kernel hacking
  - Kernel low-level debugging functions -> **YES**
  - Early prink -> **YES**
- File systems
  - The Extended 4 (ext4) filesystem -> **YES**
```

Releases
========

2014-10-15 - 3.17.0-85
----------------------

- used .config: https://github.com/online-labs/kernel-config/blob/3.17.0-85/.config-3.17-std
- community discuttion: https://community.cloud.online.net/t/official-linux-kernel-new-modules-optimizations-hacks/226/6?u=manfred
- kernel commit: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tag/?id=v3.17

Licensing
=========

© 2014-2015 Online Labs - [MIT License](https://github.com/online-labs/kernel-config/blob/master/LICENSE).
