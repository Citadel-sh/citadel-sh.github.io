---
layout: post
title:  "Installing Cycript on MacOS High Sierra"
date:   2018-01-11 14:25:21 -0500
published: true
author: "Kevin Mitchell"
categories: "reverse-engineering"
permalink: /:categories/:path
---
I received errors when I first tried to install Cycript on my iMac so hopefully this is useful for others. 

First download cycript from [here](http://www.cycript.org). In terminal `cd` into `cycript_0` and run `./cycript`. If you recieve no errors skip the next part. 

When running this command on High Sierra 10.13.3 I recieved the following: 

```bash
dyld: Library not loaded: /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/libruby.2.0.0.dylib
  Referenced from: /Users/<iMacName>/Downloads/./cycript_0/Cycript.lib/cycript-apl
  Reason: image not found
Abort trap: 6
```

This error was really frustrating for me. I first tried to see how the path was different, and noticed that I only had a `/System/Library/Frameworks/Ruby.framework/Versions/2.3` path. Running `sudo mkdir -p /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/` resulted in `Operation not permitted`. 

After doing some digging I found a fix for this. It seems kind of hacky to me, so please comment below if you have any alternative solutions. 

1. Restart computer
2. Hold `command + r` while the computer is rebooting, this will bring you into recovery mode. 
3. Open terminal
4. Run: `csrutil disable`
5. Restart computer normally
6. Open terminal and run:
```
sudo mkdir -p /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ 
sudo ln -s /System/Library/Frameworks/Ruby.framework/Versions/2.3/usr/lib/libruby.2.3.0.dylib /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/libruby.2.0.0.dylib
```
7. Restart computer into recovery mode and run `csrutil enable`
8. Restart the computer once more and use as normally.  