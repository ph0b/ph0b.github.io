---
id: 164
title: 'Porting an Android app to Android TV and the Nexus Player'
date: '2014-11-03T19:00:22+01:00'
author: 'Xavier Hallade'
layout: post
guid: 'http://ph0b.com/?p=164'
permalink: /porting-android-app-android-tv-nexus-player/
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
    - 'Android TV'
    - 'Nexus Player'
    - porting
    - x86
---

The **Nexus Player** is the **Android TV** **Nexus** device from Google.

[![](https://ph0b.com/wp-content/uploads/2014/11/player-overview-1600.jpg)](//ph0b.com/wp-content/uploads/2014/11/player-overview-1600.jpg)

This device, made by Asus embeds 8GB memory, 1GB of ram, costs **99$** and features an **Intel**® SOC, the Atom Z3560 (codenamed Moorefield). That means it has got a **4-cores** **64-bit** Silvermont CPU clocked at **1.83 Ghz** and a **PowerVR** Series 6 GPU (**G6430**) that supports **OpenGL ES 3.1**.

Let’s look at how you can port your app to this device and Android TV in general.

## Android TV requirements for applications

Simply said, If you want your application to be available for Android TV devices and visible from their launcher, you need to provide an activity that will handle the *action.Main* intent for the *LEANBACK\_LAUNCHER* category, to integrate TV-specific assets, and to specify that your application doesn’t require a touchscreen.

These are the very basic requirements for an application to be installed and used on Android TV devices like the Nexus Player. Let’s look at how this works in practice and then how to adapt your application so it provides a good user experience on Android TV and the Nexus Player:

### Declaring an Activity for the Leanback Launcher Main intent

Like a classic Android Launcher, the Android TV Launcher will look for the activity of your application that you declared as a target of the *android.intent.action.MAIN* intent.

But in the case of the Android TV Launcher, the category of the intent isn’t *android.intent.category.LAUNCHER* but ***android.intent.category.LEANBACK\_LAUNCHER***.

Here is how you should declare the Main Activity of your application for Android TV:

```
<activity
            android:name=".TvMainActivity"
            android:banner="@drawable/ic_banner"
            android:theme="@style/TvAppTheme">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LEANBACK_LAUNCHER" />
            </intent-filter>
</activity>
```

<span style="text-decoration: underline;">Note:</span> There are two categories of apps inside the Leanback Launcher -&gt; Apps and Games. If you want to appear among the Games, you need to set the *isGame* property of your application to true:

```
<application>
  ...
  android:isGame="true"
  ...>
```

On top of the specific intent filter, that’s also in this activity *android:banner* property that you specify your application’s banner that will be displayed on the Launcher. You can set this banner inside your &lt;activity&gt; or inside your &lt;application&gt;

### The banner

The logo of your application that will be displayed on the launcher is the *android:banner* associated to your application or to the activity that accepts the Leanback Launcher intent. Here it’s the drawable we called ic\_banner.

It has to be a 320x180px picture that you have to put inside your *drawable-xhdpi* resource folder. This pictures has to include the Name of your application, localized if needed, and have no alpha channel:

[![videos_by_google_banner](//ph0b.com/wp-content/uploads/2014/11/videos_by_google_banner.png)](//ph0b.com/wp-content/uploads/2014/11/videos_by_google_banner.png)

All this will allow your application to be seen and started from the Android TV launcher.

But For your application to be compatible with Android TV, you also need to support non-touchscreen Android devices and declare it:

### Supporting non-touchscreen Android devices

First, you need to implement support for D-pad navigation. Standard Android UI elements all support it, but in practice some simple adjustments may be needed:

#### Adjusting Initial Focus

Adjust the element that will have the initial focus inside your view using *requestFocus()* from Java or *&lt;requestFocus /&gt;* from your xml layouts:

```
<View android:id="@+id/my_view" android:focusable="true" >
      <requestFocus />
</View>
```

The *android:focusable* property may need to be set to true if you want the focus request to work when your view is inflated.

#### Adjusting navigation flow

Sometimes you may need to adjust the navigation within your view. You can choose where the focus is going from one element to another using *nextFocus* properties:

```
<View 
    android:nextFocusDown="@id/bottom_view"
    android:nextFocusUp="@id/top_view"
    android:nextFocusLeft="@id/left_view"
    android:nextFocusRight="@id/right_view"
/>
```

#### Declaring non-touchscreen devices support

Once D-pad navigation is working for you app, you can declare it inside your AndroidManifest.xml. Simply declare that the “touchscreen” feature isn’t required:

```
<uses-feature
    android:name="android.hardware.touchscreen"
    android:required="false" />
```

## Shaping a UI for Android TV

While the previous requirements may be enough for your app to be capable of being installed and appear on the Android TV launcher, there are more steps to follow to provide the right experience.

The theme of your activity has to be one with no ActionBar/Toolbar. (Even if the ActionBar can potentially work with D-pad, that’s a really bad experience to give to your users).

Two potential themes you can use are Theme.Leanback from the Leanback support library and android:Theme.NoTitleBar.

These themes don’t integrate overscan margins themselves, hence you may need to add some to your activities. Note that Leanback elements already include margins for proper overscan handling.

```
<LinearLayout
...
    android:layout_marginTop="27dp"
    android:layout_marginLeft="48dp"
    android:layout_marginRight="48dp"
    android:layout_marginBottom="27dp"
...>
```

### Using the Leanback support library

The Leanback support library is available from API Level 17 and provides ready-to-use elements and a theme for your app.

If you want to continue supporting lower API levels from your application, you can keep all the references to the Leanback theme and elements inside the TV-specific part of your app and resources-v21, then use gradle’s manifest merger feature to avoid your minSdkVersion to be upgraded:

```
<manifest 
    xmlns:tools="http://schemas.android.com/tools"
    ...>

    <uses-sdk tools:overrideLibrary="android.support.v17.leanback" />
   ...
```

You can start looking at the [sample ](https://github.com/googlesamples/androidtv-Leanback/)to see how to use the Leanback support library. I also recommend reading this good [introductory article](https://medium.com/building-for-android-tv/building-for-android-tv-episode-1-2d03f9ba541e) from Sebastiano Gottardo (musixMatch).

## Going further with Games

Android TV games can really be improved by adding a good multiplayer experience and Gamepad support.

From your manifest, you can specify that you support gamepads:

```
<uses-feature 
    android:name="android.hardware.gamepad"
    android:required="false" />
```

Set the feature as required only if gamepads are mandatory for your app, but remember that devices like the Nexus Player aren’t bundled with one. Additional Android devices can be used as controllers, but they only feature D-pad input as of now.

To support multiple user inputs, you can refer to this [training](http://developer.android.com/training/game-controllers/index.html).

## Integrating with the Recommendations system

The Recommendations system occupies half of the space of the Launcher, to appear inside it, send a notification with “recommendation” set as category, as described in the [official training](https://developer.android.com/training/tv/discovery/recommendations.html).

## Having a specific APK for Android TV

Through the *features* filtering functionality of the Play Store, you can restrict your APK for distribution to Android TV compatible devices, by setting the feature *android.software.leanback* as required:

```
<uses-feature android:name="android.software.leanback"
        android:required="true" />
```

When you require the leanback feature, your APK will be available only on Android-TV compatible devices.

You can choose to distribute this APK as a specific application on the Play Store, but you also have the choice to upload it as a TV-version of an already existing application, through the “[Multiple APKs](developer.android.com/google/play/publishing/multiple-apks.html)” feature of the Play Store.

You simply have to upload this TV-specific apk that requires the leanback feature and has a different versionCode than your classic application.

## Optimizing your Application for the Nexus Player

While the Player is a fully 64-bit capable platform and the kernel is indeed 64-bit, all the userspace remains x86 32-bit as of now.

The device is able to run ARM-specific binaries perfectly fine, but providing x86 binaries is always better. You can read more about compatibility considerations for x86 devices from my previous article: [How to improve x86 support of Android apps, libs, engines](//ph0b.com/improving-x86-support-of-android-apps-libs-engines/ "How to improve x86 support of Android apps, libs, engines")

## Last things to check before uploading your app

A recapitulation of the guidelines for Android TV can be found on the Android developer website: [Tv App Quality](https://developer.android.com/distribute/essentials/quality/tv.html).

If you have *android:screenOrientation=”portrait”* added to any activity of your application, the *android.hardware.screen.portrait* requirement is implicitly set to true.

You need to explicitly set it to false inside your manifest if you want your application to be available on Android TV devices:

```
<uses-feature
    android:name="android.hardware.screen.portrait"
    android:required="false" />
```

The same is needed for:

- *<span class="pln">android</span><span class="pun">.</span><span class="pln">hardware</span><span class="pun">.</span><span class="pln">location</span><span class="pun">.</span>android.permission.ACCESS\_FINE\_LOCATION*
- *android.hardware.camera.autofocus* and *android.hardware.camera* implied by *android.permission.CAMERA*
- *android.hardware.microphone* implied by *android.permission.RECORD\_AUDIO*
- *android.hardware.telephony* implied by many telephony-specific permissions

## Submitting your application

Once your application is ready and you’ve uploaded your APK to the Play Store developer console, you still need to add a screenshot as well as your banner:

| ![distribute-tv2](//ph0b.com/wp-content/uploads/2014/11/distribute-tv2.png) | ![banner](//ph0b.com/wp-content/uploads/2014/11/banner.png) |
|---|---|

Then tick the proper checkbox to ask for distribution on Android TV:

[![distribute-tv](//ph0b.com/wp-content/uploads/2014/11/distribute-tv.png)](//ph0b.com/wp-content/uploads/2014/11/distribute-tv.png)It will be manually reviewed before being available on the Play Store browsed from Android TV devices.