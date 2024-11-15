---
id: 217
title: 'What you should know about .so files'
date: '2015-06-30T14:23:50+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=217'
permalink: /android-abis-and-so-files/
nkweb_code_in_head:
    - default
nkweb_Use_Custom_js:
    - default
nkweb_Custom_js:
    - ''
nkweb_Use_Custom_Values:
    - default
nkweb_Custom_Values:
    - ''
nkweb_Use_Custom:
    - 'false'
nkweb_Custom_Code:
    - ''
categories:
    - Android
tags:
    - Android
    - NDK
---

In its early days, the Android OS was pretty much supporting only one CPU architecture: ARMv5.  
Do you know how many it does support now? … **7**!

Seven distinct CPU-architectures are currently supported by the Android OS: *ARMv5*, *ARMv7* (since **2010**), *x86* (**2011**), *MIPS* (**2012**), *ARMv8*, *MIPS64* and *x86\_64* (**2014**). Each of them is associated with a respective **ABI**.

An **A**pplication **B**inary **I**nterface is the definition of how binaries (esp. *.so files*) should be made in order to work on the platform, from the instruction set used and memory alignment, to the system libraries that are available. On Android there is one per CPU architecture:  *armeabi*, *armeabi-v7a*, *x86, mips, arm64-v8a*, *mips64*, *x86\_64*.

## Why should you care about .so files in the first place?

If you’re using the NDK, you’re generating .so files, so you obviously already care. If you’re dealing only with Java code, you could think you don’t need to, as it’s cross-platform… but in fact, even in this case, quite often, you may rely without knowing it on libraries and engines that make your app embed *.so files* and become ABI-dependent/

Using the RenderScript support Library? OpenCV? Unity? android-gif-drawable? SQLCipher? You’ve got .so files inside your APK, and you need to care about these.

The ABIs supported by your Android application are determined by the presence of *.so files* inside your APK, under the “**lib/&lt;ABI&gt;**” folders where ABI can be one or more of *armeabi*, *armeabi-v7a*, *arm64-v8a*, *mips, mips64*, *x86*, *x86\_64* (the current seven ABIs).

