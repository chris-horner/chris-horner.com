---
title: "Comparing Android build times by CPU"
date: 2020-08-15T15:00:49+10:00
summary: "Just how much of a difference does a beefy CPU make?"
draft: true
---

I know many in the community might crucify me for saying this, but I actually don't mind using a Windows machine for Android development; especially if it's my desktop. My two go-to tools are Android Studio and [Affinity Designer](https://affinity.serif.com/en-gb/), both of which run fine on Microsoft's OS.

<img src="../../img/pc.jpg" style="width: 320px; max-width: 100%; display: inline; float: right; margin-bottom: 8px; margin-left: 16px;">

_Very_ occasionally I've had the need to run a bash script, but in those instances Windows Subsystem for Linux ([WSL](https://docs.microsoft.com/en-us/windows/wsl/about)) has been perfectly capable. The only real loss for me on the desktop is Keynote.

Much of this stems from my interest in PC gaming. The allure of high refresh rate displays, large game libraries, and affordable performance means that macOS isn't an option. I always figured that I had the hardware for fun, I may as well put it to work too.

However the tool of choice for many Android developers seems to be a Macbook Pro, and there's a lot to like about them. The build quality is superb, the screen is fantastic, and they provide a Unix environment without the hassle of needing to recompile your kernel to get wifi working. (Linux burnt me a little).

But when it came working on my own Android projects, I always assumed I was better off making use of my gaming PC. I knew that a desktop grade CPU would outperform a laptop CPU, so compile times would surely be better compared to a Macbook?

But just how much of a performance difference is there? What kind of CPU do you need in a desktop to outperform a laptop? Will any mid range CPU be better? What about a monster, productivity focused CPU? I decided to do some digging.
