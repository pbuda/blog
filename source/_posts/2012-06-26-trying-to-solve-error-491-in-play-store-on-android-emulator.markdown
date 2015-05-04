---
layout: post
title: "Trying to solve Error 491 in Play Store on Android Emulator"
date: 2012-06-26 09:23
comments: true
categories: 
---
Installing Google Play Store on Android emulator is quite easy - I described it in my [previous article](/2012/05/installing-google-play-on-android-emulator.html). However there is one problem that does not allow downloading of apps - the dreadful Error 491.
After some investigation it appears that the error happens because Android emulator does not have DRM! This is the exact error that is displayed in Logcat:

{% codeblock %}
06-25 07:51:42.142: WARN/DownloadManager(342): Exception for id 7: android/drm/DrmManagerClient
        java.lang.NoClassDefFoundError: android/drm/DrmManagerClient
        at com.android.providers.downloads.DownloadDrmHelper.isDrmMimeType(DownloadDrmHelper.java:45)
        at com.android.providers.downloads.Helpers.checkCanHandleDownload(Helpers.java:148)
        at com.android.providers.downloads.Helpers.generateSaveFile(Helpers.java:81)
        at com.android.providers.downloads.DownloadThread.processResponseHeaders(DownloadThread.java:563)
        at com.android.providers.downloads.DownloadThread.executeDownload(DownloadThread.java:256)
        at com.android.providers.downloads.DownloadThread.run(DownloadThread.java:172)
{% endcodeblock %}

I've tried a few things to fix this, but nothing really helped. The CyanogenMod Google Apps package contained some DRM jar, but it didn't help at all. So I've decided to browse my Galaxy S2 and found this _/system/app/DrmProvider.apk_ which I thought might help. So I pulled it from device and pushed it to the emulator. As expected, it kind of worked, but it now threw another exception:

{% codeblock %}
06-25 07:53:52.043: WARN/DownloadManager(724): Exception for id 8: null
        java.lang.ExceptionInInitializerError
        at com.android.providers.downloads.DownloadDrmHelper.isDrmMimeType(DownloadDrmHelper.java:45)
        at com.android.providers.downloads.Helpers.checkCanHandleDownload(Helpers.java:148)
        at com.android.providers.downloads.Helpers.generateSaveFile(Helpers.java:81)
        at com.android.providers.downloads.DownloadThread.processResponseHeaders(DownloadThread.java:563)
        at com.android.providers.downloads.DownloadThread.executeDownload(DownloadThread.java:256)
        at com.android.providers.downloads.DownloadThread.run(DownloadThread.java:172)
        Caused by: java.lang.UnsatisfiedLinkError: Library drmframework_jni not found; tried [/vendor/lib/libdrmframework_jni.so, /system/lib/libdrmframework_jni.so]
        at java.lang.Runtime.loadLibrary(Runtime.java:393)
        at java.lang.System.loadLibrary(System.java:535)
        at android.drm.DrmManagerClient.<clinit>(DrmManagerClient.java:56)
        ... 6 more
{% endcodeblock %}

So I dig deeper and tried extracting missing .so files. Browsing my phone revealed four files that I might be interested in: _libdrm1.so_, _libdrm1_jni.so_, _libdrmframework.so_ and _libdrmframework_jni.so_. Again, pulled those from device and pushed to emulator but I got the first exception and got stuck.

At this point I don't have any more ideas how to solve this problem, but I will still try. It appears that installing DRM on emulator might be much more problematic than pulling some apks from real devices.