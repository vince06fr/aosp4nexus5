# How to Install GApps for the Android Emulator. #

This documentation assumes, that you have set up your build environment as described in [AOSPforNexus5](AOSPforNexus5.md).

## Initialize ##

Change to your working directory and set environment.

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
```

## Choose a Target ##

For the emulator we need the "generic" target aosp\_arm-eng.

```
$ lunch aosp_arm-eng
```

As an alternative, _lunch_ can be called without an argument. Then a list of possible targets is displayed and the number of the target can be entered ("1" for "aosp\_arm-eng").

The "eng" build type stands for "development configuration with additional debugging tools".

## Build plain AOSP for the Emulator ##

If you want to build with "Google Play Store" skip the following steps and continue with the next step "Add Support for _GApps_".

The following description is for building plain AOSP for the emulator.

The Makefile can be started with multiple-threads (-j).

```
$ lunch aosp_arm-eng
$ make -j4
```

Now all sources are compiled and the images are created.
On my machine it took about 6 hours.
The disk space used so far is nearly 70G.

## Run Emulator ##

The path is set from the _lunch_ command, so we can start the emulator.

```
emulator
```

After a short time an emulator window opens and android is starting.

# Add Support for _GApps_ (_Google Play Store_) #

As you can see your device has not many apps installed. You can install apk-files from your desktop using adb.

```
$ adb install xxx.apk
```

For the Google-Apps this method does not suffice,
because they require additional System-Rights like "MANAGE\_USERS".
To get the right the `GoogleServicesFramework` has to be installed as System-App.

## Getting the GApps binaries ##

This is up to you.

A possibility is to unzip the file gapps-lp-20141109-signed.zip from the BasketBuild https://s.basketbuild.com/gapps. The files you need are
  * `system/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk`
  * `system/priv-app/GmsCore/GmsCore.apk`
  * `system/priv-app/Phonesky/Phonesky.apk`


This is the minimal set of binaries necessary to get the Google Play Store up and running.

## Installation of GApps ##

We have to add our new products somewhere. I am not sure, where the right place is. I deceided to use the generic\_no\_telephony make file.

At first we have to modify the generic\_no\_telephony.mk file to add our new vendor "feri" specific files. To be safe make a copy of the original file first.

```
$ cd ~/WORKING_DIRECTORY
$ cp /build/target/product/generic_no_telephony.mk /build/target/product/generic_no_telephony.mk-orig
$ vi /build/target/product/generic_no_telephony.mk
```

Go to the end of the file and add the line marked with the "+"-sign (of course without the "+").

```
...
$(call inherit-product-if-exists, frameworks/webview/chromium/chromium.mk)
$(call inherit-product, $(SRC_TARGET_DIR)/product/core.mk)

+ $(call inherit-product-if-exists, vendor/feri/generic/device-vendor.mk)

# Overrides
PRODUCT_BRAND := generic
PRODUCT_DEVICE := generic
PRODUCT_NAME := generic_no_telephony
```

Then I created the following folders and files.

File "~/WORKING\_DIRECTORY/vendor/feri/generic/device-vendor.mk"

```
PRODUCT_PACKAGES += \
    Phonesky GmsCore GoogleServicesFramework
```

File "~/WORKING\_DIRECTORY/vendor/feri/generic/proprietary/Android.mk"

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_MODULE := Phonesky
LOCAL_MODULE_TAGS := optional
LOCAL_BUILT_MODULE_STEM := package.apk
LOCAL_MODULE_OWNER := google
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)


include $(CLEAR_VARS)
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_MODULE := GmsCore
LOCAL_MODULE_TAGS := optional
LOCAL_BUILT_MODULE_STEM := package.apk
LOCAL_MODULE_OWNER := google
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)


include $(CLEAR_VARS)
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
LOCAL_MODULE := GoogleServicesFramework
LOCAL_MODULE_TAGS := optional
LOCAL_BUILT_MODULE_STEM := package.apk
LOCAL_MODULE_OWNER := google
LOCAL_MODULE_CLASS := APPS
LOCAL_PRIVILEGED_MODULE := true
LOCAL_SRC_FILES := $(LOCAL_MODULE).apk
LOCAL_CERTIFICATE := PRESIGNED
include $(BUILD_PREBUILT)
```

The three APK files have to be copied to "~/WORKING\_DIRECTORY/vendor/feri/generic/proprietary"

```
$ cp GoogleServicesFramework.apk ~/WORKING_DIRECTORY/vendor/feri/generic/proprietary
$ cp GmsCore.apk ~/WORKING_DIRECTORY/vendor/feri/generic/proprietary
$ cp Phonesky.apk ~/WORKING_DIRECTORY/vendor/feri/generic/proprietary
```

That's it. I needed to clean all previous output using the "clobber" target.

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
$ lunch aosp_arm-eng
$ make clobber
```

Now all output is cleared. Start the rebuild with 4 threads

```
$ make -j4
```

This took about 6 hours.

### First start of the Goolge Play Store ###

Start the emulator.

```
$ emulator
```

The emulator window starts. Open the list of all Apps and start the PlayStore.

The Play Store connects to the internet, the emulator simulates this as a 3g connection. So it is important, that your host system has a working internet connection. My Laptop was connected to the internet by WLAN, but the kind of network should not matter.

If all worked well a Dialog "Add your account" appears. All interaction is very slow, so you have to be very patient.
Sometimes it took 10 minutes before the next step appeared.
If something goes wrong just restart the playstore.

After entering the email address and password the message "Unfortunately, Google Play services has stopped" appears. That happened only once and I just ignored this message. Then the "Google Play Terms of Service" have to be accepted.

Even if you are connected to the internet it is likely, that the message "Authentication is reuired..." with a "Retry" button appears. I think this has something to do with concurrency. The Authentication is not finished and Play Store waits for this. I waited and tapped sometimes on the "Retry" button. Finally the Play Store Startscreen appeared and was fully functional.