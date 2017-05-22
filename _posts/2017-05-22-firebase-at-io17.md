---
layout: post
title: IO 2017: What's new in Firebase
comments: true
twitter: false
---

Great times are coming for Firebase. There were [25 talks](https://events.google.com/io/schedule/?section=may-17&track=firebase) about Firebase in the past #io17.

This is a quick bullet point summary of all the new stuff already available or coming in the next weeks

### Fabric

* Crahslytics is going to be integrated into Firebase as the main crash reporting solution.
* Digits technology will allow **phone number authentication** in Firebase, free for up to 10,000 verifications - which should cover most of the regular apps out there.

Phone authentication is available now on [iOS](https://firebase.google.com/docs/auth/ios/phone-auth) and [Web](https://firebase.google.com/docs/auth/web/phone-auth) and comming to [Android](https://firebase.google.com/docs/auth/android/phone-auth) in the next weeks.

> Digits SDK will be **deprecated** in the coming weeks, so keep an eye on [Digits blog](http://get.digits.com/blog/introducing-firebase-phone-authentication) for more info on migration tools.

### Opensource
* All Firebase SDKs are now open source https://opensource.google.com/projects/firebase-sdk

### Firebase Cloud Functions
* Cloud functions now [integrate with Hosting](https://firebase.google.com/docs/hosting/functions) to provide full dynamic webapps.

### Firebase Storage

* You can now map existing storage buckets into your Firebase project.
* The region in which your data is going to be stored now can be chosed. This is great for legal & performance reasons.

### Firebase Database

* Expanding to 100,000 simultaneous connections.
* Introducing [profiler](https://firebase.google.com/docs/database/android/profile) to allow you to introspect bandwidth and latency at path level.

### Performance Monitor (beta)
* New dashboard to get insights about your app network response latency, success rate, payload size, etc.
* Requires you to add a new sdk dependency into your app https://firebase.google.com/docs/perf-mon/
* It uses something called _traces_ to measure time based events. Some are already defined but you can add your own.

### Test Lab

* Adding first-class support for games: game tests can now run on test lab.
* Test lab now allows you to simulate different network conditions, like 4G, 3G, or slower ones.
* Expanding device seleccion to Google Pixel and Galaxy S7.
* Access to the lates version of Android O.

### Firebase Alpha
* You can now sign up to get the new stuff before it's released and provide feedback about it at https://services.google.com/fb/forms/firebasealphaprogram/


## See more
* [The Firebase Blog](https://firebase.googleblog.com/2017/05/whats-new-from-firebase-at-google-io.html)
* [Google IO 2017 What's new in Firebase session](https://www.youtube.com/watch?v=m7a26ymUu2U&t=2s&list=PLl-K7zZEsYLma7gxYxtEwO1rsAPn7wkV_&index=1)

   