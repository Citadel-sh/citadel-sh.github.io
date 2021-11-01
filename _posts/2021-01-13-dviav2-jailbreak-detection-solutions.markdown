---
layout: post
title:  "DVIAv2 Jailbreak Detection Solutions"
date:   2021-01-13 14:25:21 -0500
published: true
author: "Kevin Mitchell"
categories: "reverse-engineering"
permalink: /:categories/:path
---
[DIVA-v2](https://github.com/prateek147/DVIA-v2) is a widely distributed iOS application used in the mobile security space to demonstrate vulnerabilities within iOS. In this article we'll show you how to circumvent the jailbreak detection exercises. 

Jailbreak detection and circumvention can be implemented in a variety of ways. This post will teach you how to use the LLDB debugger and provides basic insight into the required ARM assembly needed to bypass DVIA-v2's jailbreak detection techniques. It should be noted that there are many ways to solve these challenges and this is not the only way.

In this post we will be focusing on the solution, rather than tool installation. Here are a couple of resources we used to get up and running. These are great posts in their own right:
- https://www.allysonomalley.com/2019/01/06/ios-pentesting-tools-part-4-binary-analysis-and-debugging/
- https://kov4l3nko.github.io/blog/2018-05-25-my-experience-with-lldb-and-electra-jb/

The following are instuctions for connecting to a process via LLDB, calcuate Address Space Layout Randomization ([ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization)) offset and creating breakpoints. We've listed these here because they are used with each solution and would be redundant otherwise.  

**Connect LLDB to a running process**
- Computer: `iproxy 1337 1337 &`
- iPhone: `ps aux | grep <App Name>`
- iPhone: `/electra/jailbreakd_client <PID from step 2> 1`
- iPhone: `/Developer/usr/bin/debugserver localhost:1337 -a <PID from step 2>`
- Computer: `lldb` 
- LLDB: `platform select remote-ios`
- LLDB: `process connect connect://localhost:1337`

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/c9d31369-d74f-5da6-bd81-2166e8536ac5.png)

**Calculating the ASLR Offset**
- LLDB: `image dump sections <App Name>`
- Calculate ASLR Offset (red - yellow)
    - hex calculator: https://www.miniwebtool.com/hex-calculator/?number1=0x0000000100cb0000&operate=2&number2=0x0000000100000000

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/c916258c-5c98-54ab-8287-e599e86e80e1.png)

**Setting breakpoint**
- LLDB: `br s -a 0x<ASLR Offset>+0x<memory adress from hopper>`

**Common LLDB Commands**
- `br list` list breakpoints
- `br del <break point number>` delete breakpoints
- `s` step through current function 
- `finish` Step out of the currently selected frame
- `c` finish the execution 
- `register write <register> <value>` write a value to a register
- `register read <register>` read the value of the register 
- `register read` read the value of all of the registers 

**Process that we will use below**
- Choose a memory address.
- Set breakpoints on the memory addresses + ASLR offset. 
- Observe the contents of the registers 
- Write values to the register to change the execution flow.  

**Jailbreak Test 1**

Following the outline, our first step is examine DVIA-v2's binary in Hopper.  Searching for `test1` we can see that `__T07DVIA_v232JailbreakDetectionViewControllerC20jailbreakTest1Tappedyyp` could be interesting. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/f7b49a8c-ab5a-5e0d-9c6b-35da08b6eb83.tiff)

Theres a `bl` command here which means `branch with link`. We should see what happens when the function branches. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/e5a77949-dd0f-5f47-bbaa-2bf5d71223dd.tiff)

The function returns after a variety of operations. We can see what is happening by stepping through the function as it performs these operations. First, set a breakpoint at the highlighted memory address. Next, step through the function several times (and read the contents of the registers) until something interesting pops up. In this case, a check in FileManager occurs and I `finish` and rewrite the contents of the return register. 

The following is the an abbreviated result of stepping through the function. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/088f6f70-da1c-5d61-bec1-c4dca6f6721f.tiff)

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/7711a4ec-e7ac-57b5-bcf6-4346ea077fd6.tiff)

Now, finish the functions execution and examine x0's contents.
![](https://citadelsh.s3.amazonaws.com/prod/posts/images/2f0dd6ea-2b92-5f8e-be23-df225de80df6.tiff)

The function could be returning `true` for a variety of reasons and it's not guaranteed that a value of 0x0000000000000001 means â€œjailbrokenâ€ .  This is a trial and error process, so be ready to set a lot of breakpoints and read a lot of registers. 

Next we can set the contents of x0 to 0.

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/14aca4a4-ee94-5285-9640-a1f3da2635fc.tiff)

Continue the execution by pressing `c`. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/c9f375fc-a68b-51fa-ab49-e40dfb5e6069.jpeg)

**Jailbreak Test 2**

This solution we will examine the `+[JailbreakDetection isJailbroken]` class method. Pictured below, weâ€™re interested in setting a breakpoint on the address where `ret` is called. `ret` returns from a subroutine with the arguments in x0, which is the return register. The return register will contain the value the function returns.

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/4b454a08-394b-5423-89cd-c45a9ff613df.png)

After setting a breakpoint on this address and hitting the breakpoint, the register x0 will have the value of `0x0000000000000001`. Since we want to return false, set the value of x0 to 0. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/bcbf42dd-4dda-5694-941e-fe5af9fe7ea6.tiff)

Continue the execution of the function and you will see the following on your iPhone.
![](https://citadelsh.s3.amazonaws.com/prod/posts/images/68602c71-2858-5372-8670-9652fbe7ea1b.jpeg)

**Jailbreak Test 3, 4, 5**

Searching hopper for `test3` results in: 
![](https://citadelsh.s3.amazonaws.com/prod/posts/images/1726d143-66bb-52c0-8d0f-c7356395d424.tiff)

As you can see here, we have a `jailbreakTest3Tapped` and a `jailbreakTest3`. Weâ€™re willing to bet that `jailbreakTest3Tapped` is an IBAction that calls `jailbreakTest3()`. Therefore we are most interested in seeing the contents of `jailbreakTest3()`. 

Inside there are some interesting operations. First, we AND the contents of register eight with the value `0x1`. Second, TBZ is defined as `Test bit and branch if zero to a label at a PC-relative offset`. This definition can be translated into â€œif the contents of register eight is zero go to `0x0000000100195da8`, otherwise continue". 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/78b2a3da-ce7c-5b53-a5dd-6488d5e4ecf2.tiff)

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/2b502faa-4006-5d7e-ad24-780223f043e0.tiff)

Because we know our device is jailbroken and â€œDevice is jailbrokenâ€  appears when we press â€œJailbreak Test 3â€  we can conclude that register 8 contains the value 0x1 and this is why our functions reaches `0x0000000100195a04`. 

Set a breakpoint at `0x0000000100195d8` and read the contents of register 8.

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/b7bb15c1-def7-5ad8-a1ef-210a30e4692f.tiff)

Just as we thought, the value is 0x1. Let's write the value 0x0 to register 0. 

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/7221049f-6849-5570-a520-d49bb41a9873.tiff)

Continuing the execution you should see the following.

![](https://citadelsh.s3.amazonaws.com/prod/posts/images/aa3f65de-3813-5231-94dd-69466a6ab7c8.jpeg)

Applying this same technique to Jailbreak Test 4 and JailbreakTest 5 will also work. 