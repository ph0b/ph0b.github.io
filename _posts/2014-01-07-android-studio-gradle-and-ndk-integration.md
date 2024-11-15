---
id: 71
title: 'Android Studio, gradle and NDK integration'
date: '2014-01-07T19:15:08+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=71'
permalink: /android-studio-gradle-and-ndk-integration/
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
    - adt
    - Android
    - 'Android Build System'
    - 'Android Studio'
    - gradle
    - NDK
---

<span style="line-height: 1.5;">With the recent changes (release 0.7.3 around Dec 27), the new Android Build System starts to be really interesting also if you are using the NDK!</span>

Now this is really easy to integrate native libraries in your package and generate APKs for different architectures while correctly handling version codes (for more information on why this may be important, please refer to my [first article](http://ph0b.com/alljoyn-for-x86-android-platforms/ "x86 version of AllJoyn for Android platforms")).

<span style="text-decoration: underline;"><span style="text-decoration: underline;">update 2015/08/10:</span></span> this article remains fully up-to-date if you’re using the recent Android Studio and the stable Android gradle plugin (set *android.useDeprecatedNdk=true* inside *gradle.properties*). Another article on gradle-experimental is in the work!

<span style="text-decoration: underline;">update 2014/11/1:</span><span style="line-height: 1.5;"> changed output.abiFilter to output.getFilter(com.android.build.OutputFile.ABI) in order to work with gradle 0.14</span>

<span style="text-decoration: underline;">update 2014/09/19:</span> I’ve added information and modified my sample [build.gradle](#mygradlefile) to demonstrate the brand new **APK Splits** feature introduced by the latest android gradle plugin (**0.13**)

<span style="text-decoration: underline;">update 2:</span> I’ve modified the sample [build.gradle](#mygradlefile) file to make it call **ndk-build** scripts by itself.

<span style="text-decoration: underline;">update 1:</span> Here is a screencast on how to set up a project with **NDK sources** from Android Studio:

<iframe allowfullscreen="allowfullscreen" frameborder="0" height="390" loading="lazy" src="//www.youtube.com/embed/okLKfxfbz40/?vq=hd720" width="640"></iframe>

<span style="line-height: 1.5; font-size: 26px; font-weight: bold;">Integrating .so files into your APK</span>

If you are using Android Studio and need to integrate native libraries in your app, you may have had to use some complex methods before, involving maven and .aar/.jar packages… the good news is you don’t need these anymore 🙂

[![jniLibs](http://ph0b.com/wp-content/uploads/2014/01/jniLibs.png)](http://ph0b.com/wp-content/uploads/2014/01/jniLibs.png)

You only need to put your .so libraries inside the **jniLibs** folder under sub-directories named against each supported ABI (x86, mips, armeabi-v7a, armeabi), and that’s it !

Once it’s done, all the .so files will be integrated into your apk when you build it:

[![fatbinary](http://ph0b.com/wp-content/uploads/2014/01/fatbinary.png)](http://ph0b.com/wp-content/uploads/2014/01/fatbinary.png)

If the **jniLibs** folder name doesn’t suit you (you may generate your .so files somewhere else), you can set a specific location in **build.gradle**:

```
android {
    ...
    sourceSets.main {
        jniLibs.srcDir 'src/main/libs'
    }
}
```

# Building one APK per architecture, and doing it well !

You can use flavors to build one APK per architecture really easily, by using **abiFilter** property.

**ndk.abiFilter(s)** is by default set to **all**. This property has an impact on the integration of .so files as well as the calls to ndk-build (I’ll talk about it at the end of this article).

Let’s add some architecture flavors in **build.gradle**:

```
android{
  ...
  productFlavors {
        x86 {
            ndk {
                abiFilter "x86"
            }
        }
        mips {
            ndk {
                abiFilter "mips"
            }
        }
        armv7 {
            ndk {
                abiFilter "armeabi-v7a"
            }
        }
        arm {
            ndk {
                abiFilter "armeabi"
            }
        }
        fat
    }
}
```

And then, sync your project with gradle files:

[![sync](http://ph0b.com/wp-content/uploads/2014/01/sync.png)](http://ph0b.com/wp-content/uploads/2014/01/sync.png)

<span style="line-height: 1.5;">You should now be able to enjoy these new flavors by selecting the build variants you want:</span>

[![buildVariants](http://ph0b.com/wp-content/uploads/2014/01/buildVariants.png)](http://ph0b.com/wp-content/uploads/2014/01/buildVariants.png)

<span style="line-height: 1.5;">Each of these variants will give you an APK for the designated architecture:</span>

<figure aria-describedby="caption-attachment-87" class="wp-caption aligncenter" id="attachment_87" style="width: 580px">[![thinbinary](http://ph0b.com/wp-content/uploads/2014/01/thinbinary.png "app-x86-release-unsigned.apk")](http://ph0b.com/wp-content/uploads/2014/01/thinbinary.png)<figcaption class="wp-caption-text" id="caption-attachment-87">app-x86-release-unsigned.apk</figcaption></figure><span style="line-height: 1.5;">The fat(Release|Debug) one will still contain all the libs, like the standard package from the beginning of this blog post.</span>

*But don’t stop reading here! T*<span style="line-height: 1.5;">hese arch-dependent APKs are useful when developing, but if you want to upload several of these to the Google Play Store, you have to set a different </span>**versionCode**<span style="line-height: 1.5;"> for each. </span>And thanks to the latest android build system, this is really easy:

# Automatically setting different version codes for ABI dependent APKs

The property **android.defaultConfig.versionCode** holds the **versionCode** for your app. By default it’s set to **-1** and if you don’t change it, the **versionCode** set in your **AndroidManifest.xml** file will be used instead.

Hence if you want to be able to dynamically modify your **versionCode**, you need to first specify it inside your **build.gradle**:

```
android {
    ...
    defaultConfig{
        versionName "1.1.0"
        versionCode 110
    }
}
```

But this is still possible to keep setting this variable only inside your **AndroidManifest.xml** if you retrieve it “manually” before modifying it:

```
import java.util.regex.Pattern

android {
    ...
    defaultConfig{
        versionCode getVersionCodeFromManifest()
    }
    ...
}

def getVersionCodeFromManifest() {
    def manifestFile = file(android.sourceSets.main.manifest.srcFile)
    def pattern = Pattern.compile("versionCode=\"(\\d+)\"")
    def matcher = pattern.matcher(manifestFile.getText())
    matcher.find()
    return Integer.parseInt(matcher.group(1))
}
```

Once it’s done, you can prefix the **versionCode** inside your different flavors:

```
android {
    ...
    productFlavors {
        x86 {
            versionCode Integer.parseInt("6" + defaultConfig.versionCode)
            ndk {
                abiFilter "x86"
            }
        }
        mips {
            versionCode Integer.parseInt("4" + defaultConfig.versionCode)
            ndk {
                abiFilter "mips"
            }
        }
        armv7 {
            versionCode Integer.parseInt("2" + defaultConfig.versionCode)
            ndk {
                abiFilter "armeabi-v7a"
            }
        }
        arm {
            versionCode Integer.parseInt("1" + defaultConfig.versionCode)
            ndk {
                abiFilter "armeabi"
            }
        }
        fat
    }
}
```

Here I’ve prefixed it with 6 for x86, 4 for mips, 2 for ARMv7 and 1 for ARMv5. If you’re asking yourself ***why!?*** please refer to this paragraph I wrote before on [architecture dependent APKs on the Play Store](http://ph0b.com/improving-x86-support-of-android-apps-libs-engines/#ArchDependentAPKs).

# Improving multiple APKs creation and versionCode handling with *APK Splits*

Since version 0.13 of the android plugin, instead of having a product flavors to get multiple APKs, you can use splits to have a single build (and variant) that will produce multiple APKs (and it’s much cleaner and faster).

```
    splits {
        abi {
            enable true // enable ABI split feature to create one APK per ABI
            universalApk true //generate an additional APK that targets all the ABIs
        }
    }
    // map for the version code
    project.ext.versionCodes = ['armeabi':1, 'armeabi-v7a':2, 'arm64-v8a':3, 'mips':5, 'mips64':6, 'x86':8, 'x86_64':9]

    android.applicationVariants.all { variant ->
        // assign different version code for each output
        variant.outputs.each { output ->
            output.versionCodeOverride =
                    project.ext.versionCodes.get(output.getFilter(com.android.build.OutputFile.ABI), 0) * 1000000 + android.defaultConfig.versionCode
        }
    }
```

# <span style="font-size: 26px; font-weight: bold; line-height: 1.3846153846;">Compiling your C/C++ source code from Android Studio</span>

<span style="line-height: 1.5;">If you have a </span>**jni/**<span style="line-height: 1.5;"> folder in your project sources, the build system will try to call </span>**ndk-build**<span style="line-height: 1.5;"> automatically.</span>

<del>As of 0.7.3, this integration is only working on Unix-compatible systems, cf [bug 63896](https://code.google.com/p/android/issues/detail?id=63896). On Windows you’ll want to disable it so you can call **ndk-build.cmd** yourself. You can do so by setting this in **build.gradle**:</del> this has been fixed 🙂

The current implementation is ignoring your Android.mk makefiles and create a new one on the fly. While it’s really convenient for simple projects (you don’t need **\*.mk** files anymore !), it may be more annoying for projects where you need all the features offered by Makefiles. You can then disable this properly in build.gradle:

```
android{
    ...
    sourceSets.main.jni.srcDirs = [] //disable automatic ndk-build call
}
```

<span style="line-height: 1.5;">If you want to use the on-the-fly generated Makefile, you can configure it first by setting the </span>**ndk**<span style="line-height: 1.5;">.</span>**moduleName**<span style="line-height: 1.5;"> property, like so:</span>

```
android {
 ...
 defaultConfig {
        ndk {
            moduleName "hello-jni"
        }
    }
}
```

 And y<span style="line-height: 1.5;">ou’re still able to set these other ndk properties:</span>

- ****cFlags****
- **ldLibs**
- **stl** (ie: **gnustl\_shared**, **stlport\_static**…)
- **abiFilters** (ie: **“x86”, “armeabi-v7a”**)

You can also set **android.buildTypes.debug.jniDebuggable** to true so it will pass **NDK\_DEBUG=1** to **ndk-build** when generating a debug APK.

If you are using RenderScript from the NDK, you’ll need also to set the specific property **<span style="line-height: 1.5;">defaultConfig.</span>**<span style="line-height: 1.5;">**renderscriptNdkMode** to *true*.</span>

If you rely on auto-generate Makefiles, you can’t easily set different **cFlags** depending on the target architecture when you’re building multi-arch APKs. So if you want to entirely rely on gradle I recommend you to generate different libs per architecture by using flavors like I’ve described earlier in this post:

```
  ...
  productFlavors {
    x86 {
        versionCode Integer.parseInt("6" + defaultConfig.versionCode)
        ndk {
            cFlags cFlags + " -mtune=atom -mssse3 -mfpmath=sse"
            abiFilter "x86"
        }
    }
    ...
```

<a name="mygradlefile"></a>

# My sample .gradle file

Putting this altogether, Here is one **build.gradle** file I’m curently using. It’s using APK Splits to generate multiple APKs, it doesn’t use ndk-build integration to still rely on Android.mk and Application.mk files, and doesn’t require changing the usual location of sources and libs (sources in **jni/**, libs in **libs/**). It’s also automatically calling ndk-build script from the right directory:

```
import org.apache.tools.ant.taskdefs.condition.Os

apply plugin: 'com.android.application'

android {
    compileSdkVersion 21
    buildToolsVersion "21.1"

    defaultConfig{
        minSdkVersion 16
        targetSdkVersion 21
        versionCode 101
        versionName "1.0.1"
    }

    sourceSets.main {
        jniLibs.srcDir 'src/main/libs'
        jni.srcDirs = [] //disable automatic ndk-build call
    }

   project.ext.versionCodes = ['armeabi':1, 'armeabi-v7a':2, 'arm64-v8a':3, 'mips':5, 'mips64':6, 'x86':8, 'x86_64':9] //versionCode digit for each supported ABI, with 64bit>32bit and x86>armeabi-*

    android.applicationVariants.all { variant ->
        // assign different version code for each output
        variant.outputs.each { output ->
            output.versionCodeOverride =
 project.ext.versionCodes.get(output.getFilter(com.android.build.OutputFile.ABI), 0) * 1000000 + defaultConfig.versionCode
        }
    }

    // call regular ndk-build(.cmd) script from app directory
    task ndkBuild(type: Exec) {
        if (Os.isFamily(Os.FAMILY_WINDOWS)) {
            commandLine 'ndk-build.cmd', '-C', file('src/main').absolutePath
        } else {
            commandLine 'ndk-build', '-C', file('src/main').absolutePath
        }
    }

    tasks.withType(JavaCompile) {
        compileTask -> compileTask.dependsOn ndkBuild
    }
}
```

# <span style="line-height: 1.5; font-size: 26px; font-weight: bold;">Troubleshooting</span>

## <span style="line-height: 1; text-align: justify;">NDK not configured</span>

If you get this kind of error:

```
Execution failed for task ':app:compileX86ReleaseNdk'.
> NDK not configured
```

This means the tools haven’t found the NDK directory. You have two ways to fix it: set the **ANDROID\_NDK\_HOME** variable environment to your NDK directory and delete **local.properties**, or set it manually inside **local.properties**:

```
ndk.dir=C\:\\Android\\ndk
```

## No rule to make target

If you get this kind of error:

```
make.exe: *** No rule to make target ...\src\main\jni
```

This may come from a current [NDK bug](https://code.google.com/p/android/issues/detail?id=66937) on Windows, when there is only one source file to compile. You only need to add one empty source to make it work again.

## <span style="line-height: 1.5;">Other issues</span>

<span style="line-height: 1.5;">You may also find some help on the official adt-dev google group: <https://groups.google.com/forum/#!forum/adt-dev> </span>

# Getting more information on NDK integration

The best place to get more information is the official project page: <http://tools.android.com/tech-docs/new-build-system>.

You can look at the changelog and if you scroll all the way down you’ll also get access to sample projects dealing with NDK integration, inside the latest “gradle-samples-XXX.zip” archive.