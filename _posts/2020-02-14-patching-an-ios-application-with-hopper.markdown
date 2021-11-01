---
layout: post
title:  "Patching an iOS Application with Hopper"
date:   2020-02-14 14:25:21 -0500
categories: jekyll update
published: true
author: "Cole Roberts"
categories: "reverse-engineering"
permalink: /:categories/:path
---

In here are many tutorials on Hopper that do a much better job of explaining its intricacies. I will list a few resources at the bottom of this tutorial. The goal of this tutorial is to simply patch an application and re-install it to your phone. I will be using a naive jailbreak detection application just to keep things interesting. I will also be using the premium version of Hopper for this tutorial. If you donâ€™t have it, follow along knowing that all of this is possible if you come across an extra $100.

First you should clone the [Naive Jailbreak Detection](https://github.com/klmitchell2/NaiveJailbreakDetection) repo to your machine and run the application on your phone. The app will look like this after pressing the button:

![](https://citadelsh.s3.us-west-2.amazonaws.com/bac98a82-6b1e-513c-9f51-2ef061293905.jpeg)

Then, as explained in [Part  1](https://www.citadel.sh/blog/decrypting-an-ios-appstore-application), decrypt the application and open its `NaiveJailbreakDetection` executable with Hopper. Make sure you save the `Payload` folder, weâ€™ll be using it later. Once this is done you should see something very similar to this:
![](https://citadelsh.s3.us-west-2.amazonaws.com/fb61526d-34e7-52f0-beca-33d1f8e8192f.png)


When Iâ€™m looking for jailbreak detection within an application I usually start out by simply searching for the word `jail` in Hopper: 
![](https://citadelsh.s3.us-west-2.amazonaws.com/fe68d3f6-6fb5-5bc3-9e62-d0fe63efbb3f.png)

By looking at the instructions, we can see that `isJailBroken` is being called. I'm willing to bet that this is a function that returns a `BOOL`, but instead of changing this value (we'll do this in another tutorial with Cycript) we are simply going to bypass the call all together by `NOP`ing out the region with Hopper.
![](https://citadelsh.s3.us-west-2.amazonaws.com/c60fc535-703c-588e-8711-e0cfc9697e34.png)

Specifically we are going to eliminate the `bl` command. `bl` stands for branch with link, and we can think of this as the underlying instruction that will call the `isJailBroken` method. You can find a full list of ARM instructions [here](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.den0024a/ch05s01.html)
![](https://citadelsh.s3.us-west-2.amazonaws.com/c60fc535-703c-588e-8711-e0cfc9697e34.png)

With this instruction highlighted go to `Modify`->`NOP Region` and you will see the following:

![](https://citadelsh.s3.us-west-2.amazonaws.com/f59f2353-f8c4-5b93-a3e5-4462dbc3746c.png)

Next go to `File` -> `Produce New Executable` and save the executable. Take the executable and move it into `Payload`->`NaiveJailbreakDetection`, replacing the original executable. Zip the payload folder and rename it `Payload.zip` to `NaiveJailBreakDetection.ipa` 

Now you will need to download Cydia impactor from [here](http://www.cydiaimpactor.com) , weâ€™ll use this to install our `.ipa` onto the device. 

Plug your iPhone into your desktop and open Impactor.app. 
![](https://citadelsh.s3.us-west-2.amazonaws.com/a49a9191-4005-5793-8a74-314f341f187e.png)

Drag and drop `NaiveJailBreakDetection.ipa` onto it. You will be prompted to provide and Apple ID and a password from appleid.apple.com. Sign into this website with an Apple ID and generate a password. Type this password (with out `-`) and you the installation process with begin. The patched application will now be on your iPhone. 

Running and pressing the jailbreak detection button will look like this: 
![](https://citadelsh.s3.us-west-2.amazonaws.com/921a8034-1db2-521c-83cc-bec8700c168f.jpeg)

**Links**
- [https://www.youtube.com/watch?v=YcfuQY5z_-A](https://www.youtube.com/watch?v=YcfuQY5z_-A)
- [https://resources.infosecinstitute.com/ios-application-security-part-28-patching-ios-application-hopper/#gref](https://resources.infosecinstitute.com/ios-application-security-part-28-patching-ios-application-hopper/#gref)
