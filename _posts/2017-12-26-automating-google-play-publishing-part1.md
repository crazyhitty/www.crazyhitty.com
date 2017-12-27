---
layout: post
title: Automating Google Play Publishing - Part 1
categories: [tutorial]
comments: false
---

Manual publishing is a chore and a waste of time as it involves a lot of steps which are redundant and can be easily automated.

<!--more-->

**Manual publishing requires these steps:**

1. Create a signed APK for your app.
2. Login to the google play developer account.
3. Select the app and create a release.
4. Upload the APK with changelog.
5. Submit the update.
6. Repeat the above 5 steps again for a new release.

Doing this again and again can become quite irritating after a while. Also, if you are uploading multiple APKs for every release then manual publishing would only lead to headaches.

So, automating this part can help save you some time which you can use to procrastinate ;)

**Prerequisites:**

1. Google play developer account.
2. An already listed app in your developer account.
3. CI (Continuous Integration) tool of your choice. I will use Travis CI for this tutorial as it is free to use with open source projects.

## Signing your APK properly

In order to create a signed build of your app, you can either use AndroidStudio's inbuilt UI (Build > Generate Signed APK...) or specify your keystore details in your `build.gradle`. We will use the latter as it is will help us automate the signing process for our app.

Add this snippet to your build.grade's android section:



You can now run `./gradlew assembleRelease` command to create a signed APK directly.

In the next part, we will discuss how to upload your signed APK directly to google play.

**PS:** In upcoming parts of this blog series, I will also explain how to secure your keystore details for your open source projects.

**References:**

1. <https://medium.com/@harmittaa/travis-ci-android-example-357f6e632fc4>