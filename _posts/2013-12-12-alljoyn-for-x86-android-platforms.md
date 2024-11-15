---
id: 43
title: 'x86 version of AllJoyn for Android platforms'
date: '2013-12-12T14:31:02+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=43'
permalink: /alljoyn-for-x86-android-platforms/
categories:
    - Android
tags:
    - AllJoyn
    - Android
    - Intel
    - library
    - x86
---

Alljoyn is a cross-platform library designed for peer-to-peer communication between various devices over different transports.

This library is open-source (Apache License) and initially created by Qualcomm Innovation Center. The project has joined the AllSeen Alliance two days ago and now it’s hosted here: <https://allseenalliance.org/source-code>

Even if the precompiled x86 version isn’t distributed from the official website, compiling it from the sources is perfectly supported.

When you download the release package for Android, here is what you get:

```text
cpp/    core AllJoyn functionality, implemented in C++
          - built from Git projects alljoyn_core and common
          - required for all AllJoyn applications
java/   optional Java language binding          (built from alljoyn_java)
          - required for Android apps
c/      optional ANSI C language binding        (built from alljoyn_c)
          - required by Unity binding
unity/  optional Unity language binding         (built from alljoyn_unity)
about/  implements AllJoyn About Feature. (built from about.git/(cpp and java))
```

 If you use the Java binding **alljoyn.jar**, you’ll end up with **liballjoyn\_java.so** in **lib/armeabi**/. The binaries (.so/.a files) from this package are ARMv5 only but compiling it from the sources is perfectly supported.

Here is the x86 release version I’ve compiled for you: [alljoyn-3.4.5-android-sdk-rel-x86.zip](/wp-content/uploads/2013/12/alljoyn-3.4.5-android-sdk-rel-x86.zip). It follows the same architecture than the ARMv5 release.

If you’re just using the AllJoyn .jar on Android, you can directly get the [liballjoyn\_java.so](/wp-content/uploads/2013/12/liballjoyn_java_x86.zip) and put it inside the **lib/x86** directory of your Android application.

If you want to recompile it yourself or know more about the process, here is what I did:

## Recompiling Alljoyn for x86-based Android platforms

First, get the sources:

```shell
git clone https://git.allseenalliance.org/gerrit/core/alljoyn
cd alljoyn
```

Then, retrieve **libcrypto.so** and **libssl.so** from a real x86 device or the x86 emulator and put them inside **build/android/x86/release/dist/cpp/lib/** :

```shell
mkdir -p build/android/x86/release/dist/cpp/lib/
adb pull /system/lib/libcrypto.so build/android/x86/release/dist/cpp/lib/
adb pull /system/lib/libssl.so build/android/x86/release/dist/cpp/lib/
```

Now you can build the lib for x86 as explained by the project documentation – the tedious part being the need of having a copy of the AOSP sources:

```shell
export CLASSPATH=/usr/share/java/junit.jar
scons OS=android CPU=x86 ANDROID_NDK=/opt/android-ndk ANDROID_SDK=/opt/android-sdk ANDROID_SRC=/home/ph0b/android-build VARIANT=release BINDINGS=cpp,c,java,unity
```
