---
id: 236
title: 'The new NDK support in Android Studio'
date: '2015-08-17T18:50:08+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=236'
permalink: /new-android-studio-ndk-support/
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
    - 'Android Build System'
    - 'Android Studio'
    - gradle
    - NDK
---

During Google I/O 2015, end of May, Google announced a new support for the NDK by Android Studio 1.3, integrating Jetbrains CLion capabilities, and the Android gradle plugin. This support has been released only in July, and while it’s really promising, it’s still in heavy development.

The new NDK support requires the use of Android Studio 1.3 RC1+/2.0+ and the android gradle-experimental plugin. This article is for those who are willing to give it a try. If you’re looking into the NDK support while using the gradle(-stable) plugin, you can check this older (but still up-to-date) [article on the NDK and Android Studio](http://ph0b.com/android-studio-gradle-and-ndk-integration/).

**last update – 2016/09/20: Android Studio 2.2 now supports external CMake and ndk-build tools very well. It’s in my view much better than the gradle-experimental plugin explained here. Please follow this guide to learn how to use it: <https://developer.android.com/studio/projects/add-native-code.html>**

## Migrating to the gradle-experimental plugin and the new com.android.model.\*

The gradle-experimental 0.7.2 requires using gradle-2.10. Start by setting it from your project settings:

[![gradle-2-5](http://ph0b.com/wp-content/uploads/2015/08/gradle-2-5.png)](http://ph0b.com/wp-content/uploads/2015/08/gradle-2-5.png)

Or inside *gradle/wrapper/gradle-wrapper.properties:*

```
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
```

Then, change the reference to the android gradle plugin to the new gradle-experimental plugin, from *./build.gradle*:

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle-experimental:0.7.2'
    }
}
```

The gradle-experimental plugin introduces a change in the DSL. The android plugins *com.android.model.application* and *com.android.model.library* are replacing the former *com.android.applicatio*n and *com.android.library* plugins.

You need to migrate your apps and libs *build.gradle* files to use these new plugins. Here is an example of the same configuration, with the old DSL (top) and the new (bottom):

```
apply plugin: 'com.android.application'

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        applicationId "com.ph0b.example"
        minSdkVersion 15
        targetSdkVersion 23
        versionCode 4
        versionName "1.0.1"

        ndk {
            moduleName "mymodule"
            ldLibs "log"
            stl "gnustl_static"
            cFlags "-std=c++11 -fexceptions"
        }
    }

    signingConfigs {
        release {
            storeFile file(STORE_FILE)
            storePassword STORE_PASSWORD
            keyAlias KEY_ALIAS
            keyPassword KEY_PASSWORD
        }
    }

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.txt'
            signingConfig signingConfigs.release
        }
        debug {
            jniDebuggable true
        }
    }
}
```

```
apply plugin: 'com.android.model.application'

model {
    android {
        compileSdkVersion rootProject.ext.compileSdkVersion
        buildToolsVersion rootProject.ext.buildToolsVersion
    
        defaultConfig {
            applicationId "com.ph0b.example"
            minSdkVersion.apiLevel 15
            targetSdkVersion.apiLevel 23
            versionCode 4
            versionName "1.0.1"
        }
    }

    android.ndk {
        moduleName "mymodule"
        ldLibs.addAll(['log'])
        cppFlags.add("-std=c++11")
        cppFlags.add("-fexceptions")
		platformVersion 15
        stl 'gnustl_shared'
    }

    android.signingConfigs {
        create("release") {
            keyAlias KEY_ALIAS
            keyPassword STORE_PASSWORD
            storeFile file(STORE_FILE)
            storePassword KEY_PASSWORD
        }
    }

    android.buildTypes {
        release {
            shrinkResources true
            useProguard true
            proguardFiles.add(file('proguard-rules.txt'))
            signingConfig = $("android.signingConfigs.release")
        }
    }
}
```

To summarize the changes required: all the android declarations are now going under *model {},* <del>the various assignments now have to use explicitly ‘*=*‘</del>*,* collections must not be overwritten, use *.removeAll()*, *.add()*, *.addAll()* instead. Variants and other new configurations have to be declared using ‘*create()*‘. Properties like *xxxSdkVersion* have changed to *xxxSdkVersion.apiLevel*.

As it’s experimental, you can expect regular changes around the DSL across versions. For example, *minifyEnabled* has been changed to *isMinifyEnabled,* then to *minifyEnabled* again, and now there is also *useProguard;* *jniDebuggable* has been changed to *isJniDebuggable* and then to *ndk.debuggable* (and is now set by default for debug builds).

You’ll notice that with both DSLs, there is a configuration block for the NDK. This is the place where you’re supposed to set all the NDK configuration when using gradle, as by default *Android.mk* and *Application.mk* files will be simply ignored.

## Enjoying the new C++/NDK support in Android Studio

To activate the C++/NDK support inside Android Studio, you only need to have a NDK module declared inside your application or library build.gradle:

```
model {
    android {
    //...
    ndk {
        moduleName "mymodule"
    }
  }
}
```

Once it’s done, you can go to your Java sources, create a method prefixed with the *native* keyword, and press ALT+Enter to generate its C or C++ implementation:

[![AS-ndk-demo](http://ph0b.com/wp-content/uploads/2015/08/AS-ndk-demo1.gif)](http://ph0b.com/wp-content/uploads/2015/08/AS-ndk-demo1.gif)

Yes, it’s that magical 🙂

The implementation will be added under ‘*jni*‘, inside an existing cpp file if there is one, or inside a new one.

In order to get started with NDK modules, you can have a look at all the samples that have been ported to use the new gradle-experimental plugin: <https://github.com/googlesamples/android-ndk>

Here is everything you can configure for a ndk module:

```
    android.ndk {
        moduleName = "mymodule"
        ldLibs.addAll(['log'])
        ldFlags.add("")
        toolchain = "clang"
        toolchainVersion = "3.9"
        abiFilters.add("x86")
        CFlags.add("")
        cppFlags.add("")
        debuggable = false
        renderscriptNdkMode = false
        stl = "system"
        platformVersion = 15
    }
