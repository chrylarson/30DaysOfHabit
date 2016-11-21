---
layout: post
title:  "How to use Cordova on Ubuntu to build Android apps"
date:   2014-03-04 12:28:00 -0500
categories: blog tutorial
permalink: /blog/how-use-cordova-ubuntu-build-android-apps
---
<amp-img width="600" height="450" alt="How to use Cordova on Ubuntu to build Android apps" layout="responsive" src="{{ site.baseurl }}/img/cordova-apps-for-android.png "></amp-img>
Building native HTML and JavaScript apps for Android can be done with Apache Cordova on Ubuntu.

* Download the [Android SDK](http://developer.android.com/sdk/index.html) from developer.android.com
* Unzip the Android SDK and place the contents of the 'sdk' folder in ~/Development/adt-bundle/sdk
* Add the SDK to your path
  * `PATH=$PATH:~/Development/adt-bundle/sdk/platform-tools:~/Development/adt-bundle/sdk/tools`
  * To permanently add the Android SDK to your Ubuntu path follow [this Ubuntu.com article](https://help.ubuntu.com/community/AndroidSDK#Modifying_the_PATH_Environment_Variable)
* install Apache Cordova with npm `npm install -g cordova` ([Install nodejs](https://nodejs.org) if you don't have it.)
  * cd to your Documents directory and create a helloWorld folder, then cd into that folder
  * `cd ~/Documents`
  * `mkdir helloWorld`
  * `cd helloWorld`
* Create a Cordova app and cd to the new folder
  * `cordova create hello com.example.hello "Hello World"`
  * `cd hello/`
* Add Android as a cordova platform
  * `cordova platform add android`
* Create an Andorid emulator
  * `android create avd -n hello -t 1`
* Run the cordova app on the emulator
  * `cordova emulate android`
* Build your web app and place it in the www folder, making sure to include config.xml and cordova.js