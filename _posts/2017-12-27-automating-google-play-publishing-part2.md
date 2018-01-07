---
layout: post
title: Automating Google Play Publishing - Part 2
categories: [tutorial]
comments: true
published: true
---

Inorder to publish the release build on google play we need to setup our developer account.

<!--more--> 

## Setting google play developer account for publishing.

After logging into google play developer account, go into `Settings > Developer account > API access`

Now, link `Google Play Android Developer` under Linked Project section.

![](../../img/google_play_api_access.png)

After linking the account, create a OAuth Client and Service Account. Creating OAuth Client is very easy, just click on `CREATE OAUTH CLIENT` button and it will create an OAuth Client. And, for Service Account, after clicking on `CREATE SERVICE ACCOUNT`, it will show a dialog indicating that you need to visit `Google API Console` page and manually create a service account.

![](../../img/google_play_api_access_create_service_account.png)

On `Google API Console` page, click the button on top named `Create Service Account`. Set the service account name and select the role as `Service Account Actor` which can be found under `Project` section in `Role` dropdown. Make sure to tick the checkbox `Furnish a new private key` and select `JSON` as key type. After all this, clicking on `CREATE` button will create this service account and download a json file which will contain service account details. Keep this json file somewhere safe as it would be required for authenticating with Google Play Access API later on.

![](../../img/google_api_console_service_account.png)

This will create a service account for your Google Play Developer Account. Make sure to check the API Access page again to make sure that the created service account is appearing on that page.

That's all for now, in next part we will do the actual stuff and upload your signed apk on google play :)