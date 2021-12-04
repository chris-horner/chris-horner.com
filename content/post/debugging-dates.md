---
title: "The Case of the Disappearing Days"
date: 2021-12-05T08:49:39+11:00
summary: "We lost the day of the week. Where'd it go?"
draft: false
---

<img src="../../img/cash-chat-date.png" style="width: 320px; max-width: 100%; display: inline; float: right; margin-bottom: 8px; margin-left: 16px;">

The team I work with at Cash came across an interesting bug. We display dates _everywhere_ in Cash's UI; especially in support chat. Above groups of messages we display the date and time those messages arrived. 

If the date is within the current week, instead of the date we display the name of the day. So if `28 July` were in the current week, we would instead display it as `Wednesday`.

Formatting dates like this is simple thanks to the `java.time` API. [`DayOfWeek`](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html) is provided as an enum, complete with a [`getDisplayName()`](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html#getDisplayName-java.time.format.TextStyle-java.util.Locale-) method for easy formatting. A [`TextStyle`](https://docs.oracle.com/javase/8/docs/api/java/time/format/TextStyle.html) can be passed into this method to configure how this formatting will work. These styles are:
- `FULL`
- `FULL_STANDALONE`
- `NARROW`
- `NARROW_STANDALONE`
- `SHORT`
- `SHORT_STANDALONE`

We have quite a bit of space available in the UI, so we opted to use `FULL_STANDALONE`. After all, the documentation describes this as:

> Full text for stand-alone use, typically the full description. For example, day-of-week Monday might output "Monday".

Perfect.

Except it wasn't. One day we looked at a production build and noticed that instead of the text `Wednesday`, the number `3` was shown in its place. Wednesday _is_ the third day of the week according to the ISO-8601 standard, but `3 13:48` doesn't make for a very descriptive header.

Our first thought was that this must be related to [minification](https://developer.android.com/studio/build/shrink-code). Debug builds we create locally aren't minified with R8 and the day displayed fine. Was that the difference?

We modified our `build.gradle`
```groovy
debug {
  minifyEnabled true
}
```
and ran a build. However after an agonisingly long compile the name of the day continued to display correctly.

Was it something specific to _release_ builds? We pulled down a _debug_ build from our CI server just to verify that those would also display the date correctly.

They didn't.

We were perplexed. Why would our local builds show the name of a day correctly, but CI builds show a number instead? Was there something about our CI's configuration? Its version of the JDK? Why was this _one_ part of the `java.time` API behaving differently?

The `java.time` API is only available on Android SDK 26 and above. Cash's minimum SDK is 24, so we rely on [API desugaring](https://developer.android.com/studio/write/java8-support-table) to include those classes for older devices. Maybe there was a strange interaction between the version of the desugaring library we were using and our CI?

We bumped the desugaring library to the latest release, but continued to see the same issue.

Then it hit us. This whole time we'd been testing on emulators with relatively modern Android releases. We remembered that in certain situations, Android Studio will change the steps it takes in a build depending on the target device. (Things like avoiding multidexing if it's not necessary). What would happen if we built locally for API 24?

Sure enough, `Wednesday` became `3`.

This was corroborated by something a Google engineer mentioned on an [unrelated issue](https://issuetracker.google.com/issues/205866514).

> When one is debugging on a specific emulator, AGP injects as minSDK the emulator SDK.

So when building for our emulators from Android Studio, we were skipping the desugaring process, not including the `java.time` classes, and using those defined in the emulator's system. Executing builds from command line ensured the desugaring step always ran and used our bundled classes instead.

Do modern versions of Android include a different version of the `java.time` API?

We came across a [Jira issue](https://bugs.openjdk.java.net/browse/JDK-8146356) on the JDK itself that looked exactly like the bug we were experiencing. It was marked as resolved on the 11th of March, 2020. Digging further we found [we weren't alone](https://issuetracker.google.com/issues/176502609).

> The desugared library uses the implementation present at https://github.com/google/desugar_jdk_libs, which is derived from OpenJDK-8. There is on-going work to create a new version derived from OpenJDK-11, in that new version, the bug should effectively be fixed.

So how do we fix it for now? Simple. Use `FULL` instead of `FULL_STANDALONE` for the `TextStyle`.

### What did we learn?

- Builds from Android Studio won't be the same as command line builds when targeting particular device versions
- The `java.time` API included in the desugaring library has bugs that are fixed in later Android releases
- It's not always minification that causes problems (though it often is)

Big shout out to Kathy Ma and Jessica Woods for being great detectives. This was a tricky one!
