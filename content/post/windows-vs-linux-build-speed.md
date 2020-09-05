---
title: "Windows vs Linux Android Build Time"
date: 2020-09-05T16:02:58+10:00
summary: "How much does your OS impact your build times?"
draft: true
---

After recently comparing [build times by CPU](../cpu-build-comparison), some [pointed out](https://www.reddit.com/r/androiddev/comments/if2i04/comparing_android_build_times_by_cpu/g2lk7of) that build times might be _greatly_ affected by the operating system. I was curious about this, and decided it was time for another benchmarking adventure.

Ideally we'd compare Linux, Windows, and macOS on identical hardware. I considered using my 2018 Macbook and running Bootcamp for Windows, but unfortunately it looks like Apple's thermal throttling "fix" [only apply when running macOS](https://www.youtube.com/watch?v=7gJ8mGFjeqA&t=429), which wouldn't make for a fair fight.

Instead, I decided to directly compare just Linux and Windows using the Ryzen 3950X. I _could_ have taken this a step further and made a hackintosh, but that much effort might be an exercise for another time.

<img src="../../img/android_ssd_replacement.jpg" style="width: 100%; max-width: 100%;">

Not wanting to mess with my existing Windows installation, I removed my current SSD and replaced it with a spare Kingston PCIe Gen 2.0 x4 M.2 SSD. I first installed Linux, then Windows onto the exact same disk when performing the tests.

For projects, I decided to again use [Tivi](https://github.com/chrisbanes/tivi) and [Socket Weather](https://github.com/chris-horner/SocketWeather/) as representatives of large and small codebases respectively. Like the previous post, the apps were built at the commits:

- Tivi: `e9d71b84792d74c35bae38641c1620bc44a6f878`
- Socket Weather: `4ba5df8f1ad4cdceb0337cd2fff5af1010095079`

#### Tweaking Windows 10 Professional

One important step I've always taken is to ensure that Windows Defender isn't relentlessly scanning files during a build. A quick search reveals this is a [common problem](https://discuss.gradle.org/t/why-is-gradle-so-much-slower-on-windows-ntfs/20108), but adding [appropriate exclusion](https://docs.microsoft.com/en-us/windows/android/defender-settings) directories _should_ help alleviate the issue.

#### Which Linux Distro?

I decided to install [Pop!\_OS](https://pop.system76.com/), an Ubuntu derivative that advertises itself as:

> an operating system for STEM and creative professionals who use their computer as a tool to discover and create

I'd also heard it was easy to install with Nvidia drivers out of the box. Great for me who  wanted to spend as little time configuring the OS as possible.

#### Methodology

After my previous round of tests, I've since learned I could have automated much of my testing with [gradle-profiler](https://github.com/gradle/gradle-profiler). In the future I'll definitely be using this tool, but in order to keep the data from this round of testing comparable to the previous round, I decided to follow the same process outlined in [the previous post](../cpu-build-comparison).

#### Results

Let's start with the median time of a full Socket Weather build.

<a href="../../img/wvl_socketweather_full.svg">
  <picture>
    <source srcset="../../img/wvl_socketweather_full_dark.svg" media="(prefers-color-scheme: dark)">
    <img src="../../img/wvl_socketweather_full.svg">
  </picture>
</a>

Whoa! Linux takes roughly one third the time of Windows with a gain of 67%. What about for incremental builds?

<a href="../../img/wvl_socketweather_inc.svg">
  <picture>
    <source srcset="../../img/wvl_socketweather_inc_dark.svg" media="(prefers-color-scheme: dark)">
    <img src="../../img/wvl_socketweather_inc.svg">
  </picture>
</a>

Not quite as big a difference at 32% faster, but still significant.

---

Let's move onto the heftier Tivi, a project more representative of what developers might be working on day to day.

<a href="../../img/wvl_tivi_full.svg">
  <picture>
    <source srcset="../../img/wvl_tivi_full_dark.svg" media="(prefers-color-scheme: dark)">
    <img src="../../img/wvl_tivi_full.svg">
  </picture>
</a>

Here we see Linux building 16% faster than Windows. Not quite as impressive as Socket Weather, but a seven second gain isn't bad.

<a href="../../img/wvl_tivi_inc.svg">
  <picture>
    <source srcset="../../img/wvl_tivi_inc_dark.svg" media="(prefers-color-scheme: dark)">
    <img src="../../img/wvl_tivi_inc.svg">
  </picture>
</a>

For an incremental build, Linux was just shy of being two seconds faster; an increase of roughly 21%.

#### Conclusions

It certainly looks like the crown goes to the penguin on this one. This was a genuine surprise to me. I'd always figured after taming Windows Defender, Microsoft's OS wouldn't impose significant penalties compared to other operating systems.

What's the cause of the difference? I find this a difficult question to answer.

Maybe it's related to the filesystem? Windows uses NTFS, while the distribution used for these benchmarks uses EXT4.

Perhaps there's configuration in Windows Defender that I missed? If anyone else has done similar tests, please get in touch!

Does this make Linux a better choice? Maybe. It depends on your workflow. It certainly seems to offer faster build times on the same hardware. It also integrates nicely with many existing development tools thanks to being another \*nix environment.

However there's also software it can't run. (The lack of Affinity Designer is a deal breaker for me). There's also hardware compatibility to consider. It's neat that you _can_ recompile your kernel to add support for things, but not something I necessarily _want_ to do 😅.

---

Thanks for following along with this little series. I might do more in the future that compares build times by RAM and disk speed, but I'll wrap this up for now.
