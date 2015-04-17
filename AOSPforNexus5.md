# How-To AOSP #


---

I have tested the documentation here at December, 22nd on my Laptop with 4GB Memory and Core i5 processor. I deployed Android 5.0.2r1 to a  Nexus 5 device.

---


You are a developer and want to start customizing your Android device using the given sources in the **A** ndroid **O** pen **S** ource **P** roject (AOSP), but you are stuck because of the complexity?

This documentation will help you with a detailed description for the Nexus 5 device. The general concepts will work for all devices, but with a Nexus 5 the steps will work out of the box without modification.


The structure of this documentation:




# Set up build environment #

As stated in [AOSP - Downloading and Building](http://source.android.com/source/building.html) the preferred development environment is the Linux distribution "Ubuntu 12.04".

## Install Ubuntu 12.04 ##

I tried using a virtual machine under Windows 7 (the free VMWare player worked well), but the results were so slow, that I decided to install Linux in a separate partition and boot from there.

It is important to use the 64 bit version of Ubuntu, otherwise there will be problems with pre-built binaries (e.g. ccache). Even if you have an Intel processor the 64 bit version is named amd64: ubuntu-12.04.5-desktop-amd64.iso.
The desktop installation image can be downloaded from http://releases.ubuntu.com/12.04/

My laptop has only 4GB, so I installed a 16GB swap space partition. The ext4 linux partition has 160GB. As a minimum I would recommend 100GB.

After the fresh Linux install I followed the instructions from [AOSP - Initializing a Build Environment](https://source.android.com/source/initializing.html)

**Hint** for newbies: If you are looking at the Desktop where to start a shell-terminal, just Press `<Ctrl>`-`<Alt>`-'t' or click on the dash-home and start typing "terminal" and then click on the application "Terminal".

### Chosing a Branch ###

First I chose the release to be built from http://source.android.com/source/build-numbers.html.
I selected the currently newest build "LRX22G", Branch **android-5.0.2\_r1**

### Installing the JDK ###

For this build JDK 7 is required. Open JDK is recommended.

```
$ sudo apt-get update
$ sudo apt-get install openjdk-7-jdk
```

### Installing required packages ###

The recommended "apt-get install" fails, because "libgl1-mesa-glx:i386" has unmet dependencies. The necessary libg1api-mesa release seems to have changed with the Ubuntu sub-releases. I found this hint here http://stackoverflow.com/questions/23254439/android-setting-up-a-linux-build-environment-libgl1-mesa-glxi386-package-have

But the proposed solution to use libglapi-mesa-lts-saucy:i386 does not work with the current release, so I listed all possible solutions using apt-cache:

| $ sudo apt-cache search libglapi-mesa |
|:--------------------------------------|

So I found the newest release "trusty" which works.
So the following installation commands have to be executed.

```
$ sudo apt-get install git gnupg flex bison gperf build-essential \
  zip curl libc6-dev libncurses5-dev:i386 x11proto-core-dev \
  libx11-dev:i386 libreadline6-dev:i386 \
  libgl1-mesa-dev g++-multilib mingw32 tofrodos \
  python-markdown libxml2-utils xsltproc zlib1g-dev:i386
```

```
$ sudo apt-get install libglapi-mesa-lts-trusty:i386
```

The recommended symbolic link already exsits, so the following command must not be executed.

| $ sudo ln -s /usr/lib/i386-linux-gnu/mesa/libGL.so.1 /usr/lib/i386-linux-gnu/libGL.so |
|:--------------------------------------------------------------------------------------|

### Configuring USB Access ###

```
$ sudo vi /etc/udev/rules.d/51-android.rules
```

Press "i" (change from command mode to insertion mode).
Paste the following content into the editor

```
# adb protocol on passion (Nexus One)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e12", MODE="0600", OWNER="<username>"
# fastboot protocol on passion (Nexus One)
SUBSYSTEM=="usb", ATTR{idVendor}=="0bb4", ATTR{idProduct}=="0fff", MODE="0600", OWNER="<username>"
# adb protocol on crespo/crespo4g (Nexus S)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e22", MODE="0600", OWNER="<username>"
# fastboot protocol on crespo/crespo4g (Nexus S)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e20", MODE="0600", OWNER="<username>"
# adb protocol on stingray/wingray (Xoom)
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", ATTR{idProduct}=="70a9", MODE="0600", OWNER="<username>"
# fastboot protocol on stingray/wingray (Xoom)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="708c", MODE="0600", OWNER="<username>"
# adb protocol on maguro/toro (Galaxy Nexus)
SUBSYSTEM=="usb", ATTR{idVendor}=="04e8", ATTR{idProduct}=="6860", MODE="0600", OWNER="<username>"
# fastboot protocol on maguro/toro (Galaxy Nexus)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e30", MODE="0600", OWNER="<username>"
# adb protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d101", MODE="0600", OWNER="<username>"
# adb protocol on panda (PandaBoard ES)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="d002", MODE="0600", OWNER="<username>"
# fastboot protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d022", MODE="0600", OWNER="<username>"
# usbboot protocol on panda (PandaBoard)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d00f", MODE="0600", OWNER="<username>"
# usbboot protocol on panda (PandaBoard ES)
SUBSYSTEM=="usb", ATTR{idVendor}=="0451", ATTR{idProduct}=="d010", MODE="0600", OWNER="<username>"
# adb protocol on grouper/tilapia (Nexus 7)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e42", MODE="0600", OWNER="<username>"
# fastboot protocol on grouper/tilapia (Nexus 7)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4e40", MODE="0600", OWNER="<username>"
# adb protocol on manta (Nexus 10)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4ee2", MODE="0600", OWNER="<username>"
# fastboot protocol on manta (Nexus 10)
SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", ATTR{idProduct}=="4ee0", MODE="0600", OWNER="<username>"
```

Then press `<Esc>` (Stop insertion mode).
Type ":w" (write to file) and press `<Enter>`.
Type ":q" (quit) and press `<Enter>`.

### Setting up ccache ###

Add setting for using ccache by editing .bashrc in the home folder.
ccache increases the recompilation speed. The size of the cache will be set later.
We will not change the default cache dir "CCACHE\_DIR", so the default "~/.ccache" will be used.

```
$ vi ~/.bashrc
```

Press `<Shift>`-"G" (Jump to end of document).
Press "o" (Open a new line after current line).
Paste the following text into the editor.

```
export USE_CCACHE=1
```

Press `<Esc>` (change to command mode).
Type ":w" (write to file) and press `<Enter>`.
Type ":q" (quit) and press `<Enter>`.

The change is not instantly active, but as soon as you open a new bash shell.
So to be sure, the variable is defined also set the variable in the command line:

```
$ export USE_CCACHE=1
```


# Check out the sources #

## Installing Repo ##

Create user specific bin folder:

```
$ mkdir ~/bin
```

Add this folder to your path:

```
$ PATH=~/bin:$PATH
```

To make this change permanent add this line to ~/.bashrc.

```
$ vi ~/.bashrc
```

Press `<Shift>`-"G" (Jump to end of document).
Press "o" (Open a new line after current line).
Paste the following text into the editor.

```
PATH=~/bin:$PATH
```

Press `<Esc>` (change to command mode).
Type ":w" (write to file) and press `<Enter>`.
Type ":q" (quit) and press `<Enter>`.

Download the binary for repo and make it executable.

```
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```

## Initializing a Repo client ##

Create the root directory for your development.

```
$ mkdir ~/WORKING_DIRECTORY
$ cd ~/WORKING_DIRECTORY
```

Check out the sources for the branch "android-5.0.2\_r1"

```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-5.0.2_r1
```

You have to enter your name and email address. Also you are asked, whether you can see colors.

## Downloading the Android Source Tree ##

```
$ repo sync
```

This will take some time, my laptop needed 5 hours and 30GB disk space.

Now we have checked out the binaries for ccache, we can configure its size:

```
$ cd ~/WORKING_DIRECTORY/prebuilts/misc/linux-x86/ccache
$ ./ccache -M 50G 
```

If you get an error, that ccache is not executable, it is very likely, that your Linux is 32bit. You need a 64bit Linux installation (see above amd64, not x86).

# Build and flash to device #

## Initialize ##

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
```

## Choose a Target ##

For Nexus 5 we need "hammerhead" as target.

```
$ lunch aosp_hammerhead-userdebug
```

For the emulator we need "aosp\_arm-eng" as target"

| $ lunch aosp\_arm-eng |
|:----------------------|

As an alternative, _lunch_ can be called without an argument. Then a list of possible targets is displayed and the number of the target can be entered ("21" for "aosp\_hammerhead-userdebug").

The "userdebug" build type allows us later to access a root shell on the Android device.

## Build for the Emulator ##

The description was moved to an own page [GApps4Emulator](GApps4Emulator.md), please look there.


## Add device specific binaries ##

The device specific binaries have to be downloaded here http://source.android.com/source/building-devices.html#obtaining-proprietary-binaries

Because we work with the newest release I downloaded the three Nexus 5 binaries form the binaries preview page https://developers.google.com/android/nexus/blobs-preview

| **Download-File** | **Bytes** |
|:------------------|:----------|
| lge-hammerhead-1632165-898085bf.tgz | 154037 |
| broadcom-hammerhead-1632165-faaa4e24.tgz | 68095 |
| qcom-hammerhead-1632165-a0e3f7be.tgz | 22170416 |

I copied the three downloaded files into folder "~/Downloads"

Then all files have to be extracted:

```
$ cd ~/Downloads
$ tar xzvf lge-hammerhead-1632165-898085bf.tgz
$ tar xzvf broadcom-hammerhead-1632165-faaa4e24.tgz
$ tar xzvf qcom-hammerhead-1632165-a0e3f7be.tgz
```

Now there are three executable "extract-...-hammerhead.sh" files

The three self extracting files have to be executed from the root folder of the source tree (~/WORKING\_DIRECTORY).

```
$ cd ~/WORKING_DIRECTORY
$ ~/Downloads/extract-qcom-hammerhead.sh
$ ~/Downloads/extract-broadcom-hammerhead.sh
$ ~/Downloads/extract-lge-hammerhead.sh
```

Each script displays its license to you. Press Enter and scroll further by pressing Space. At the end you have to type "I ACCEPT". Instead of scrolling through the license using Space you can stop displaying the license by pressing "q".

Because the vendor specific scripts modify the build process a full rebuild has to be initiated. A clean can be done by making the synthetic target "clobber".
If you started a new shell, you have to set the environment

```
$ . build/envsetup.sh
$ lunch aosp_hammerhead-userdebug
```

Then clean the output

```
$ make clobber
```

Now the full rebuild can be done.

```
$ make -j4
```
This will again take 5 hours.

## Flashing to Device ##

Build the tools _adb_ and _fastboot_ (was not really necessary, make reports "Nothing to be done").

```
$ make fastboot adb
```

Now there is a little problem with fastboot. To be able to access a device in bootloader mode fastboot has to be started as root. But as root the environment variables are not set, so we have to set them manually.
So first get the currently active values. The values displayed here and in the following document assume the currently logged in user to be "**devuser**".

```
$ echo $USER
devuser
$ type fastboot
fastboot is /home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin/fastboot
$ echo $OUT
/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
$ echo $ANDROID_PRODUCT_OUT
/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
```

Now we can set the environment for root.
Replace all four occurrences of **devuser** with your currently logged in user. You can find out your username with the command "echo $USER".

```
$ sudo su -
# cd ~devuser/WORKING_DIRECTORY/
# PATH=$PATH:/home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin
# export ANDROID_PRODUCT_OUT=/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
```

Now we can use adb and fastboot with correct settings.

Attach your device and check that adb sees the device

```
# adb devices -l
List of devices attached 
0555a8e3f0df5274       device usb:2-2 product:hammerhead model:Nexus_5 device:hammerhead
```

If the device does not appear, something went wrong, you do not need to continue, before the problem is fixed (enable USB Debugging in the developer options, tap 7 times on the build number if you do not have developer options).

Set the device into bootloader mode.

```
# adb reboot bootloader
```

A laying droid appears and at the top there is a green  arrow with the text "Start".

In this mode fastboot should see the device

```
# fastboot devices
0555a8e3f0df5274	fastboot
```

If your device does not appear, you need not to continue. Fix this issue first.

### Unlock ###

If the bottom line is "LOCK STATE - locked" you have to unlock your device. For the Nexus devices this is quite simple using fastboot.

```
# fastboot oem unlock
```

Your Android device displays a message "Unlock bootloader? ..." and you have to select "Yes" or "No".
Try not to tap on the "Yes" button, it won't recognize it. You need to use the **Volume Up** and **Volume Down** buttons to select "Yes". Then press the **Power** button. Now the bottom line is red and reads "LOCK STATE - unlocked".

### Flash ###

Now flash your device.
It is important that your environment variable "$ANDROID\_PRODUCT\_OUT" is set correctly, because this is the place where fastboot looks for the image files (see above for how to set the correct values).

```
# fastboot -w flashall
```

This will take about one minute. Your device reboots after the flashing is finished.

If all went well, your first self built Android is up and running  :-)

TODO: insert screenshot!

If you want to make a screenshot of your first login-screen you can use the following command (see [Grab Android Screenshot](http://blog.shvetsov.com/2013/02/grab-android-screenshot-to-computer-via.html))

```
# adb shell screencap -p | sed 's/\r$//' > ~devuser/Pictures/screen.png
```

## Get a Root Shell ##

If your device is attached, running and you have allowed USB Debugging, you can get a Shell with the "adb shell" command.

```
# adb shell
shell@hammerhead:/ $ 
```

The shell you enter is on your Android device. The user is "shell". The "shell" user has not full access. Leave the ADB-Shell with "$ exit".

```
shell@hammerhead:/ $ id
uid=2000(shell) gid=2000(shell) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=u:r:shell:s0
shell@hammerhead:/ $ exit
```

To activate the root shell restart the adb-daemon as root and open another adb shell.

```
# adb root
restarting adbd as root
# adb shell
root@hammerhead:/ # id
uid=0(root) gid=0(root) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=u:r:su:s0
```


## Get Write Access to the System Partition ##

Even if you have a root shell you are not able to modify any binary in the root folder "/" or under "/system".
The reason is, that the root and system partition are mounted read-only.

The name of the partition can vary from device to device, so to find the partitions use mount and look for "rootfs" and "/system".

```
root@hammerhead:/ # mount | grep "rootfs"
rootfs / rootfs ro,seclabel,relatime 0 0

root@hammerhead:/ # mount | grep "/system"
/dev/block/platform/msm_sdcc.1/by-name/system /system ext4 ro,seclabel,relatime,data=ordered 0 0
```

Remount the partitions and change the "ro" flag to rw.

```
mount -o remount,rw / /
mount -o remount,rw -t ext4 /dev/block/platform/msm_sdcc.1/by-name/system /system
```

If you are running only the emulator the block device is as follows.

| mount -o remount,rw -t ext4 /dev/block/mtdblock0 /system  |
|:----------------------------------------------------------|

Now you have write access and can crash your system...   :-)
After changing any system file you should remount the partition read-only:

```
mount -o remount,ro / /
mount -o remount,ro -t ext4 /dev/block/platform/msm_sdcc.1/by-name/system /system
```


# Add support for _GApps_ (_Google Play Store_) #

As you can see your device has not many apps installed. You can install apk-files from your desktop using adb.

```
$ adb install xxx.apk
```

For the Google-Apps this method does not suffice,
because they require additional System-Rights like "MANAGE\_USERS".
To get the right the `GoogleServicesFramework` has to be installed as System-App.

## Getting the GApps binaries ##

This is up to you.

I did it this way:
  * I rooted my Nexus 10 device with Android 5.0.1 (Lollipop)
  * I copied the files
    * `/system/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk`
    * /data/app/com.android.vending-1/base.apk (renamed to `GooglePlayStore.apk`), this APK is named Phonesky.apk in Nexus 5
    * /data/app/com.google.android.gms-1/base-apk (renamed to `GooglePlayService.apk`), this APK is named `PrebuiltGmsCore.apk` in Nexus 5

Another possibility is to unzip the file gapps-lp-20141109-signed.zip from the BasketBuild https://s.basketbuild.com/gapps. The files you need are
  * `system/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk`
  * `system/priv-app/GmsCore/GmsCore.apk`
  * `system/priv-app/Phonesky/Phonesky.apk`


---

### NOTE ###
I used the Nexus 10 APKs, because the GoogleServicesFramework.apk in the Nexus 5 Factory image is missing its classes.dex. I am not totally sure about that, but I assume a pre-compiled hardware specific file (odex) is used instead. But Simply copying the odex file did not solve my problem, so I stopped trying to use the Nexus 5 files. With the Nexus 10 APKs all works fine.

The APKs from the basketbuild also worked well.

---


This is the minimal set of binaries necessary to get the Google Play Store up and running.

### INFO: How to get the APKs out of a rooted device? ###

As said above I rooted my Nexus 10 and transfered the files as described here.

To find the position of the binaries you can use the Package-Manager. To get a list of all installed packages:

```
pm list packages
```

To find the installation-path of the apk the Package-Manager can also be used.

```
# pm path com.google.android.gsf
# pm path com.android.vending
# pm path com.google.android.gms
```

#### transfer files ####

You can transfer the files using adb push and pull.
To copy from device to your desktop use "pull".

```
$ adb pull /system/priv-app/GoogleServicesFramework/GoogleServicesFramework.apk ./GoogleServicesFramework.apk
$ adb pull /data/app/com.android.vending-1/base.apk ./Phonesky.apk
$ adb pull /data/app/com.google.android.gms-1/base-apk ./GmsCore.apk
```

To copy from your desktop to the Android device use "push" and the /sdcard or any other writeable filesystem as target folder.

```
$ adb push ./GoogleServicesFramework.apk /sdcard/GoogleServicesFramework.apk 
$ adb push ./Phonesky.apk /sdcard/Phonesky.apk
$ adb push ./GmsCore.apk /sdcard/GmsCore.apk
```

## manual install the binaries as system apps ##

However you got your three APKs in the following description we assume the APKs be named GoogleServicesFramework.apk, GmsCore.apk and Phonesky.apk. If they are named different you can rename them or adjust the names in the following description.

First get a root shell, then mount the system partition for read-write and install the three apks under /system/priv-app and finally set the file-access-rights.

### get root shell ###

```
$ adb root
$ adb shell
```

### mount system partition read-write ###

(for exact parameters use "mount | grep '/system'", see above)

```
# mount -o remount,rw -t ext4 /dev/block/platform/msm_sdcc.1/by-name/system /system
```

### install APKs as System-App ###

```
# cd /system/priv-app

# mkdir GoogleServicesFramework
# mkdir Phonesky
# mkdir GmsCore

# cp /sdcard/GoogleServicesFramework.apk GoogleServicesFramework/GoogleServicesFramework.apk
# cp /sdcard/Phonesky.apk Phonesky/Phonesky.apk
# cp /sdcard/GmsCore.apk GmsCore/GmsCore.apk

# chmod 755 GoogleServicesFramework
# chmod 755 Phonesky
# chmod 755 GmsCore

# chmod 644 GoogleServicesFramework/GoogleServicesFramework.apk
# chmod 644 Phonesky/Phonesky.apk
# chmod 644 GmsCore/GmsCore.apk
```

### reboot to update system apps ###

Now you can reboot your device.
Before the login display appears, there should be a Message Box saying

```
Android is upgrading...
() Optimizing app 1 of 3
```

### First start of the Goolge Play Store ###

Before starting the Play Store you should configure your Wi-Fi connection, because the Play Store connects to the internet to authenticate your Google account.

If all worked well a Dialog "Add your account" appears.

After entering the email address and password the message "Unfortunately, Google Play services has stopped" appears. That happened only once and I just ignored this message. Then the "Google Play Terms of Service" have to be accepted.

Even if you are connected to the internet it is likely, that the message "Authentication is reuired..." with a "Retry" button appears. I think this has something to do with concurrency. The Authentication is not finished and Play Store waits for this. I waited and tapped sometimes on the "Retry" button.
After about one minute the Play Store Startscreen appeared and was fully functional.


### drawback of the manual method ###

The Google Play Store works as long as no new image is flashed.
As soon as a new image is flashed all manual steps have to be repeated.
So we look for a method to include the APKs into the system.img.

## Automatic Installation of GApps ##

I took a look at the vendor-specific files from lge. There is a prebuilt APK file "qcrilmsgtunnel.apk" which is added to the system image.

At first we have to modify the full\_hammerhead.mk file to add our new vendor "feri" specific files.

```
$ cd ~/WORKING_DIRECTORY
$ vi device/lge/hammerhead/full_hammerhead.mk
```

Go to the end of the file (`<Shift>`-"G") and add the following line ("o").

```
$(call inherit-product-if-exists, vendor/feri/hammerhead/device-vendor.mk)
```

Press `<Esc>`, Type ":w"  and press `<Enter>` and then ":q" and press `<Enter>`.

Then I created the following folders and files.

File "~/WORKING\_DIRECTORY/vendor/feri/hammerhead/device-vendor.mk"

```
PRODUCT_PACKAGES += \
    Phonesky GoogleServicesFramework GmsCore
```

File "~/WORKING\_DIRECTORY/vendor/feri/hammerhead/proprietary/Android.mk"

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

(Note: To use "LOCAL\_MODULE\_OWNER := feri" the vendor\_owner\_whitelist in "build/core/tasks/vendor\_module\_check.mk" has to be extended. To avoid this we just use the predefined vendor "google")

The three APK files are copied to "~/WORKING\_DIRECTORY/vendor/feri/hammerhead/proprietary"

```
$ cp GoogleServicesFramework.apk ~/WORKING_DIRECTORY/vendor/feri/hammerhead/proprietary
$ cp Phonesky.apk ~/WORKING_DIRECTORY/vendor/feri/hammerhead/proprietary
$ cp GmsCore.apk ~/WORKING_DIRECTORY/vendor/feri/hammerhead/proprietary
```

That's it. Now just rebuild, no need to cleanup.

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
$ lunch aosp_hammerhead-userdebug
$ make -j4
```

This should not take too long, perhaps 5 minutes.

Now flash the device as above (replace **devuser** with your login).

```
$ sudo su -
# cd ~devuser/WORKING_DIRECTORY/
# PATH=$PATH:/home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin
# adb reboot bootloader
```

```
# export ANDROID_PRODUCT_OUT=/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
# fastboot -w flashall
```

After rebooting you should be able to use the Play Store as described above [First Play-Store start](#First_start_of_the_Goolge_Play_Store.md)


# Debugging Java and C sources using `LogCat` #

## Debugging a Java Class ##

In the first step we will add debugging output to a Java class.

### Modify Java class ###

We will modify a Java class and add some logging.
For example, we are interested, when an App accesses its Shared-Preferences.
The Shared-Preferences are a simple storage, where an application may persists some values.

So we want to log, every access to `android.app.ContextImpl.getSharedPreferences()`.
To do this we simply edit the file "`~/WORKING_DIRECTORY/frameworks/base/core/java/android/app/ContextImpl.java`".
At line 899 there is the method to be logged.
We just add an call to "android.util.Log" with the tag "HOWTOAOSP".

```
    @Override
    public SharedPreferences getSharedPreferences(String name, int mode) {

        android.util.Log.d("HOWTOAOSP","ctximpl.getSharedPreferences("+name+")");

        SharedPreferencesImpl sp;
        ...
    }
```

So that's already all we have to do on the code site to log into logcat.

### Building ###

We just start the complete build process again. Make is not so clever with Java files, so the rebuild lasts 25 minutes.

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
$ lunch aosp_hammerhead-userdebug
$ make -j4
```

### flashing ###

Again first remember the environment variables set for your Build-User

```
$ echo $ANDROID_PRODUCT_OUT
$ type fastboot
```

Become Super-User and setup the environment (replace **devuser** with your values).

Attach your device and go into bootloader

```
$ sudo su -
# cd ~devuser/WORKING_DIRECTORY/
# PATH=$PATH:/home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin
# adb reboot bootloader
```

Flash the newly created image.

```
# export ANDROID_PRODUCT_OUT=/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
# fastboot -w flashall
```

Sometimes after flashing a new image there are two error messages.

  * Unfortunately, Gallery has stopped
  * Unfortunately, Camera has stopped

I am not sure, what the reason is for this, but it has nothing to do with the changes we just made.
I am not really interested in getting Camera and Gallery to work, so I ignore this error.


### Testing Debug Output ###


To view the debuglogfile adb can be used with the "logcat"-command. As second parameter a filter can be defined at which level logging should be displayed.
We are only interested in our own output, so we can set the filter for the tag "HOWTOAOSP" to verbose and for all other tags to fatal.

```
$ adb logcat "HOWTOAOSP:v *:f"
```

Now we start the Setting app on the Android device. The following lines appear in the log

```
D/HOWTOAOSP( 3810): ctximpl.getSharedPreferences(development)
D/HOWTOAOSP( 3810): ctximpl.getSharedPreferences(home_prefs)
```

So our debugging output works well.   :-)

More detailed information about `LogCat` can be found here http://developer.android.com/tools/debugging/debugging-log.html#startingLogcat

## Debugging a C file ##

We are interested in logging calls to the "su" binary.
"su" is called to get root rights.

### Modify C file ###

Make a copy of the original file, so you are able to rollback any changes made by mistake.

```
$ cd ~/WORKING_DIRECTORY/system/extras/su
$ cp su.c su.c-orig
```

Edit the file "su.c" and add the lines below marked with "+":

```
/*
**
** Copyright 2008, The Android Open Source Project
...
#include <pwd.h>

#include <private/android_filesystem_config.h>

+ #include <android/log.h>
+
+ #define ALOG(fmt,args...) __android_log_print(ANDROID_LOG_INFO, "HOWTOAOSP", fmt, ##args);

void pwtoid(const char *tok, uid_t *uid, gid_t *gid)
...
int main(int argc, char **argv)
{
    struct passwd *pw;
    uid_t uid, myuid;
    gid_t gid, gids[10];
+     int i;

    /* Until we have something better, only root and the shell can use su. */
    myuid = getuid();
    
+     ALOG("su: calling uid=%d", myuid);
+     for (i=0; i<argc; i++) {
+         ALOG("  arg: '%s'", argv[i]);
+     }
    
    if (myuid != AID_ROOT && myuid != AID_SHELL) {
        fprintf(stderr,"su: uid %d not allowed to su\n", myuid);
        return 1;
    }
    ....
    fprintf(stderr, "su: exec failed\n");
    return 1;
}
```

Because su.c did not use android-logging before (include `<android/log>`) we have to add the logging library to the makefile.

So make a backup and edit the "Android.mk" file in the same directory and add "liblog" to LOCAL\_STATIC\_LIBRARIES

```
...
LOCAL_STATIC_LIBRARIES := libc liblog
...
```

### Building ###

As above restart a full build. It us much faster then rebuilding after changing a Java Class. It will take about 3 minutes. Perhaps this is the Advantage of ccache?

```
$ cd ~/WORKING_DIRECTORY
$ . build/envsetup.sh
$ lunch aosp_hammerhead-userdebug
$ make -j4
```

### flashing ###

As above flash the image to your device.

Attach your device and go into bootloader

```
$ sudo su -
# cd ~devuser/WORKING_DIRECTORY/
# PATH=$PATH:/home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin
# adb reboot bootloader
```

Flash the newly created image.

```
# export ANDROID_PRODUCT_OUT=/home/devuser/WORKING_DIRECTORY/out/target/product/hammerhead
# fastboot -w flashall
```

Reboot your device.


### Testing Debug Output ###

As above we use adb to view debug logging.

```
$ adb logcat "HOWTOAOSP:v *:f"
```

The binary "su" is normally not called, so there is nothing in the logfile.
So we open a second terminal and execute some commands while looking at the logfile in the other window (replace **devuser** with your login name).

```
$ PATH=$PATH:/home/devuser/WORKING_DIRECTORY/out/host/linux-x86/bin
$ adb shell
shell@hammerhead:/ $ id
uid=2000(shell) gid=2000(shell) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=u:r:shell:s0
shell@hammerhead:/ $ su root
shell@hammerhead:/ # id
uid=0(root) gid=0(root) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=u:r:su:s0
shell@hammerhead:/ # su adb
shell@hammerhead:/ $ id
uid=1011(adb) gid=1011(adb) groups=1003(graphics),1004(input),1007(log),1011(adb),1015(sdcard_rw),1028(sdcard_r),3001(net_bt_admin),3002(net_bt),3003(inet),3006(net_bw_stats) context=u:r:su:s0
shell@hammerhead:/ $ exit
shell@hammerhead:/ # exit
shell@hammerhead:/ $ exit
```

In the other window you see the following logs.

```
I/HOWTOAOSP( 3401): su: calling uid=2000
I/HOWTOAOSP( 3401):   arg: 'su'
I/HOWTOAOSP( 3401):   arg: 'root'
I/HOWTOAOSP( 3450): su: calling uid=0
I/HOWTOAOSP( 3450):   arg: 'su'
I/HOWTOAOSP( 3450):   arg: 'adb'
```

So once again, our debug-logging works.    :-)

The binary "su" could be deployed much simpler than flashing by pushing the binary with adb.


# Next Steps #

more to come...
  * manual install of busybox
  * modifying su.c to allow becoming superuser for all accounts


# Links #

Here are links to other interesting resources.

[Howto Build Android KitKat (4.4) for the Google Nexus 5](http://nosemaj.org/howto-build-android-kitkat-nexus-5)