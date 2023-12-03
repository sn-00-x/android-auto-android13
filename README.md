## Android Auto as user app with media apps support on GrapheneOS / Android 13

This repository provides everything that is needed to add Android Auto support (with media apps support) to GrapheneOS (Android 13). However, the patch itself does not rely on GrapheneOS, so it should be possible to make this work on a different OS with slight modifications. **If you are looking for an Android 14 patch, see [here](https://github.com/sn-00-x/android-auto).**

The provided [patch](https://raw.githubusercontent.com/sn-00-x/android-auto-android13/main/frameworks-base.patch) includes compatibility changes and makes it possible to e.g. watch Netflix on your car screen with the help of Screen2Auto while restricting it's permissions when not connected to a head unit.
Furthermore the repo includes optional custom built Google Search App and Google Speech Services stubs.

To make this run, you need to build your own rom. If you are instead looking for a simpler method to run Android Auto (that involves rooting your device), have a look at [aa4mg](https://github.com/sn-00-x/aa4mg)

## Build

- Set up your build environment according to the official [GrapheneOS build instructions](https://grapheneos.org/build), sync sources (the patch is based on [this release](https://grapheneos.org/releases#2023080800)) and proprietary files.
- clone this repo into vendor/android-auto:
```
cd vendor
git clone https://github.com/sn-00-x/android-auto-android13.git
```
- Apply the patch in root directory:
```
cd ..
patch -p1 < vendor/android-auto/frameworks-base.patch
```
- To use the Google Search App and Google Speech Services stubs, add the following line `device/google/$device/device-$device.mk`:
```
PRODUCT_PACKAGES += gappsstub speechservicestub
```
- If you want to use Screen2Auto, you need to set it's package name and the sha256 hash of your signing cert in `frameworks/base/core/java/android/app/compat/sn00x/AndroidAutoHelper.java` ( PACKAGE_SCREEN2AUTO and SIGNATURES_SCREEN2AUTO in line 54ff). See [Install Screen2Auto](#install-screen2auto) section below.
- To be able to play content protected by DRM, such as Netflix, the device must be missing liboemcrypto.so. Therefor search and comment out the following section in `vendor/google_devices/$device/proprietary/Android.bp`
```
/*
cc_prebuilt_library_shared {
    name: "liboemcrypto",
    owner: "google_devices",
    target: { android_arm64: { srcs: [ "vendor/lib64/liboemcrypto.so" ] } },
    compile_multilib: "64",
    check_elf_files: false,
    prefer: true,
    strip: { none: true },
    soc_specific: true,
}
*/
```
  Note that without liboemcrypto.so Widevine DRM will fall back to using L3 and Netflix will only stream in SD, but at least can be mirrored to the head unit.
- Continue the build process described in [GrapheneOS build instructions](https://grapheneos.org/build)

## Get Android Auto and Google Maps

- Be sure to get Android Auto and Maps with correct signatures. When downloading directly from PlayStore (without having preinstalled stubs on your system), the apps may be provided with other signatures. However, those may checked and rejected when connecting to a car.
  Android Auto must be signed with eeb557fc154afc0d8eec621bdc7ea950 / 9ca91f9e704d630ef67a23f52bf1577a92b9ca5d. Get it e.g. [here](https://www.apkmirror.com/apk/google-inc/android-auto/android-auto-10-2-6332-release/android-auto-10-2-633224-release-android-apk-download/)
  Google Maps must be signed with 38918a453d07199354f8b19af05ec6562ced5788 / f0fd6c5b410f25cb25c3b53346c8972fae30f8ee7411df910480ad6b2d60db83. Get it e.g. [here](https://www.apkmirror.com/apk/google-inc/maps/maps-11-93-0307-release/google-maps-11-93-0307-android-apk-download/)
- Go to phone settings and grant "nearby devices" permission to Android Auto
- Connect to car, follow instructions
- In case your device says there is no app to handle the connection, reboot and try again.
- When an error regarding restricted settings is shown, go to phone settings -> Apps -> Android Auto -> click the three dots in the upper right corner -> Allow restricted settings
- Continue setting up android auto
- In case you run in any problems, replug device

## Install Screen2Auto

Screen2Auto 3.7 can be obtained from https://inceptive.ru/projects/s2a/download

However Google has blacklisted Screen2Auto (after all it circumvents security restrictions of Android Auto).
So you need to find a way to rename the package and sign it with your own cert. I will neither provide a renamed package nor provide any information on how to do this. I don't want Google to blacklist the version I use or change the way they blacklist apps.

Assuming you have a renamed version of Screen2Auto, install it and only grant "System settings recording" when asked for permissions. Screen2Auto needs this permission to modify system settings to be able to e.g. rotate the screen. Leave the other permissions untouched. The patch in this repo handles granting "display over other apps" permission to Screen2Auto only while connected to a head unit.

Make the following changes in Screen2Auto:
- Touch button settings -> Set Double tab to "Launcher"
- Other settings -> Enable "Alternative touch" (but click cancel when asked for enabling Accessibility Service)

If touch does not work for you correctly, try to select another profile under Other settings -> Alternative touch

## Sources

sources for custom built stubs with build instructions for each of the stubs:

https://github.com/SolidHal/SpeechServices-Package-Spoof

https://github.com/SolidHal/Gapp-Package-Spoof

## Thanks
big thanks to everyone involved in the thread here https://github.com/microg/GmsCore/issues/897

https://github.com/VarunS2002/Xposed-Disable-FLAG_SECURE

https://github.com/Magisk-Modules-Repo/liboemcryptodisabler
