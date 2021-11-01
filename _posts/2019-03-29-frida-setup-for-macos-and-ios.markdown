---
layout: post
title:  "Using Frida and Passionfruit with MacOS & iOS"
date:   2019-03-29 14:25:21 -0500
published: true
author: "Kevin Mitchell"
categories: "reverse-engineering"
permalink: /blog/:categories/:path
---
  I recently needed to setup a pentesting enviroment for an iPhone 6S Plus running iOS 11.0.3. I've jailbroken the phone on with Electra (iOS 11.0 - 11.1.2). I've been wanting to give [Frida](https://www.frida.re/docs/ios/) a shot so I figured this would be a perfect time for me to document the set up process. 

Overall Frida is a great tool to introspect (instrument) applications with. Developers have made a variety of toolkits that help cut down on manual work.

**In this tutorial ill show you how to:**
- Setup Frida on your iOS device 
- Setup Frida on macOS
- Commands via Frida & Frida Shell
- Passion Fruit iOS Analyzer 

**iOS Device Setup**

Start Cydia and add Frida's repository by going to `Manage -> Sources -> Edit -> Add` and enter `https://build.frida.re`. You should now be able to find and install the Frida package. 

**macOS Setup**

Download Frida to macOS by running this command: 
`sudo pip install frida-tools`, and we can confirm the installation by running
```shell
frida-ps -U
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/7512f4c1-46be-5ae8-8cad-da6080524a84.png)

**Commands via Frida & Frida Shell**

Lets connect the device via USB and see which processes are running
```shell
frida-ps -Uai
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/e1aba663-92d9-574e-a043-f152d170914b.png)

Next we are going to use the frida shell to retrieve the classes from an application. You can use the PID
```shell
frida-ps -U 686
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/c18c877b-24f6-5fb1-8230-a4f8455a3c6a.png)

Or you can use the name of the app
```shell
frida -U DVIA-v2
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/be24472a-be8c-5a8c-a4d3-2d132181d5a0.png)

