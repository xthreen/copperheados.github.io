---
layout:      default
title:       Installation
description: Instructions on installing, flashing, sideloading and updating CopperheadOS
---

# Installing CopperheadOS
{: .no_toc}

* TOC
{:toc}

## Supported devices

CopperheadOS is officially supported on the following devices:

* Pixel 2 (walleye)
* Pixel 2 XL (taimen)
* Pixel 3 (blueline)
* Pixel 3 XL (crosshatch)
* Pixel 3a (sargo)
* Pixel 3a XL (bonito)

CopperheadOS has ended support on the following devices:

* HiKey (hikey)
* HiKey 960 (hikey960)
* Nexus 5X (bullhead)
* Nexus 6P (angler)
* Pixel (sailfish)
* Pixel XL (marlin)

CopperheadOS can be ported to other Android devices with Treble support via the standard [device porting process](/android/docs/building#device-porting-process). Most devices do not meet Copperhead [security requirements](/android/docs/devices#minimum-requirements-for-copperheados-support) for official support.

## Prerequisites

You will need at least 4GB of memory on your computer.

Download the latest SDK platform-tools directly from Google [here](https://developer.android.com/studio/releases/platform-tools).

To make your life easier, add the directories to your PATH in your shell profile configuration. This
will make it so you do not have to be inside a specific directory to get access to fastboot.

Adding to your $PATH in [Mac](https://www.architectryan.com/2012/10/02/add-to-the-path-on-mac-os-x-mountain-lion/#.Uydjga1dXDg), [Windows](https://helpdeskgeek.com/windows-10/add-windows-path-environment-variable/) and [Linux](https://opensource.com/article/17/6/set-path-linux).

## Enabling USB Debugging and OEM Unlocking

USB Debugging and OEM unlocking will need to be enabled on the phone.

Enable the developer settings menu by going to Settings -> About phone and pressing on the build
number menu entry until developer mode is enabled.

Next, go to Settings -> System -> Advanced -> Developer options -> and toggle on Enable OEM Unlocking, then scroll further
down the menu and toggle on USB Debugging. Confirm your action with "Okay" on the following dialog box.

Now connect your phone to the computer, if you have not done so already. Unlock the phone if it is locked. You will need
to give permission on the phone to allow the computer to run the rest of the install process.

### Latest CopperheadOS images

CopperheadOS currently provides factory images to the [Copperhead Partner network](https://copperhead.co/partners) and is not open to the public for download.

## Flashing factory images

First, boot into the bootloader interface, using ```adb
reboot bootloader``` in your terminal of choice.

Alternatively, you can do this by powering off the device and then
powering it on while holding both the Volume Down and Power buttons.

### Supported devices unlocking directions
The bootloader now needs to be unlocked to allow flashing new images:

    fastboot flashing unlock

This command needs to be confirmed on the phone.

### Legacy Pixel 2 XL bootloader unlocking directions
On some older models of Pixel 2 XL it's necessary to unlock the
critical partitions.

    fastboot flashing unlock_critical

This command needs to be confirmed on the phone.

## Flashing factory images

Next, extract the factory images and run the script to flash them. Extracting the factory images
depends on the OS you're using and the applications available.

* [Using 7zip on Windows](https://itstillworks.com/untar-windows-8537010.html)
* [Extracting files in Mac](https://support.apple.com/en-ca/guide/terminal/apdc52250ee-4659-4751-9a3a-8b7988150530/mac)

*$DEVICE will either be crosshatch, taimen, walleye, sargo, bonito or blueline. The $DATE of the factory
image will correspond to our [releases](/android/docs/updates).*

Note that the ```fastboot``` command run by the flashing script requires a fair bit of free
space in a temporary directory, which defaults to ```/tmp```:

    tar xvf $DEVICE-factory-$DATE.tar.gz
    cd $DEVICE-factory-$DATE
    ./flash-all.sh
    
In Windows run ```./flash-all.bat```

Use a different temporary directory if your ```/tmp``` doesn't have 2GiB available:

    mkdir tmp
    TMPDIR=$PWD/tmp ./flash-all.sh

Now proceed to locking the bootloader before using the phone. User data will be erased in a factory
reset when the bootloader is locked or unlocked.

## Setting custom AVB key

The custom AVB key for supported Pixel series phones should automatically
be set upon flashing.

## Locking the bootloader

Locking the bootloader is important as it enables full verified boot. It also prevents using
fastboot to flash, format or erase partitions.  Verified boot will detect modifications to any of
the OS partitions (boot, recovery, system, vendor) and it will prevent reading any modified /
corrupted data. If changes are detected, error correction data is used to attempt to obtain the
original data at which point it's verified again which makes verified boot robust to non-malicious
corruption.

Reboot into the bootloader menu and set it to locked:

    fastboot flashing lock

The command needs to be confirmed on the phone. A factory reset will be performed.

### Post installation precautions

OEM unlocking should be disabled again in the developer settings menu on the phone.
This prevents unlocking the bootloader without access to the owner account. CopperheadOS prevents
bypassing the OEM unlocking toggle by wiping the data partition from the hidden recovery menu,
unlike stock Android.  You can still trigger factory resets from within the OS. Note that this
means that recovering a device with a forgotten password is not possible without Copperhead doing
it, which is the main purpose of this feature (anti-theft).  Stock Android can be more forgiving
because it's tied to a Google account.

**Note: Unlocking the bootloader again will erase all user data. Only unlock the bootloader if
it is absolutely necessary.**

You should now disable OEM unlocking and factory reset the CopperheadOS device
for optimal hardware security.


## Updating

### Update client

See [the section in the usage guide](/android/docs/usage_guide#updates-on-pixel-phones).

### Sideloading

Updates can also be downloaded from Copperhead and installed via
recovery with adb sideloading. The zip files are signed and will be verified by the CopperheadOS
recovery image. Sideloading should only be done if OTA updates are not working and there is a
critical issue with Updater.

First, boot into recovery. You can do this either by using ```adb reboot recovery``` from the
operating system or selecting the Recovery option in the bootloader menu.

You should see an Android lying on their back being repaired, with the text "No command" meaning
that no command has been passed to recovery.

Next, access the recovery menu by holding down the power button and pressing the volume up button
a single time. This key combination toggles between the GUI and text-based mode with the menu and
log output.

Finally, select the "Apply update from ADB" option in the recovery menu and sideload the update
with adb:

    adb sideload $DEVICE-ota_update-$DATE.zip
