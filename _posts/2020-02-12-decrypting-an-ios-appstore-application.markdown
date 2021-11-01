---
layout: post
title:  "Decrypting an iOS AppStore application"
date:   2020-02-12 14:25:21 -0500
categories: jekyll update
published: true
author: "Cole Roberts"
categories: "reverse-engineering"
permalink: /:categories/:path
---

In this tutorial we'll be using an iPhone 7 running iOS 11.1 and [Electra 1.0.4](https://coolstar.org/electra/). We'll also be using [bfinject](https://github.com/BishopFox/bfinject) to decrypt an iOS AppStore application. 

**NOTE:** When using bfinject for the first time, we recieved `[!] Unknown jailbreak. Aborting.`. Upon further inspection we noticed there were several forks of bfinject that fixed this and other dependency issues. We used [this fork of bfinject](https://github.com/klmitchell2/bfinject), which looks for dependencies in the updated paths. Additionally, `/u/_exgen_` [made a fork](https://github.com/MJavad/bfinject) that not only incorporates those updates, but it also fixes the issue of having to turn off tweaks in `Electra.app`. 

Before you jailbreak your device make sure tweaks are turned off in `Electra.app`:

You'll want to clone this repo to your machine and make a `bfinject/` folder on your device (we used `/private/var/root`), and move `bfinject.tar` to the `bfinject/` folder on your device. Next extract the tarball with `tar xvf bfinject.tar` and you'll have a `bfinject` executable and various other files and dependencies. 

![](https://i.imgur.com/VGZAol0.png)

In `bfinject/` run `bash bfinject` and you should see this:  
![](https://i.imgur.com/tiVlGjt.png)

Next, make sure the application is running and use `bash bfinject -P SomeApp -L test` and you'll see this: 
![](https://i.imgur.com/1uxG4Ks.png)

And on the device:
![](https://i.imgur.com/PaAv6K9.png)

**NOTE:** If you see and error like `md5sum: command not found` or `tail: command not found`, it could mean that these utils are missing in `/user/bin`. In order to fix this, you will need to find those utils (I had these on another jailbroken device). Once you acquire the executable, compress it and move it `/usr/bin` and give it exectuable permissions. 

CoolStar has a `Core Utilities(/bin)` repo, but even with it installed we still had utilities missing on a jailbroken device. If any one else has ran into this problem and has a different solution, please reach out. 

We now know that bfinject is working properly and we can start leveraging it's tools. In order to decrypt an application and view it in Hopper/IDA you can use the following command: `bash bfinject -P SomeApp -L decrypt`:  
![](https://i.imgur.com/57y6SS0.png)

And on the device: 
![](https://i.imgur.com/zHgV2b1.png)
![](https://i.imgur.com/G6Nba0T.png)

Next you can use FileZilla or NetCat (above) to move `decrypted-app.ipa` back onto your machine for static analysis via Hopper/IDA. You will find this file on your device at: 
```/var/mobile/Containers/Data/Application/6E6A5887-8B58-4FC5-A2F3-7870EDB5E8D1/Documents/```

Take `decrypted-app.ipa` and rename it as `decrypted-app.zip`. Unzip and you will now have a `Payload` folder and inside right click and view the decrypted contents of the app inside. You can now use Hopper/IDA to open up the `SomeApp.app/SomeApp` binary. 

In Part 2 we will patch an iOS app.  