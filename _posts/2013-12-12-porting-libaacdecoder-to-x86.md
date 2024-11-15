---
id: 35
title: 'Porting libaacdecoder to x86'
date: '2013-12-12T11:35:55+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=35'
permalink: /porting-libaacdecoder-to-x86/
categories:
    - Android
tags:
    - aac
    - Android
    - Intel
    - library
    - media
    - porting
    - x86
---

***Latest update: the maintainer of the lib accepted my patch, 0.7+ release packages now include x86 libs: [aacdecoder releases](https://code.google.com/p/aacdecoder-android/downloads/list "aacdecoder downloads")***

Porting is a big word here ! As often, adding x86 support to libaacdecoder was a matter of fixing Makefiles.

[libaacdecoder ](https://code.google.com/p/aacdecoder-android/ "libaacdecoder")supports AAC as well as MP3 decoding through the use of OpenCore decoders. It’s based on [opencore-aacdec](https://code.google.com/p/opencore-aacdec/) which compiles fine for x86.

The release package, as of 0.6.1, includes only binaries for ARMv5 and ARMv7. If you try to recompile it by yourself from the sources, you may even think that other architectures aren’t supported… but this is wrong!

If you’re in a hurry, here is the compiled library package (version 0.6.1) with the x86 version of libaacdecoder.so added: [aacdecoder-android-libs-0.6.1.zip](http://ph0b.com/wp-content/uploads/2013/12/aacdecoder-android-libs-0.6.1.zip)

And now for the “technical details”, 2 small changes where involved:

## 1. As usual, add x86 to the list of targeted ABIs

Open ./decoder/jni/Application.mk and add x86 to the list of ABIs or simply put “all”:

```
APP_ABI := all
```

Now if you call ndk-build again, the toolchain will try to generate x86 versions of the library as well, but here it will fail with many errors like these:

```
...
     [exec] ./codecs_v2/audio/mp3/dec/src/asm/pvmp3_dct_16_gcc.s:225: Error: no such instruction: `smull r9,r7,r11,r7'
     [exec] ./codecs_v2/audio/mp3/dec/src/asm/pvmp3_dct_16_gcc.s:226: Error: too many memory references for `add'
     [exec] ./codecs_v2/audio/mp3/dec/src/asm/pvmp3_dct_16_gcc.s:227: Error: too many memory references for `sub'
...
```

because it’s trying to integrate ARM assembly code. Let’s fix this:

## 2. Fix the list of sources set inside opencore-mp3dec Makefile

Open ./decoder/jni/opencore-mp3dec/Android.mk.

You’ll see ARM asm sources included:

```
LOCAL_SRC_FILES := \
...
 	src/asm/pvmp3_dct_16_gcc.s \
 	src/asm/pvmp3_dct_9_gcc.s \
 	src/asm/pvmp3_mdct_18_gcc.s \
 	src/asm/pvmp3_polyphase_filter_window_gcc.s
```

Of course these can’t be compiled for x86… but inside src/ you already have C++ equivalents that can ! So you just need to conditionally include these:

```
ifeq ($(TARGET_ARCH),arm)
 LOCAL_SRC_FILES += \
 	src/asm/pvmp3_dct_16_gcc.s \
 	src/asm/pvmp3_dct_9_gcc.s \
 	src/asm/pvmp3_mdct_18_gcc.s \
 	src/asm/pvmp3_polyphase_filter_window_gcc.s
else
 LOCAL_SRC_FILES += \
 	src/pvmp3_dct_16.cpp \
	 src/pvmp3_dct_9.cpp \
	 src/pvmp3_mdct_18.cpp \
 	src/pvmp3_polyphase_filter_window.cpp
endif
```

While you’re here, you’ll also see an ARM macro always defined:

```
LOCAL_CFLAGS := -DPV_ARM_GCC_V4 $(PV_CFLAGS)
```

It will be better to conditionally set it:

```
LOCAL_CFLAGS :=  $(PV_CFLAGS)
ifeq ($(TARGET_ARCH),arm)
  LOCAL_CFLAGS += -DPV_ARM_GCC_V4 
endif
```

Once these changes are done you’ll be able to compile libaacdecoder for all the supported architectures.

Here is the latest release package, updated with all the libs: [aacdecoder-android-libs-0.6.1.zip](http://ph0b.com/wp-content/uploads/2013/12/aacdecoder-android-libs-0.6.1.zip)