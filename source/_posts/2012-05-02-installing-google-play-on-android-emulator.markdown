---
layout: post
title: "Installing Google Play on Android Emulator"
date: 2012-05-02 21:40
comments: true
categories: 
---
### The problem

I wanted to install Google Play on emulator. Problem is no guide on the net could tell me precisely how to do this. I'm not sure why, it seemed that they worked with old Android Market (pre version 3 era) or at least people shown some screenshots of that but I couldn't make it work at home.

The difference was that I tried to install Google Play on an Ice Cream Sandwitch emulator (API level 15) and it always failed.

### First attempts

I first found [this site](http://blog.varunkumar.me/2010/11/how-to-install-android-market-in-google.html). I tried setting up everything from there but I just couldn't make it work. Possible because their Vending.apk couldn't start, I have no clue. Then I found [this site](http://www.tech-recipes.com/rx/10004/accessing-android-market-from-android-sdk/) and I somehow managed to install Vending.apk on emulator using this guide, but I couldn't reproduce that.

One of the problems with those guides (which made me spend a lot of time on something useless) was that both these guides made me delete two files from AVD: cache.img and userdata-qemu.img. Why? Apparently when you wanted to restart emulator, these files were responsible for reinitialization of emulator to it's starting state (like full wipe/factory reset). I tried and tried and I always ended up without my changes.

### New Hope

After that I found that there is a snapshot option in AVD configuration window so I just checked it and finally I was able to see last state of emulator. Maybe it's not perfect - it works like hibernation on your laptop - but it works and after a whole day of fight it was enough.

I then though why do I install some obsolete Vending.apk and GoogleServicesFramework.apk? Why can't I have a new one there? So I connected my phone and pulled those apks from it. Of course it didn't work, but I was not sure why. But I remembered something - ~~CyanogenMod has Google apps packed in a zip~~ (UPDATE: CyanogenMod does no longer host Google Apps, but you can [find them here](http://goo.im/gapps/)! So I downloaded their package and extracted everything I needed - Vending.apk and GoogleServicesFramework.apk. But it was not enough.

So I had an AVD set up with snapshots enabled and I proceeded to installing my new apks - full script will be posted at the end.

    1. You have to remount system partition in read-write mode.
    2. Then it's necessary to allow writing to /system/app directory
    3. After that is done just push apks to /system/app/. (notice the dot at the end) 
    4. Just for the sake of other guides I also remove /system/app/SdkSetup* (both apk and odex)

Voila! Google Play is installed on emulator! But it still doesn't work. In order for it to start working, you need an account - Google account. This step also cost me some time. My AVD was based on Android 4.0.3, not the Google APIs one. So I changed it and... still nothing.

Apparently in recent releases Google decided to decouple their account provider from the OS so they have a separate apk that allows configuration of Google accounts - it's called GoogleLoginService.apk. Pushing this to emulator solved all the problems - you don't even need Google APIs (you can set up against Android 4.0.3 option). Now when you enter Play Store you get prompted for an account and it works!

### Conclusion

After two days of trying I finally was able to launch and browse Google Play Store on an Android emulator with Ice Cream Sandwitch. Phew.

Command I use to start my emulator
{% codeblock %}
emulator -avd am -partition-size 200 -no-audio -no-boot-anim
{% endcodeblock %}
Full script (assumes you have created and started your AVD)
{% codeblock %}
adb shell mount -o remount,rw -t yaffs2 /dev/block/mtdblock0 /system
adb shell chmod 777 /system/app
adb push GoogleLoginService.apk /system/app/.
adb push GoogleServicesFramework.apk /system/app/.
adb push Vending.apk /system/app/.
adb shell rm /system/app/SdkSetup*
{% endcodeblock %}
That script will work as long as you run it from a dir in which you have all the necessary apks. Of course you have to have your PATH environment variable set correctly (so that all those commands are actually accessible) but I suppose you know that already.

### Settings of the AVD
{% img /images/avdsettings.png Settings of the AVD %}
### Working Google Play Store
{% img /images/emulator.png Working Google Play Store %}