Now lets grab the frist 20 Objective-C classes from the app with the following command:
```shell
Object.keys(ObjC.classes).slice(0, 20)
```
![](https://citadelsh.s3.us-west-2.amazonaws.com/e1beae6a-055b-58e2-8af9-48166b32437f.png)

And we can also get the method names of a class
```shell
ObjC.classes.AAAccountManager.$methods
```https://citadelsh.s3.us-west-2.amazonaws.com/52efcb67-4131-5de5-934f-c5e5b4f7f319.pnghttps://citadelsh.s3.us-west-2.amazonaws.com/e1beae6a-055b-58e2-8af9-48166b32437f.png)

It should be noted that when using `ObjC` we are only getting classes for Objective-C classes in the project. Frida explains this: 
`ObjC.available: a boolean specifying whether the current process has an Objective-C runtime loaded. Do not invoke any other ObjC properties or methods unless this is the case`

For development, we can place the above commands in a `.js` file
```shell
frida -U DVIA-v2 -l demo.js
```
![](https://citadelsh.s3.us-west-2.amazonaws.com/e3a0bf99-5029-55c4-b711-650915197a06.png)

We can also do live edits if we open the `.js` file in a text editor while the Frida shell is running. [When you save your file the edits will show up in the frida shell.](https://imgur.com/lOsRb45)


The javascript in the link prints out all of the classes within the application. You can find more examples here `https://github.com/coolx28/frida-scripts`

**Passionfruit Setup**

[Passionfruit](https://github.com/chaitin/passionfruit) is claimed to be a crappy iOS app analyzer, but this has been one of the most useful tools I have found. Its powered by Frida and allows you intraspect an entire iOS application. 

You can install Passionfruit via npm 
```shell
npm install -g passionfruit
```
</br>
And launch Passfruit with following command 
```shell
npm install -g passionfruit
```
</br>
Next open up a browser and navigate to `localhost:31337` and you will see the following 
![](https://citadelsh.s3.amazonaws.com/808d97e0-8c08-5ad2-8e91-e5da7fef3120.png)

Clicking on app, you will be able to see files, modules, classes, and local storage. Having all of this info available in one place is convenient, especially compared to writing your own scripts to retrieve this info. 

![](https://citadelsh.s3.us-west-2.amazonaws.com/6cccce71-f686-50f5-9f12-7815ee254c19.png)

 ↕date_created žñ>=f☺   y►  •_id [ð¿Á;Ñø ♦|9ô☻id %   5645efb2-63e4-50f1-aeaf-e35210d8c18a ♦images ♣    ☻slug $   frida-setup-for-ios-macos-and-tools ◘active ☺☻author_id %   26a0e8a6-e662-5d28-8c3d-cea4240b73ce ☻title &   Frida setup for iOS, macOS, and tools ☻content j☼  I recently needed to set up a pentesting enviroment for an iPhone 6S Plus running iOS 11.0.3. I've jailbroken the iPhone with Electra (iOS 11.0 - 11.1.2). I've also been wanting to give [Frida](https://www.frida.re/docs/ios/) a shot so I figured this would be a perfect time for me to document the set up process. 

Overall Frida is a great tool to instrument applications with. Developers have made a variety of toolkits that help cut down on manual work.

**In this tutorial I'll show you how to:**
- Setup Frida on your iOS device 
- Setup Frida on macOS
- Commands via Frida & Frida Shell
- Passionfruit setup

</br>
**iOS Device Setup**

Start Cydia and add Frida's repository by going to `Manage -> Sources -> Edit -> Add` and enter `https://build.frida.re`. You should now be able to find and install the Frida package. 

**macOS Setup**

Download Frida to macOS by running this command: 
`sudo pip install frida-tools`, and we can confirm the installation by running (make sure your iPhone is plugged in)
```shell
frida-ps -U
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/7512f4c1-46be-5ae8-8cad-da6080524a84.png)

**Commands via Frida & Frida Shell**

We can run the following command to see which processes are running
```shell
frida-ps -Uai
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/e1aba663-92d9-574e-a043-f152d170914b.png)

Next we are going to use the frida shell to retrieve the classes from an application. You can use the PID
```shell
frida -U 686
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/c18c877b-24f6-5fb1-8230-a4f8455a3c6a.png)

Or you can use the name of the app
```shell
frida -U DVIA-v2
```

![](https://citadelsh.s3.us-west-2.amazonaws.com/be24472a-be8c-5a8c-a4d3-2d132181d5a0.png)

Now lets grab the first 20 Objective-C classes from the app with the following command
```shell
Object.keys(ObjC.classes).slice(0, 20)
```
![](https://citadelsh.s3.us-west-2.amazonaws.com/e1beae6a-055b-58e2-8af9-48166b32437f.png)

And we can also get the method names of a class
```shell
ObjC.classes.AAAccountManager.$methods
```
![](https://citadelsh.s3.us-west-2.amazonaws.com/52efcb67-4131-5de5-934f-c5e5b4f7f319.png)

It should be noted that when using `ObjC` we are only getting classes for Objective-C classes in the project. Frida explains this: `ObjC.available: a boolean specifying whether the current process has an Objective-C runtime loaded. Do not invoke any other ObjC properties or methods unless this is the case.`


For development, we can place the above commands in a `.js` file
```shell
frida -U DVIA-v2 -l demo.js
```
![](https://citadelsh.s3.us-west-2.amazonaws.com/e3a0bf99-5029-55c4-b711-650915197a06.png)

We can also do live edits if we open the `.js` file in a text editor while the Frida shell is running. [When you save your file the edits will show up in the frida shell.](https://imgur.com/lOsRb45)


The javascript in the link prints out all of the classes within the application. You can find more examples [here.](https://github.com/coolx28/frida-script)

**Passionfruit Setup**

[Passionfruit](https://github.com/chaitin/passionfruit) is claimed to be a crappy iOS app analyzer, but this has been one of the most useful tools I have found. Its powered by Frida and allows you to intraspect an entire iOS application. 

You can install Passionfruit via npm 
```shell
npm install -g passionfruit
```
</br>
And launch Passionfruit with following command 
```shell
passionfruit
```
</br>
Next open up a browser and navigate to `localhost:31337` and you will see the following 
![](https://citadelsh.s3.amazonaws.com/808d97e0-8c08-5ad2-8e91-e5da7fef3120.png)

Clicking on app, you will be able to see files, modules, classes, and local storage. Having all of this info available in one place is convenient, especially compared to writing your own scripts to retrieve this info. 

![](https://citadelsh.s3.us-west-2.amazonaws.com/6cccce71-f686-50f5-9f12-7815ee254c19.png)
