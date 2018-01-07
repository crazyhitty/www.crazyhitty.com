---
layout: post
title: Automating Google Play Publishing - Part 3
categories: [tutorial]
comments: true
published: true
---

For uploading APK to google play automatically, we can either use the API's provided by google itself to manually upload APK or use an existing solution. Good thing for us is that there is already an existing open source gradle plugin for publishing apps to google play, so we don't need to get our hands dirty. 

<!--more-->

The plugin we are going to use is named **[gradle play publisher](https://github.com/Triple-T/gradle-play-publisher)** that will automate all this stuff for us.

## Including the plugin in your project

Add this to your project level build.gradle file.

{% highlight gradle %}
buildscript {
    repositories {
        ...
    }
    dependencies {
        ...
        classpath 'com.github.triplet.gradle:play-publisher:1.2.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
{% endhighlight %}

And, in your app (or module) level build.gradle file.

{% highlight gradle %}
apply plugin: 'com.android.application'
apply plugin: 'com.github.triplet.play'

android {
    playAccountConfigs {
        defaultAccountConfig {
            serviceAccountEmail = "service_account_email_here"
            jsonFile = file("someDirectory/my_json_key.json")
        }
    }

    defaultConfig {
        ...
        playAccountConfig = playAccountConfigs.defaultAccountConfig
    }

    signingConfigs {
        release {
            storeFile file("someDirectory/my_keystore.jks")
            storePassword "my_store_pass_here"
            keyAlias "my_key_alias_here"
            keyPassword "my_key_pass_here"
        }
    }

    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}

play {
    track = 'production'
}
{% endhighlight %}

You can also modify the `play` section according to your own needs.

{% highlight gradle %}
play {
    track = 'production' // or 'rollout' or 'beta' or 'alpha'
    userFraction = 0.2 // only necessary for 'rollout', in this case default is 0.1 (10% of the target)
    untrackOld = true // will untrack 'alpha' while upload to 'beta'
}
{% endhighlight %}

Here is the description about properties used in `play` section.

| Property                                                                                     | Description                                                                                                                                                                                                                                           |
|----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [track](https://github.com/Triple-T/gradle-play-publisher#specify-the-track)                 | Defines on which channel the APK should be published. Can be one of the **production**, **beta**, **alpha** or **rollout**.                                                                                                                           |
| [userFraction](https://github.com/Triple-T/gradle-play-publisher#specify-the-track)          | Rollout the APK for some users only. Default value is 0.1 (which means 10%). This property is necessary if you are publishing with `track = 'rollout'`.                                                                                               |
| [untrackOld](https://github.com/Triple-T/gradle-play-publisher#untrack-conflicting-versions) | Doesn't track for old APKs for version conflicts.                                                                                                                                                                                                     |
| [errorOnSizeLimit](https://github.com/Triple-T/gradle-play-publisher#text-requirements)      | Plugin checks if the provided details like **title**, **short description**, **long description** and **recent changes** are not exceeding their required limit. You can disable this check by setting this property to false. By default it is true. |
| [uploadImages](https://github.com/Triple-T/gradle-play-publisher#upload-images)              | Upload images for Google Play Store listing. By default it is false.                                                                                                                                                                                  |

**PS:** In the [previous blog]({{ site.baseurl }}{% post_url 2017-12-27-automating-google-play-publishing-part2 %}) we discussed about how to create a service account and json key file. These properties are required inorder to authenticate with your google play account. Please check the previous blog to know more about these properties.

## Preparing for release 🚀

Inorder to prepare a release, first we need to setup the appropriate metadata which will be used by the plugin.

![](../../img/google_play_publish_directory_structure.png)


You can either create these directories and files manually or use this gradle task to create them automatically. Make sure you are in your project directory while running this command.

{% highlight terminal %}
$ ./gradlew bootstrapReleasePlayResources
{% endhighlight %}

This would create all the resources required in order to publish your APK.

###### Play Store Metadata Structure


```
- [src]
  |
  + - [main]
      |
      + - [play]
          |
          + - [en-US] -> Locale in which your content would be published
          |   |
          |   + - [listing] -> Details about play store listing
          |   |   |
          |   |   + - fulldescription	
          |   |   |
          |   |   + - shortdescription
          |   |   |
          |   |   + - title
          |   |   |
          |   |   + - video
          |   |
          |   + - whatsnew -> For showing changelog/features of the latest update
          |
          + - [de-DE]
          |   |
          |   + - [listing]
          |   |   |
          |   |   + - fulldescription
          |   |   |
          |   |   + - shortdescription
          |   |   |
          |   |   + - title
          |   |   |
          |   |   + - video
          |   |
          |   + - whatsnew
          |
          + - contactEmail
          |
          + - contactPhone
          |
          + - contactWebsite
          |
          + - defaultLanguage -> Default locale for publishing. (For example: en-US)
```

For more info about play store metadata, [check the plugin's readme section](https://github.com/Triple-T/gradle-play-publisher#play-store-metadata).

After setting it up, we can finally publish the app using this plugin.

{% highlight terminal %}
$ ./gradlew publishApkRelease
{% endhighlight %}

This command will create a signed APK for your project and then publish it on google play with the summary of recent changes (whatsnew). Your release would now be available on Google Play Developer Console. Just check it once to make sure it release was successful or not.

**PS:** Make sure to bump up versionCode and versionName in your `build.gradle` file.

###### Gradle tasks provided by this plugin

| Task                          | Description                                                                                                      |
|-------------------------------|------------------------------------------------------------------------------------------------------------------|
| publishApkRelease             | Uploads signed APK and the summary of recent changes.                                                            |
| publishListingRelease         | Uploads only images and description.                                                                             |
| publishRelease                | Uploads everything including signed APK, recent changes, images and description.                                 |
| bootstrapReleasePlayResources | Retrieves current listing details from Google Play and assemble it properly in directories and files (metadata). |

That's it for now folks. Now, you know how to publish signed APK with just a single gradle task. In next part, we will learn how to integrate CI and move the publishing part to CI.