---
layout: post
title:  "Reverse Engineering Ionic Applications"
date:   2020-08-07 14:25:21 -0500
categories: jekyll update
published: true
author: "Cole Roberts"
categories: "reverse-engineering"
permalink: /:categories/:path
---
 In this guide we'll cover the complete reverse engineering process for Ionic applications. Hybrid applications built with tools like Ionic, PhoneGap, and Cordova are not typically built with security in mind. They expose the application's source code at the top level where it can be changed, recompiled, and run on any device. 

Let's get started by cloning the repository used in the example [here.](https://github.com/colealanroberts/ionic-reverse-engineering) This is identical to the container bundle that you'd find in any Ionic 2 or 3 application on the device. We're going to assume that these files were copied off the device. You'll need a jailbroken iPhone, if your phone isn't jailbroken you should follow [our simple guide ](https://www.citadel.sh/blog/jailbreaking-your-iphone-with-electra) on jailbreaking with Electra.

In the wild you'll need to SSH into the device and copy the application container from the device. This usually can be found at the directory: `/private/var/containers/Bundle/Application` You'll need to copy the respective application directory to your computer where you'll find all of the source code readily available.

Navigate to the folder `www` where you'll find all of the application source code. Open up `main.js` in your text editor of choice. It will likely be minified and uglified and should look something like this.

![https://citadelsh.s3.us-west-2.amazonaws.com/29eefc8d-de78-5ac6-8fa3-94ded1aa385c.png](https://citadelsh.s3.us-west-2.amazonaws.com/29eefc8d-de78-5ac6-8fa3-94ded1aa385c.png)

You'll want to install a beautifier or use an online tool. VSCode has a Beautifier package which works well and immediately gives us a readable file.

If we search `signIn` or another common method name for authentication we'll find this:

![https://citadelsh.s3.us-west-2.amazonaws.com/dbc1b0d4-6ac6-5e30-b337-b6f8c4dcc85a.png](https://citadelsh.s3.us-west-2.amazonaws.com/dbc1b0d4-6ac6-5e30-b337-b6f8c4dcc85a.png)

We notice a few API calls to a service called `mockApiService` which returns an error response if the username or password were incorrect.This is a mock API and for the purpose of this example it wil always return this error response. We also notice that on line 256 (may vary depending on beautify tool) the call to `n.navCtrl.push(f)`. If we find `f` we'll see that `return !!this.userService.isSignedIn()` is returned during a lifecycle event like `ngOnInit` or`ionViewCanEnter`.

Let's delete the line `return !!this.userService.isSignedIn()` and replace it with `return true`.

We should also delete the authentication checks in the `g` function and only include the push event. Here's our modified main.js file:

![https://citadelsh.s3.amazonaws.com/b0169b80-6890-5c11-88bb-7599c69a3d71.png](https://citadelsh.s3.amazonaws.com/b0169b80-6890-5c11-88bb-7599c69a3d71.png)

Next create a new folder anywhere on your computer and run `ionic start appName blank`. When prompted to integrate with iOS and Android type `y` and continue. When the initial process is complete run `ionic cordova build ios`. If you receive a warning about provisioning profiles open the iOS project in Xcode at `app-folder/platforms/ios` select your Signing Team and Provisioning Profile and run the build command again. If the build is succesfull you'll see something similar to this.

![https://citadelsh.s3.us-west-2.amazonaws.com/cb816df4-b94e-567f-852e-1ef60318a1e3.png](https://citadelsh.s3.us-west-2.amazonaws.com/cb816df4-b94e-567f-852e-1ef60318a1e3.png)

Next copy _all_ of the files in your modified folder `www` folder to your newly generated blank app folder: `app-name/platforms/ios/www/build/` overwriting the existing files. Then rebuild the app using Xcode and run it on your device. If all is successful you should be able to tap the signin button and circumvent any authorization checks. This is a proof of concept but shows you how to succesfully reverse engineer an Ionic 3 app.