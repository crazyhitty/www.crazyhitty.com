---
layout: post
title: Automating Google Play Publishing - Part 4
categories: [tutorial]
comments: true
published: true
---

In the previous blog post we learned how to publish APK easily using a single gradle task. But what if you want to offload the build part also? That's where CI (Continuous Integration) comes in.

<!--more-->

## What is CI?

From wikipedia,

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

When you push your commits with this configuration, Travis CI will setup an android environment and execute the script, which is just basically gradle's assembleDebug task. Also, the build will only be initiated when commits are pushed to branches named `master` and `develop`, you can remove this restriction by removing the `branches` section from `.travis.yml` and then Travis CI will create build whenever the commits are pushed for any branch.

## Publishing to google play from Travis

Publishing would require us to create a signed APK and then use google play publish API to create a release on google play. We have already discussed about all of that stuff in previous parts of this tutorial series.

In the [previous part]({{ site.baseurl }}{% post_url 2018-01-05-automating-google-play-publishing-part3 %}) we learned that using a gradle plugin we can directly publish the app on google play and the command to do that would be:

{% highlight terminal %}
$ ./gradlew publishApkRelease
{% endhighlight %}

But before we do that we need to encrypt important files first.

## Encrypting sensitive data for Travis

Without encryption your sensitive data can be used by anyone which makes it very easy for anyone to create the same application with same certificates, which is very scary. Also, it is very helpful for open source projects where you cannot directly put your release keys. Encrypting will allow you to secure these files/keys and will only be decrypted when the travis CI build is being executed in the server.

**Things to encrypt:**

1. Release keystore file.
2. Release keystore credentials.
3. Google Play service account key file.
4. Google Play service account credentials.

Inorder to make things easier we will be using travis command line interface, which will provide us commands for encrypting and other stuff.

###### Setting up Travis CLI

You can easily install travis using this command, just make sure you have ruby/gem environment set up beforehand.

{% highlight terminal %}
$ sudo gem install travis
{% endhighlight %}