```

Since 0.7.0, you can also add ABI-specific configurations:

```
	android.abis {
		create("x86") {
			cppFlags.add('-DENABLE_SSSE3')
			ldLibs.add('')
			ldFlags('')
		}
		create("armeabi-v7a") {
			cppFlags.addAll(["-mhard-float", "-D_NDK_MATH_NO_SOFTFP=1", "-mfloat-abi=hard"])
			ldLibs.add("m_hard")
			ldFlags.add("-Wl,--no-warn-mismatch")
		}
	}
```

## Debugging a NDK project

In order to get the debug capabilities of AS, create and use a new Run/Debug configuration from the “Android Native” default. While it was possible to use GDB in Android Studio 1.3, now only the LLDB backend is available.

[![native-debug-config](http://ph0b.com/wp-content/uploads/2015/08/native-debug-config.png)](http://ph0b.com/wp-content/uploads/2015/08/native-debug-config.png)

Use it with your debug variant, which will have *ndk.debuggable* flag set to *true* by default.

## Going further with the NDK with Android Studio

Many advanced features, such as the ability to have dependencies between native libraries, reuse prebuilts, tune specific toolchain options and having dynamic version codes while still having a project in a good shape is a bit complex, as the gradle-experimental plugin is still undergoing a lot of improvements across versions.

### Getting the APP\_PLATFORM right

When you’re building a NDK module, the android platform you’re compiling it against is a quite important setting, as it basically determines the minimum platform your module will be guaranteed to run on.

With earlier versions than *gradle-experimental:0.3.0-alpha4*, the chosen platform was the one set as *compileSdkVersion*. Fortunately with subsequent releases, you can now set *android.ndk.platformVersion* independently, and you should make it the same as your minSdkVersion.

### Using external libraries and separate modules

#### with sources

If you have access to your 3rd party libraries source code, you can embed it into your project and make it statically compile with your code.

There is an example of this with the native\_app\_glue library from the NDK, inside the [native-activity sample](https://github.com/googlesamples/android-ndk/blob/229cbe86238d401bb06166b8dfadec8198532589/native-activity/app/build.gradle#L22). For example, you can copy the library sources inside a subfolder inside your *jni* folder and add a reference to its directory so the includes are properly resolved:

```
    android.ndk {
        //...
        cppFlags.add('-I' + file("src/main/jni/native_app_glue"))
    }
```

#### with sources in different modules

Now from 0.6.0-alpha7 version, you can finally have clean dependencies between native libraries, by setting the dependency on another module from your model:

```
    android.sources {
        main {
            jni {
                dependencies {
                    project ":yourlib" buildType "release" productFlavor "flavor1" linkage "shared"
                }
            }
        }
    }
```

In order to keep debugging working, you may have to edit your app-native run configuration, to add **/build/intermediates/binaries/release/obj/\[abi\]** to the symbol directories.

#### with native prebuilts

This technique works with static and shared prebuilts too! Inside your model, you’ll have to add a “lib repository”:

```
    repositories {
        libs(PrebuiltLibraries) {
            yourlib {
                headers.srcDir "src/main/jni/prebuilts/include"
                binaries.withType(SharedLibraryBinary) {
                    sharedLibraryFile = file("src/main/jni/prebuilts/${targetPlatform.getName()}/libyourlib.so")
                }
            }
        }
    }
