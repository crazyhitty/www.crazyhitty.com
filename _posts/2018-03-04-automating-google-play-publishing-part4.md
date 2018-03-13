---
layout: post
title: Automating Google Play Publishing - Part 4
categories: [tutorial]
comments: true
published: false
---

In the previous blog post we learned how to publish APK easily using a single gradle task. But what if you want to offload the build part also? That's where CI (Continuous Integration) comes in.

<!--more-->

## What is CI?

From wikipedia

> In software engineering, continuous integration (CI) is the practice of merging all developer working copies to a shared mainline several times a day.

In broader terms, CI is used to automate all the build related workload over a remote machine so that developers can build their projects without setting up the development environment. We can create our own CI server or we can use any existing CI service.

For this tutorial, we will be using [Travis CI](link to travis ci) as it provides android build support out of the box and is free for open source projects.

**Note:** Almost every CI uses a similar syntax for its configuration. So, using other CI systems won't be a big issue.

## Setting up .travis.yml

Create an empty file with name `.travis.yml` in your project directory.

Travis CI then will check if your project contains this file or not. If the project contains this file, then it will parse that file and initiate a build.

**Note:** `.travis.yml` file stores all the build related information.

**Basic android specific .travis.yml format:**

{% highlight .travis.yml %}
# Specify platform, in our scenario it would be android.
language: android

# Specify branches for build, "only" tag is used to initiate builds only for particular branches.
# In this scenario, only master and develop branches will initiate the build.
branches:
  only:
  - master
  - develop

# Specify the required components by android platform.
android:
  components:
  - tools
  - platform-tools
  - tools
  - build-tools-26.0.2
  - android-26
  - extra-google-google_play_services
  - extra-google-m2repository
  - extra-android-m2repository
  - addon-google_apis-google-19
  - sys-img-armeabi-v7a-addon-google_apis-google-26
  licenses:
  - android-sdk-license-.+

# The actual script which would be executed when a build is initiated.
script:
  - "./gradlew assembleRelease"
{% endhighlight %}

When you push your commits with this configuration, Travis CI will setup an android environment and execute the script, which is just basically gradle's assembleDebug task. Also, the build will only be initiated when commits are pushed to branches named `master` and `develop`, you can remove this restriction by removing the branches section from `.travis.yml` and then Travis CI will create build whenever the commits are pushed for any branch.

## Publishing to google play from Travis

Publishing would require us to create a signed APK and then use google play publish API to create a release on google play. We have already discussed about all of that stuff in previous parts of this tutorial series.

In the [previous part]({{ site.baseurl }}{% post_url 2018-01-05-automating-google-play-publishing-part3 %}) we learned that using a gradle plugin we can directly publish the app on google play and the command to do that would be:

{% highlight terminal %}
$ ./gradlew publishApkRelease
{% endhighlight %}

But before we do that we need to encrypt important files first.

## Encrypting sensitive data for Travis

Without encryption your sensitive data can be used by anyone which makes it very easy for anyone to create the same application with same certificates, which is very scary. Also, it is very helpful for open source projects where you cannot directly put your release keys.

**Things to encrypt:**

1. Release keystore file.
2. Release keystore credentials.
3. Google Play service account key file.
4. Google Play service account credentials.