**Note:** You can setup ruby/gem environment using this [link](https://www.ruby-lang.org/en/documentation/installation/).

You can check if travis is installed or not by using `travis --version` command in your terminal.

###### Encrypting keys

There are 4 important keys that we need to encrypt:
1. Release keystore password.
2. Release keystore key alias.
3. Release keystore key password.
4. Google play service account email.

Before encrypting keys, make sure your current directory in terminal is same as your project's location, otherwise travis will not work as expected.

Encryption for keys can be performed using `travis encrypt SOMEVAR="secretvalue"` command like this:

<script src="https://asciinema.org/a/vq8nfoF6XJknipZosG3mSWOnw.js" id="asciicast-vq8nfoF6XJknipZosG3mSWOnw" async></script>

This would encrypt your store password. But you would still need to copy this encrypted key in your .travis.yml. So, your .travis.yml should look like this after adding all of the keys.

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

# This is where all your encrypted keys should be provided.
env:
  global:
  - secure: RNFro3a7k3INfB6y4hS9EC+R3ONn/iLkHra6d6WMkxs8POcLi5pnSL8V1sfAQZEdqaX9XBcPZ1Bdlzms0nEyjCWnrEoGlCjkIM68/UEJ2dVaP4lI+YAoZaeLGGq29HTZez0BObSf3ftG8w3eLN/3sUa7LxWKlWpP7Pr2AjA34tGrLo3ebSos5MVVzb4wMQpVJa7ilg/egPGTxEEq6FRpV8S/h4hv0+Sr6gdJld/Pzbi7Lt5Bt7V8KthH0+CV/eLh2nkrr9lrjRnkUqzefrcU2t1WW7mfZ5LWE1FG9+iZBoRV7NZPVxBvWXKzCstBMVoJFfCK+pldpXvp2ENLFXb9t5jtBXp8PfX5h8bmcKjfmcTS5eUOWfuiAKf4XP5Lg8dfV1yjzkyziOoUPAs5rIkeZkur2bSxpEq5Fy1ARzA+3Ra0/hpNvu9u73og+5LDCyGKXVDj2tDbIvrsc9fbXHtblxJjvAFDXzc+aksOcFhCYbhWRWGlkVe2ijCZJ9p4Tiq05KjenlUu8JzgRnNCTQtF9XR+bGMV+uwql6WSC3dNNhKkivRwiQkcTeokRwfds0jxkkOh2sXW62z86s2HqYQahkocQR68JE98UzsoMBux9gJWhkEsaPeHyLlUarXBIJndf+qjVyp972fOdtfJguEmiFTEAZ4u2SuFVjZfhBwrjMM=
  - secure: myStoreKeyAliasEncrypted
  - secure: myStoreKeyPasswordEncrypted
  - secure: myGoogleApiServiceAccountMailEncrypted
{% endhighlight %}

Later, we would also discuss how to access these keys. But before that, lets check out how to encrypt files.

###### Encrypting files

Now, we need to encrypt these files:
1. Keystore file.
2. Google play service account json file. [Click here for more info.]({{ site.baseurl }}{% post_url 2018-01-05-automating-google-play-publishing-part3 %})

First, paste both of these files inside your project directory and then follow these steps:

1. Create a .tar archive file consisting both of these files.
2. Then, encrypt the .tar file using `travis encrypt-file mySuperSecretKeys.tar`.
3. This would create one more file named `mySuperSecretKeys.tar.enc` which is the encrypted version of your files.
4. Now, just add the script in `before_install` section in `.travis.yml` to decrypt these files just before the build process starts.
 
**NOTE:** Make sure to remove the sensitive files after encryption if your project is open source and avoid committing those files.

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

# Just before installation, decrypt the files and extract the .tar files so that they can be directly accessed during the build.
before_install:
  - openssl aes-256-cbc -K $encrypted_400f62f18203_key -iv $encrypted_400f62f18203_iv -in mySuperSecretKeys.tar.enc -out mySuperSecretKeys.tar -d
  - tar xvf mySuperSecretKeys.tar

# The actual script which would be executed when a build is initiated.
script:
  - "./gradlew assembleRelease"

# This is where all your encrypted keys should be provided.
env:
  global:
  - secure: RNFro3a7k3INfB6y4hS9EC+R3ONn/iLkHra6d6WMkxs8POcLi5pnSL8V1sfAQZEdqaX9XBcPZ1Bdlzms0nEyjCWnrEoGlCjkIM68/UEJ2dVaP4lI+YAoZaeLGGq29HTZez0BObSf3ftG8w3eLN/3sUa7LxWKlWpP7Pr2AjA34tGrLo3ebSos5MVVzb4wMQpVJa7ilg/egPGTxEEq6FRpV8S/h4hv0+Sr6gdJld/Pzbi7Lt5Bt7V8KthH0+CV/eLh2nkrr9lrjRnkUqzefrcU2t1WW7mfZ5LWE1FG9+iZBoRV7NZPVxBvWXKzCstBMVoJFfCK+pldpXvp2ENLFXb9t5jtBXp8PfX5h8bmcKjfmcTS5eUOWfuiAKf4XP5Lg8dfV1yjzkyziOoUPAs5rIkeZkur2bSxpEq5Fy1ARzA+3Ra0/hpNvu9u73og+5LDCyGKXVDj2tDbIvrsc9fbXHtblxJjvAFDXzc+aksOcFhCYbhWRWGlkVe2ijCZJ9p4Tiq05KjenlUu8JzgRnNCTQtF9XR+bGMV+uwql6WSC3dNNhKkivRwiQkcTeokRwfds0jxkkOh2sXW62z86s2HqYQahkocQR68JE98UzsoMBux9gJWhkEsaPeHyLlUarXBIJndf+qjVyp972fOdtfJguEmiFTEAZ4u2SuFVjZfhBwrjMM=
  - secure: myStoreKeyAliasEncrypted
  - secure: myStoreKeyPasswordEncrypted
  - secure: myGoogleApiServiceAccountMailEncrypted
{% endhighlight %}

## Triggering a release build on travis CI server

After setting all of this up, whenever you push to `master` or `develop` branches, travis CI server would automatically trigger a release build. After the build is complete, it would
upload the generated APK to the Google Play Store. Also, during the entire build process you can keep an eye on travis console to check if everything is working as expected or not.
You can also cancel any ongoing build in travis if required.

You can go check out my [project](https://github.com/crazyhitty/Capstone-Project) where I have successfully integrated `travis CI` and `gradle play publisher` plugin for automating
google play publishing.

**NOTE:** From now on, be more careful while pushing to develop/master ;)