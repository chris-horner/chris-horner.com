---
title: "Comparing Android Build Times by CPU"
date: 2020-08-15T15:00:49+10:00
summary: "Just how much of a difference does a beefy CPU make?"
draft: true
---

I know many in the community might crucify me for saying this, but I actually don't mind using a Windows machine for Android development; especially if it's my desktop. My two go-to tools are Android Studio and [Affinity Designer](https://affinity.serif.com/en-gb/), both of which run fine on Microsoft's OS.

<img src="../../img/pc.jpg" style="width: 320px; max-width: 100%; display: inline; float: right; margin-bottom: 8px; margin-left: 16px;">

_Very_ occasionally I've had the need to run a bash script, but in those instances Windows Subsystem for Linux ([WSL](https://docs.microsoft.com/en-us/windows/wsl/about)) has been surprisingly capable. The only real loss from macOS for me is Keynote.

Much of this stems from my interest in PC gaming. The allure of high refresh rate displays, large game libraries, and affordable performance means that macOS isn't an option. I always figured that if I had the hardware for fun, I may as well put it to work too.

However the tool of choice for many Android developers seems to be a Macbook Pro, and there's a lot to like about them. The build quality - excepting keyboard - is superb, the screen is fantastic, and they provide a Unix environment without the hassle of needing to recompile your kernel to get wifi working. (Linux burnt me a little).

But when it came working on my own Android projects, I assumed I was better off making use of my gaming PC. I knew that a desktop grade CPU would outperform a laptop CPU, so compile times would surely be better compared to a Macbook?

But just how much of a performance difference is there? What kind of CPU do you need in a desktop to outperform a laptop? Will any mid range CPU be better? What about a monster, productivity focused CPU? I decided to do some digging.

---

I'll note that this is my first attempt at any such benchmark. I'm aware that _many_ things can affect a benchmark of this nature, and it's entirely possible that there are flaws in my methodology. I'll try to be as clear as possible about the process I've taken. If anyone spots areas for improvement, I'd welcome the feedback. This is something I'd like to get better at.

With that out of the way, let's discuss how I went about the tests.

#### Projects and Methodology

When measuring build times, I wanted to select three open source projects that represented different levels of complexity.

- [Tivi](https://github.com/chrisbanes/tivi): Lots of modules, decent amount of code
  * commit `e9d71b84792d74c35bae38641c1620bc44a6f878`
- [Plaid](https://github.com/android/plaid): Some modules, medium amount of code
  * commit `e703957b5e5d4728dea94f11f8d0d27d227f9725`
- [Socket Weather](https://github.com/chris-horner/SocketWeather/): One module, small amount of code
  * commit `4ba5df8f1ad4cdceb0337cd2fff5af1010095079`

With each of these projects, I tried to profile both full and incremental builds.

##### Full builds

After successfully completing a build of each project, I executed the following command five times:

```sh
gradle --profile --offline --rerun-tasks assembleDebug
```

##### Incremental builds

I then modified their respective `MainActivity` files, adding a companion object.

```kotlin
companion object {
  val test = "test"
  fun test0() = test + "test0"
}
```

I incremented the number in both the function name and string five times. This was an attempt to see how incremental compilation would cope with a static function changing in both signature and content. With each increment, I executed:

```sh
gradle --profile --offline assembleDebug
```

Once that was done, I took the `Total Build Time` in the generated profile reports of each build type. I calculated the median time in each category, and it's that value that's shown in the charts below.

#### CPUs tested

Unfortunately due to the global pandemic, I wasn't able to test with as many CPUs (especially those in Macbooks) as I would like.

##### Core i7-8700K

This six core, twelve thread Intel processor from late 2017 [still holds up](https://www.youtube.com/watch?v=1MaBmwdgjuU) well in 2020 for gaming.

##### Core i5-7500

Released at the beginning of 2017, this four core, four thread 7500 represented great value for gaming compared to its i7-7700 contemporary at the time.

##### Ryzen 3950x

Sixteen core, thirty-two thread powerhouse released at the end of 2019. This CPU was a huge boon for anyone looking to build a powerful workstation for tasks that could take advantage of many cores. While _not cheap_ compared to other consumer processors, it's considerably more affordable than the likes of Intel's Xeon chips or AMD's Threadripper.

##### Core i9-8950HK

Another six core, twelve thread processor, this CPU was running in a 2018 Macbook. A machine infamous for its [launch day overheating](https://www.techrepublic.com/article/laptop-expert-warns-overheating-could-make-2018-macbook-pro-slower-than-last-years-model/) that was quickly addressed ([mostly](https://youtu.be/7gJ8mGFjeqA?t=517)) by Apple.

##### Core i7-4960HQ

The grandfather of the bunch. A four core, eight thread CPU housed in a late 2013 Macbook that's served me well over the years.

##### Ryzen 3600

_Yet another_ six core, twelve thread CPU. Great value for both gaming and productivity released in mid 2019.

#### Results

Let's start by looking at Socket Weather. A small project that these processors should eat for breakfast.

<a href="../../img/build_socketweather_full.svg">
  <picture>
    <source srcset="../../img/build_socketweather_full_dark.svg" media="(prefers-color-scheme: dark)">
    <img src="../../img/build_socketweather_full.svg">
  </picture>
</a>

Under ten seconds for a full build on most CPUs! Great for an Android app (though maybe [still a little slow](https://youtu.be/uZgbKrDEzAs?t=1247)).

What you'll notice here and in all the following benchmarks, is that the i5-7500 stands out like a sore thumb.

This was quite a surprise, as it compares closely to the older 4960HQ. Both have four cores, and both can boost to 3.8 Ghz. However unlike the mobile i7, this desktop i5 lacks hyperthreading. Is this enough to cause such a large difference in time?
