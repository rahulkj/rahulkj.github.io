---
layout: post
title:  "Hosting iOS and Android apps from your own portal"
date:   2013-04-23 12:10:00 -0600
categories: mobile apps
---

Want to host your iOS and Android applications for development and QA testing?

The first thought that comes is are there any sites which can help me achieve this.
If not, then how do I share my applications to my mobile testers.

Distributing the android application is not tough. You can email the application (`.apk` file), or host it on your website.
The true challenge comes when we deal with iOS applications (`.ipa` file).

There are many websites which allow you to host applications, and securely distribute them to the users. But this comes at some price $XX. If the sites are free, you do not want to share your UDID's of your iOS devices on those sites.

To build an application to host your `ipa` and `apk` files, you can perform the following:

- Create a simple web application project
- Download the iOS Beta Build from [here](http://media.ratevegas.com/BetaBuilder.app.zip)
- Extract the contents and run the BetaBuilder app
- Point the app to your `ipa` file, and export the contents to the webapps folder of your web application
- Modify the `index.html` to add a download link to your `apk` file (android application)
- Build the war and deploy it on your tomcat (or any preferred application container).

Access the link from your mobile and download the applications. You are all set!

P.S.: I do not have screenshots to show the above work flow. There are various links out there which demonstrate this.
