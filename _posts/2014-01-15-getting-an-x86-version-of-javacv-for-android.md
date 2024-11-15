---
id: 110
title: 'Getting an x86 version of JavaCV for Android'
date: '2014-01-15T19:54:46+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=110'
permalink: /getting-an-x86-version-of-javacv-for-android/
categories:
    - Android
tags:
    - Android
    - JavaCV
    - library
    - OpenCV
    - porting
    - x86
---

[JavaCV](https://code.google.com/p/javacv/) is a wrapper of mainly OpenCV and ffmpeg libraries. While these libs are perfectly compatible with x86 flavor of Android, JavaCV doesn’t provide the build scripts nor it integrate the x86 binaries.

It doesn’t take much to compile the proper x86 versions of JavaCV packages, but if you’re in a hurry, here they are:

- [javacv-0.7-bin-android-x86.zip](/wp-content/uploads/2014/01/javacv-0.7-bin-android-x86.zip)
- [javacv-0.7-cppjars-android-x86.zip](/wp-content/uploads/2014/01/javacv-0.7-cppjars-android-x86.zip)

Now let’s look at what was needed to build these. You need the usual build tools (gcc, make…) as well as maven.

You first need to recompile cppjars:

## Recompiling cppjars (OpenCV and ffmpeg)

You can easily do so by reusing the build scripts provided in the cppjars release package:

```shell
wget https://javacv.googlecode.com/files/javacv-0.7-cppjars.zip
unzip javacv-0.7-cppjars.zip
cd javacv-cppjars
```

The build scripts for Android are: **build\_opencv-android-arm.sh** and **build\_ffmpeg-android-arm.sh**. We need to create **build\_opencv-android-x86.sh** and **build\_ffmpeg-android-x86.sh** equivalents of these.

Start by copying these scripts, then in the -x86 version, replace :

- **arm-linux-androideabi-** by **i686-linux-android-** (toolchain binaries prefix)
- **arm-linux-androideabi-** by **x86-** (toolchain)
- **-march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16 -mfpu=neon…** by **-mtune=atom -mssse3 -mfpmath=sse** (compiler flags)
- **arm-linux** with **i686-linux**
- other **-arm** suffixes by **-x86**

Don’t do this blindly – it will work better if you understand what you’re doing 🙂

In the end here is the content I have for **build\_ffmpeg-android-x86.sh**:

```bash
ANDROID_BIN=$ANDROID_NDK/toolchains/x86-4.8/prebuilt/linux-x86_64/bin
ANDROID_ROOT=$ANDROID_NDK/platforms/android-14/arch-x86
tar -xjvf ffmpeg-$FFMPEG_VERSION.tar.bz2
mv ffmpeg-$FFMPEG_VERSION ffmpeg-$FFMPEG_VERSION-android-x86
cd ffmpeg-$FFMPEG_VERSION-android-x86
tar -xjvf ../last_stable_x264.tar.bz2
X264=`echo x264-snapshot-*`
cd $X264
./configure --enable-static --enable-pic --disable-cli --disable-opencl --cross-prefix=$ANDROID_BIN/i686-linux-android- --sysroot=$ANDROID_ROOT --host=i686-linux --extra-cflags="-fpic -pipe -DANDROID -DNDEBUG -mtune=atom -mssse3 -ffast-math -mfpmath=sse -fomit-frame-pointer -fstrict-aliasing -funswitch-loops -finline-limit=300" --extra-ldflags="-lm -lz -Wl,--no-undefined -Wl,-z,noexecstack"
make -j8
cd ../
patch -p1 < ../ffmpeg-$FFMPEG_VERSION-android-x86.patch
./configure --prefix=$ANDROID_NDK/../ --enable-shared --enable-gpl --enable-version3 --enable-libx264 \
 --disable-static --disable-symver --disable-doc --disable-ffplay --disable-ffmpeg --disable-ffprobe --disable-ffserver --disable-encoders --disable-muxers --disable-devices --disable-demuxer=sbg --disable-demuxer=dts --disable-parser=dca --disable-decoder=dca --disable-decoder=svq3 --enable-network --enable-version3 --disable-amd3dnow --disable-amd3dnowext --disable-outdev=sdl\
 --extra-cflags="-I$X264" --extra-ldflags="-L$X264" --enable-cross-compile --cc=$ANDROID_BIN/i686-linux-android-gcc --sysroot=$ANDROID_ROOT --target-os=linux --arch=x86 --cpu=i686 \
--enable-asm --enable-yasm --enable-pic --extra-cflags="-DANDROID -DNDEBUG -fPIC -pipe -mtune=atom -mssse3 -ffast-math -mfpmath=sse" \
--extra-ldflags="-lm -lz -Wl,--no-undefined -Wl,-z,noexecstack" --disable-stripping --disable-symver --disable-programs

make -j8
LIBS="libavcodec/libavcodec.so libavdevice/libavdevice.so libavfilter/libavfilter.so libavformat/libavformat.so libavutil/libavutil.so libpostproc/libpostproc.so libswresample/libswresample.so libswscale/libswscale.so"
$ANDROID_NDK/toolchains/x86-4.8/prebuilt/linux-x86_64/bin/i686-linux-android-strip $LIBS
mkdir -p com/googlecode/javacv/cpp/android-x86/
cp $LIBS com/googlecode/javacv/cpp/android-x86/
jar cvf ../ffmpeg-$FFMPEG_VERSION-android-x86.jar com/
rm -Rf com/
cd ../
```

Here **ffmpeg-$FFMPEG\_VERSION-android-x86.patch** is exactly the same as the arm one, you can cp it.

And for **build\_opencv-android-x86.sh**:

```bash
tar -xzvf opencv-$OPENCV_VERSION.tar.gz
mkdir opencv-$OPENCV_VERSION/build_android-x86
cd opencv-$OPENCV_VERSION
cd build_android-x86
ANDROID_BIN=$ANDROID_NDK/toolchains/x86-4.6/prebuilt/linux-x86_64/bin/ \
ANDROID_CPP=$ANDROID_NDK/sources/cxx-stl/gnu-libstdc++/4.6/ \
ANDROID_ROOT=$ANDROID_NDK/platforms/android-9/arch-x86/ \
cmake -DCMAKE_TOOLCHAIN_FILE=platforms/android/android.toolchain.cmake -DANDROID_ABI=x86 -DOPENCV_EXTRA_C_FLAGS="-O3 -ffast-math -mtune=atom -mssse3 -mfpmath=sse" -DOPENCV_EXTRA_CXX_FLAGS="-O3 -ffast-math -mtune=atom -mssse3 -mfpmath=sse" -DCMAKE_INSTALL_PREFIX=$ANDROID_NDK/../ -DBUILD_SHARED_LIBS=ON -DBUILD_TESTS=OFF -DBUILD_PERF_TESTS=OFF -DBUILD_ANDROID_EXAMPLES=OFF -DBUILD_JASPER=ON -DBUILD_JPEG=ON -DBUILD_OPENEXR=ON -DBUILD_PNG=ON -DBUILD_TBB=ON -DBUILD_TIFF=ON -DBUILD_ZLIB=ON -DBUILD_opencv_java=OFF -DBUILD_opencv_python=OFF -DENABLE_PRECOMPILED_HEADERS=OFF -DWITH_1394=OFF -DWITH_FFMPEG=OFF -DWITH_GSTREAMER=OFF -DWITH_TBB=ON -DWITH_CUDA=OFF -DWITH_OPENCL=OFF ..
make -j8
LIBS="lib/x86/libopencv*.so lib/x86/libtbb.so"
$ANDROID_NDK/toolchains/x86-4.6/prebuilt/linux-x86_64/bin/i686-linux-android-strip $LIBS
mkdir -p com/googlecode/javacv/cpp/android-x86/
cp $LIBS com/googlecode/javacv/cpp/android-x86/
jar cvf ../../opencv-$OPENCV_VERSION-android-x86.jar com/
rm -Rf com/
cd ../../
```

Here I have deleted the call to the -arm patch that was generating another android.mk file to directly rely on **platforms/android/android.toolchain.cmake** that is provided by **OpenCV** and I’ve also added specific flags to get more optimization:

```bash
-DANDROID_ABI=x86 -DOPENCV_EXTRA_C_FLAGS="-O3 -ffast-math -mtune=atom -mssse3 -mfpmath=sse" -DOPENCV_EXTRA_CXX_FLAGS="-O3 -ffast-math -mtune=atom -mssse3 -mfpmath=sse"
```

 Now if you call:

```shell
sh ./build_all.sh android-x86
cd ..
```

This will generate you **ffmpeg-2.1.1-android-x86.jar** and **opencv-2.4.8-android-x86.jar**.

Now you can finally build the JavaCV package:

## Packaging x86 version of JavaCV

First retrieve and install JavaCPP:

```shell
git clone https://code.google.com/p/javacpp/
cd javacpp/
git checkout 0.6
mvn install
cd ..
```

Once its done, retrieve JavaCV source code:

```shell
git clone https://code.google.com/p/javacv/
cd javacv
git checkout 0.7
```

 You can now package the android-x86 version of **JavaCV** by setting the **android-x86** property to **JavaCPP** a:

```shell
mvn package -Pffmpeg -Djavacpp.options="-properties android-x86 -Dplatform.root=$ANDROID_NDK -Dcompiler.path=$ANDROID_NDK/toolchains/x86-4.6/prebuilt/linux-x86_64/bin/i686-linux-android-g++ -Dcompiler.includepath=$ANDROID_NDK/sources/cxx-stl/gnu-libstdc++/4.6/include\
:$ANDROID_NDK/sources/cxx-stl/gnu-libstdc++/4.6/libs/x86/include\
:../javacv-cppjars/opencv-2.4.8/build_android-x86/\
:../javacv-cppjars/opencv-2.4.8/modules/core/include\
:../javacv-cppjars/opencv-2.4.8/modules/androidcamera/include\
:../javacv-cppjars/opencv-2.4.8/modules/flann/include\
:../javacv-cppjars/opencv-2.4.8/modules/imgproc/include\
:../javacv-cppjars/opencv-2.4.8/modules/highgui/include\
:../javacv-cppjars/opencv-2.4.8/modules/features2d/include\
:../javacv-cppjars/opencv-2.4.8/modules/calib3d/include\
:../javacv-cppjars/opencv-2.4.8/modules/ml/include\
:../javacv-cppjars/opencv-2.4.8/modules/video/include\
:../javacv-cppjars/opencv-2.4.8/modules/legacy/include\
:../javacv-cppjars/opencv-2.4.8/modules/objdetect/include\
:../javacv-cppjars/opencv-2.4.8/modules/photo/include\
:../javacv-cppjars/opencv-2.4.8/modules/gpu/include\
:../javacv-cppjars/opencv-2.4.8/modules/nonfree/include\
:../javacv-cppjars/opencv-2.4.8/modules/contrib/include\
:../javacv-cppjars/opencv-2.4.8/modules/stitching/include\
:../javacv-cppjars/opencv-2.4.8/modules/ts/include\
:../javacv-cppjars/opencv-2.4.8/modules/videostab/include\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86 \
-Dcompiler.linkpath=$ANDROID_NDK/sources/cxx-stl/gnu-libstdc++/4.6/libs/x86\
:../javacv-cppjars/opencv-2.4.8/build_android-x86/lib/x86\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libswscale\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libavcodec\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libswresample\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libpostproc\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libavfilter\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libavformat\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libavutil\
:../javacv-cppjars/ffmpeg-2.1.1-android-x86/libavdevice"
```

You’ll get your packages in **target/**:

```text
2.7M Jan 15 19:09 javacv-android-x86.jar
3.6M Jan 15 19:09 javacv-bin.zip
747K Jan 15 19:09 javacv-src.zip
734K Jan 15 19:09 javacv.jar
```

## Adding this x86 version to your app

From the packages I gave at the beginning of this article or from what you’ve just built, you can now copy the [generated .so files](/wp-content/uploads/2014/01/javacv-0.7-cppjars-android-x86.zip) that are inside **javacv-android-x86.jar**, **ffmpeg-2.1.1-android-x86.jar** and **opencv-2.4.8-android-x86.jar** into the **/lib/x86/** folder of your Android package, the same way as the arm versions inside **/lib/armeabi-v7a/**.

To get more information on how to deal with .so files and APKs, please refer to these two previous articles:

- <http://ph0b.com/improving-x86-support-of-android-apps-libs-engines/>
- <http://ph0b.com/android-studio-gradle-and-ndk-integration/>