```

And declare the dependency on this library:

```
    android.sources {
        main {
            jni {
                dependencies {
                    library "yourlib" linkage "shared" 
                }
            }
        }
    }
```

Shared linkage is the default, but of course you can use static prebuilts by using a static linkage, and declaring StaticLibraryBinary/staticLibraryFile variables.

When having dependencies on shared libraries, you need to make sure to integrate these libs inside your APK. It will be the case if they’re under *jniLibs*, otherwise you can add them manually:

```
    android.sources {
        main {
            jniLibs {
                source {
                    srcDir "src/main/jni/prebuilts"
                }
            }
        }
    }
```

### Multiple APKs

When you publish multiple APKs (per architecture, per density, etc), you have to give them different version Version Codes. I couldn’t find a way to do it with the current DSL, neither when using splits or abiFilters and flavors. If you do see a way, plese contact me and I’ll add it there. For now, I would advise to go back using the [stable gradle plugin](http://ph0b.com/android-studio-gradle-and-ndk-integration/).

The good news is you can mix use of the stable gradle plugin, and the new experimental one, even while keeping debug features working! Please follow [this gist](https://gist.github.com/ph0b/0575b30b67e04f2ec10f).

<a name="usingndkbuild"></a>

### Using Android.mk/Application.mk

If the built-in gradle support isn’t suitable to your needs, you can get rid of it, while keeping the goodness of Android Studio C++ editing.

Declare a module that correctly represents your configuration, as this will help AS to correctly resolve all the symbols you’re using and keep the editing capabilities:

```
android.ndk { // keeping it to make AS correctly support C++ code editing
        moduleName "mymodule"
        ldLibs.add('log')
        cppFlags.add('-std=c++11')
        cppFlags.add('-fexceptions')
        cppFlags.add('-I' + file("src/main/jni/prebuilts/include"))
        stl = 'gnustl_shared'
    }
```

Then, set the *jniLibs* location to *libs*, the default directory in which *ndk-build* will put the generated libs, and deactivate built-in compilation tasks:

```
model {
    //...
    android.sources {
        main {
          jniLibs {
            source {
                srcDir 'src/main/libs'
            }
        }
    }
}

tasks.all {
	task ->
	if (task.name.startsWith('compile') && task.name.contains('MainC')) {
		task.enabled = false
	}
	if (task.name.startsWith('link')) {
		task.enabled = false
	}
}
```

*Thanks to [Alex Cohn](http://stackoverflow.com/questions/32594682/how-do-we-define-multiple-modules-with-the-new-ndk-gradle-based-builder/32640943#32640943) and <span class="lG">Vitaly</span> Dyadyuk who shared this technique to disable compilation tasks.*

This way, you can call *ndk-build(.cmd)* yourself from the root of your src/main directory. ndk-build will use your usual *Android.mk*/*Application.mk* files under the *jni* folder, your libs will be generated inside *libs/&lt;ABI&gt;* as usual and get included inside your APK.

You can also add the call to *ndk-build* in your gradle configuration so it’s done automatically:

```
import org.apache.tools.ant.taskdefs.condition.Os

model {
    //...
    android.sources {
        main {
          jniLibs {
            source {
                srcDir 'src/main/libs'
            }
        }
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

tasks.all {
	task ->
	if (task.name.startsWith('compile') && task.name.contains('MainC')) {
		task.enabled = false
	}
	if (task.name.startsWith('link')) {
		task.enabled = false
	}
	if (task.name.endsWith('SharedLibrary') ) {
		task.dependsOn ndkBuild
	}
}
```

## Additional Resources

- [official samples](https://github.com/googlesamples/android-ndk)
- [gradle-experimental user guide](http://tools.android.com/tech-docs/new-build-system/gradle-experimental)
- Important bugs that may hit you: 
    - [Android Studio cannot find jni.h, but compilation still works](https://code.google.com/p/android/issues/detail?id=195483) (ETA: Android Studio 2.2)
    - [ ndk.platformVersion &lt; 21 prevents generating 64-bit libs](https://code.google.com/p/android/issues/detail?id=195135) (ETA: Android Studio 2.2)
    - [can’t dynamically modify APKs versionCodes with gradle-experimental plugin](https://code.google.com/p/android/issues/detail?id=192943) (No ETA)
    - [adding lint checks for incoherent inclusion of .so files](https://code.google.com/p/android/issues/detail?id=94805) (No ETA)