---
layout: post
title: "Getting auth token on an Android device"
date: 2012-05-01 00:18
comments: true
categories: 
---
I'm currently working on a little pet project that targets Android devices (well, at least my Galaxy) and requires auth token to operate. It wasn't hard to find the code to do this: [Toxic Bakery has already shown how to do it.](http://www.toxicbakery.com/android-development/getting-google-auth-sub-tokens-in-your-android-applications/comment-page-1/#comment-238)

My first attempt of using this code was successful and running on the device was easy - I simply added a button that invoked their method and it worked. However when I used this solution in my pet project it suddenly stopped working properly. The problem was happening when I was invalidating the auth token - recursive call would fail with IllegalStateException saying that calling that method on main thread would lead to a deadlock.

At first I didn't know what to do - I am, after all, Android newbie. I spent some time trying to find out why my first code worked and second didn't and I found a single difference - I used minSdkVersion in AndroidManifest.xml for my pet project. I have no idea why this happens, I don't even know where to start looking. But I created a simple workaround to solve the problem - I used AsyncTask to create a background loader and it works very well now, not to mention I've had a good time reading about threading in Android.

The project with workaround can be found at [GitHub](https://github.com/piotrbudaeu/android-authtoken)