[![Native Libs Monitor - screenshot](/wp-content/uploads/2015/06/image00-1024x640.png)](/wp-content/uploads/2015/06/image00.png)[*Native Libs Monitor*](https://play.google.com/store/apps/details?id=com.xh.nativelibsmonitor.app) *can help you understanding what .so files are embedded inside your APK and installed on your device, and which libraries and frameworks they’re coming from.*

Most devices are compatible with more than one ABI. For example, ARM64 devices and x86 devices can run *armeabi-v7a* and *armeabi* binaries too! But it’s always better to provide libraries compiled for the preferred ABI of the device. This way, libraries can run without an emulation layer (in case of arm libs on x86 devices), and get more performance thanks to the recent architecture improvements (hardware fpu, more registers, better vectorization, etc).

You can get the list of supported ABIs, sorted by preference, from *Build.SUPPORTED\_ABIS*. But you shouldn’t need to read it from your app, as Android’s package manager will automatically install the *.so files* compiled for your device preferred ABI that are present inside your APK, if they’re properly present under **lib/&lt;ABI&gt;**.

## That’s where things may go wrong for your app!

There is a dead simple but not well-known important rule to follow when dealing with .so files.

You should provide libraries optimized for each ABIs if possible, but it’s all in or nothing: **you shall not mix**. You have to provide the full set of the libraries you’re using in each ABI folder.

When an application is installed on a device, only the *.so files* from **one** of its supported architectures are getting installed. On x86 devices, that can be the libs from the x86 folder if there is one, else from the armeabi-v7a, else armeabi folder (yes, x86 devices support these too).

## But they may go wrong in many other ways too…

When you include a .so file, not only the architecture matters. Here is a list of really common mistakes I see from other developers, that are leading to “UnsatisfiedLinkError”, “dlopen: failed”, and other crashes or bad performance:

### **using a .so file that has been compiled against android-21+ platform version, to run it on android-15 device. Ouch.**

When using the NDK, you may be tempted to “compile against the latest platform”, but it’s in fact quite wrong as ndk platforms aren’t backward compatible, they’re forward compatible! Use the same platform level than your app’s *minSdkVersion*.

That means that when you’re including a .so file that has already been compiled, you have to check the version it has been made to run on.

### **mixing .so files that are using different C++ runtimes**

.so files can rely on various C++ runtimes, statically compiled or dynamically loaded. Mixing versions of C++ runtimes can lead to many weird crashes and should be avoided. As a rule of thumb, when having only one .so file, it’s ok to statically compile a C++ runtime, else you should have them all dynamically linking the same C++ runtime.

That means that when you’re including a .so file that has already been compiled and you already have some of them, you have to check the C++ runtime it’s using.

### **not providing all the libs for each of the supported platforms**

This was already previously described, but you should really pay attention to this, as it may happen behind your back.

Example: You have an Android app with .so files for armeabi-v7a and x86 architectures. Then you add a dependency on another library that embeds .so files and supports more architectures, as android-gif-drawable: *compile ‘pl.droidsonroids.gif:android-gif-drawable:1.1.+’*.

Release your app, and then you’ll see it crashing on some devices, like the Galaxy S6, because only the .so files from the 64-bit directory got installed.

Solution: recompile your libs to support the missing ABIs, or set *ndk.abiFilters* to the one you support.

One more thing: **If you’re a library provider, and you’re not supporting all the ABIs, you’re screwing up your customers, as they can’t support more ABIs than you do.**

### putting .so files in the wrong place

It’s easy to get confused on where .so files should be put or generated by default, here is a summary:

- **jniLibs/ABI** inside an Android Studio project (can be changed using *jniLibs.srcDir* property)
- **libs/ABI** inside an eclipse project (that’s where ndk-build generates libs by default too)
- **jni/ABI** inside an AAR (the libs will get automatically included in the APK that references the AAR).
- **lib/ABI** inside the final APK
- inside the app’s **nativeLibraryPath** on a &lt;5.0 device / **nativeLibraryRootDir/CPU\_ARCH** on a &gt;=5.0 device (done by PackageManager during app installation)

### **providing only armeabi libs and ignoring all the other ABIs**

armeabi is supported by all the x86/x86\_64/armeabi-v7a/arm64-v8a devices, so that may look like a good trick to remove all the other ABIs to generate a light APK. But it’s not: there is an impact on performance and compatibility, and not only for the library itself.

x86 devices can run ARM libraries quite well, but it’s not 100% crash proof, particularly on the older devices. 64-bit devices (arm64-v8a, x86\_64, mips64) can all run 32-bit libraries, but they do so in 32-bit mode, running a 32-bit version of ART and Android components, loosing the 64-bit performance improvements that have been made on the 64-bit version of ART, webview, media…

Reducing APK’s weight is in any case a false excuse, as you can upload distinct ABI-dependent APKs to the Play Store (for the same app entry, of course). You only have to switch to advanced mode and upload your APKs with different ABI support and versionCodes. Generating these APKs is as simple as adding this to your build.gradle:

```gradle
android {
    ... 
    splits {
        abi {
            enable true
            reset()
            include 'x86', 'x86_64', 'armeabi-v7a', 'arm64-v8a' //select ABIs to build APKs for
            universalApk true //generate an additional APK that contains all the ABIs
        }
    }
 
    // map for the version code
    project.ext.versionCodes = ['armeabi': 1, 'armeabi-v7a': 2, 'arm64-v8a': 3, 'mips': 5, 'mips64': 6, 'x86': 8, 'x86_64': 9]
 
    android.applicationVariants.all { variant ->
        // assign different version code for each output
        variant.outputs.each { output ->
            output.versionCodeOverride =
                    project.ext.versionCodes.get(output.getFilter(com.android.build.OutputFile.ABI), 0) * 1000000 + android.defaultConfig.versionCode
        }
    }
 }
```

### Using private system libraries

The libraries you can use are the ones which are part of the NDK ones. Any other lib isn’t guaranteed to work across devices and may trigger crashes starting with Android N: https://developer.android.com/preview/behavior-changes.html#ndk

TL;DR:

[![One does not simply include random libs and .so files inside an android app.](/wp-content/uploads/2015/06/nnhuh.jpg)](/wp-content/uploads/2015/06/nnhuh.jpg)
