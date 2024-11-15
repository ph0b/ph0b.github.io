---
id: 105
title: 'Adding dev tools (gcc, make&#8230;) to Galileo SD image and using nodejs with native extensions'
date: '2014-01-15T19:01:10+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=105'
permalink: /adding-dev-tools-gcc-make-to-galileo-sd-image/
categories:
    - Arduino
tags:
    - arduino
    - galileo
    - Intel
    - nodejs
    - x86
    - yocto
---

[Galileo](http://arduino.cc/en/ArduinoCertified/IntelGalileo) is an **Arduino** that runs on a (x86) Intel® Quark X1000 SOC.

![IntelGalileo_fabD_Front_450px](/wp-content/uploads/2014/01/IntelGalileo_fabD_Front_450px-150x150.jpg)

While it’s fully compatible with **Arduino** Software Development Environment, it’s also capable of running a **full Linux distribution**.

**Sept. 2014 update**: GCC and other dev tools are now part of the official linux image you can download from [Intel Developer Zone](https://software.intel.com/en-us/iot/downloads).

You can get such distribution from the [official website](https://communities.intel.com/docs/DOC-22226). The “LINUX IMAGE FOR SD for Intel Galileo” is a [poky](https://www.yoctoproject.org/tools-resources/projects/poky) distribution built using [Yocto](https://www.yoctoproject.org). It’s using **uClibc** and contains useful binaries like **SSHd**, **bluez**, **nodejs** and **OpenCV** while it can still run **Arduino sketches** !

It’s really simple to install it: you only need to unzip the .7z package at the root of a FAT32 formatted microSD.

I’ve came across some interesting nodejs modules, and some of them are relying on native extensions. Cross-compiling these seemed complicated so I’ve chosen another way to get these working on my Galileo: rebuilding the poky distribution to add dev tools.

And while it may sound complicated… it’s in fact dead simple thanks to the way **Yocto** is working.

First set up your build environment the same way you would build the same image than the one distributed on the official website, by downloading Galileo BSP and setting up Yocto:

```shell
wget http://downloadmirror.intel.com/23171/eng/Board_Support_Package_Sources_for_Intel_Quark_v0.7.5.7z
7z x Board_Support_Package_Sources_for_Intel_Quark_v0.7.5.7z
tar xzvf Board_Support_Package_Sources_for_Intel_Quark_v0.7.5/meta-clanton_v0.7.5.tar.gz
cd meta-clanton_v0.7.5
./setup.sh
source poky/oe-init-build-env yocto_build
```

Stop here to create your custom configuration:

```shell
cp ../meta-clanton-distro/recipes-core/images/image-full.bb ../meta-clanton-distro/recipes-core/images/image-custom.bb
```

Here I’ve chosen to triple the filesystem size to make it around 900mo:

```shell
IMAGE_ROOTFS_SIZE = "921600"
```

added -dev packages to the image by adding **dev-pkgs** feature:

```shell
IMAGE_FEATURES += "package-management dev-pkgs"
```

added the tools I wanted by adding this line:

```shell
IMAGE_INSTALL += "autoconf automake binutils binutils-symlinks cpp cpp-symlinks gcc gcc-symlinks g++ g++-symlinks gettext make libstdc++ libstdc++-dev file coreutils"
```

and also integrated the right firmware for my Wi-Fi/BT4.0 chipset ([Centrino 6235](http://www.amazon.com/Intel-Centrino®-Advanced-N-6235-6235ANHMW/dp/B009SJTSWU)) by adding this line:

```shell
IMAGE_INSTALL += "linux-firmware-iwlwifi-6000g2b-6"
```

Finally you can build your custom distribution:

```shell
bitbake image-custom
```

For me this process took around 20GB space and around an hour of automated download/build. Once its done you’ll find your generated files inside *yocto\_build/tmp/deploy/images*. You can copy these to your microSD like so:

- latest bzImage-\* as *bzImage*
- latest core-image-minimal-initramfs-\* as *core-image-minimal-initramfs-clanton.cpio.gz*
- latest image-\* as *image-full-clanton.ext3*
- boot folder as *boot*

Now you can easily get useful **nodejs** **modules** with native extensions – they will be compiled on the Galileo itself :

*npm install socket.io*
[![socketio-build](/wp-content/uploads/2014/01/socketio-build.png)](/wp-content/uploads/2014/01/socketio-build.png)

And if you don’t have the time and bandwidth to spare, you can directly download the [custom image](/wp-content/uploads/2014/01/linux_image_for_sd_with_dev_tools.zip) I’ve generated 🙂

update: There is now an official SD image with gcc support: <https://software.intel.com/en-us/iotdevkit>

If you want to get more information on how to build your own image or learn how to use another libc than uClibc, you should read this article: [Intel Galileo – Building Linux Image](http://www.malinov.com/Home/sergey-s-blog/intelgalileo-buildinglinuximage)

PS: I’ve encountered one small bug with the image I’ve generated, stdbuf couldn’t find ‘libstdbuf.so’. You can easily fix this by running `ln -s /usr/lib/coreutils/libstdbuf.so /usr/lib/coreutils/coreutils/`
