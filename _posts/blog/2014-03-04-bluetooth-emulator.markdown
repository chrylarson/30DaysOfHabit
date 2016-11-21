---
layout: post
title:  "Emulate Android and Bluetooth LE hardware"
date:   2014-03-04 12:28:00 -0500
categories: blog tutorial
permalink: /blog/emulate-android-and-bluetooth-le-hardware
---
<amp-img width="600" height="450" alt="Emulate Android and Bluetooth LE hardware" layout="responsive" src="{{ site.baseurl }}/img/home-automation.png "></amp-img>
Android emulators are great for developing BluetoothLE applications. The trick is getting the Android emulator to recognize the BluetoothLE adapter.

What you'll need:

* Androidx86 iso from [android-x86.org](http://android-x86.org/) I used the [4.4 release candidate](http://www.android-x86.org/releases/releasenote-4-4-rc1)
* Virtual Machine software: I used [Oracle VirtualBox](https://www.virtualbox.org/)
* A BluetoothLE USB adapter: I used the [Cirago Bluetooth 4.0 USB Mini Adapter (BTA8000)](http://www.amazon.com/Cirago-Bluetooth-Mini-Adapter-BTA8000/dp/B0090I9NRE/)
* [Android SDK](https://developer.android.com/sdk/index.html) for debugging
* Install VirtualBox
* Download Androidx86
* Open VirtualBox and create a new machine. Set type to linux/other(32bit)
* Set the virtual machine's memory and harddrive space to whatever you need (but at least the minimum specs for Android).
* When asked for the OS image, select the Androidx86 image you download from Androidx86.org
* When the virtual machine boots, choose to install Android.
* When the installation completes, shutdown the Android virtual machine and unmount the iso image
* Plug in the Bluetooth USB adapter and add it to the Android Virtual Machine's settings
* Start the Android Virtual Machine and go through the start-up screens to configure Android for use
* In the Android VM go to the settings and enable BluetoothLE (if this fails reboot the VM and try to enable again)