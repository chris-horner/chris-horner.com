---
title: "Defining Transitions in Kotlin"
date: 2019-06-01T10:20:05+10:00
summary: "Most examples show customisation of Android's Transitions with XML. Building them in Kotlin can be just as good."
draft: false
---

Android's [Transition API](https://developer.android.com/training/transitions/index.html) is powerful, but it doesn't
appear to be widely used
[or loved](https://www.reddit.com/r/androiddev/comments/73x84q/im_done_with_the_android_transitions_api/) by many
developers. It sometimes requires esoteric knowledge (`android:transitionGroup="false"` I'm looking at you), and
can be quite challenging to debug. After all, when something goes wrong and your views are just popping into place
rather than animating nicely how're you meant to diagnose the issue?

It's a shame the API isn't as easy to use as something like
[MotionLayout](https://developer.android.com/reference/android/support/constraint/motion/MotionLayout), however its your
only real option when you want to animate (and share) views when navigating from one screen to another.

<video
    controls
    loop
    width="354px"
    style="float: right; margin-left: 24px; max-width: 100%; display: inline;"
    />

 <source
     src="../../video/plaid_demo.mp4"
     type="video/mp4"
     />

</video>

If you battle through the learning curve and get your head around the API you'll be rewarded with the ability to make
some pretty slick scene transitions. I really dig some of the examples shown off in the
[material design documentation](https://material.io/design/motion/customization.html).

To get up to speed on how to use transitions I highly recommend
[this video](https://www.youtube.com/watch?v=4L4fLrWDvAU) from Google I/O 2016. It's still totally relevant and the
presenters do a great job of diving into how to achieve particular effects.

If you want something that steps you through the API and demonstrates some of the issues you might encounter along the
way then you can check out [my presentation](https://www.youtube.com/watch?v=9Y5cbC5YrOY) from Droidcon SF 2017.

One of the best sample projects Google have provided that showcases material transitions is
[Plaid](https://github.com/android/plaid). It's a fairly feature packed app that consumes and presents data from
[Dribbble](https://dribbble.com/), [Designer News](https://www.designernews.co/), and
[Product Hunt](https://www.producthunt.com/). It demonstrates some complex animation effects, so it's well worth digging
through to see what kind of tricks the developers used in order to pull things off.

In particular, I really appreciate being able to make use of the
[`ReflowText`](https://github.com/android/plaid/blob/bed7b2a3202df99e0b07aece4d5f85d111d15bd8/core/src/main/java/io/plaidapp/core/ui/transitions/ReflowText.java)
transition they created. Check it out if you ever need to have text grow or shrink as it transitions.

In terms of defining the coordination of its custom transitions, Plaid places most of them in XML resources. For
example, the transition that defines how a Dribbble shot's shared element views will enter looks something like
this:

```xml
<transitionSet
    android:duration="350"
    android:transitionOrdering="together"
    >

  <changeBounds android:interpolator="@android:interpolator/fast_out_slow_in">
    <targets>
      <target android:targetId="@id/background" />
    </targets>
  </changeBounds>

  <transition
      class="io.plaidapp.core.ui.transitions.ShotSharedEnter"
      android:interpolator="@android:interpolator/fast_out_slow_in"
      >
    <targets>
      <target android:targetId="@id/shot" />
    </targets>
  </transition>

  <transition
      class="io.plaidapp.core.ui.transitions.DarkenImage"
      app:initialRgbScale="0.85"
      app:finalRgbScale="1.0"
      android:interpolator="@android:interpolator/linear"
      >
    <targets>
      <target android:targetId="@id/shot" />
    </targets>
  </transition>

</transitionSet>
```

This isn't too bad. It's declarative, and it provides a sense of hierarchy as to how the sub-transitions belong to a
parent set. However this isn't your only option when it comes to defining a transition set. If we translate this example
into straight Kotlin it'd be something like:

```kotlin
val transition = TransitionSet()
    .setDuration(350)
    .setOrdering(ORDERING_TOGETHER)
    .addTransition(ChangeBounds()
                       .setInterpolator(FastOutSlowInInterpolator())
                       .addTarget(R.id.background))
    .addTransition(ShotSharedEnter()
                       .setInterpolator(FastOutSlowInInterpolator())
                       .addTarget(R.id.shot))
    .addTransition(DarkenImage(0.85f, 1.0f)
                       .setInterpolator(LinearInterpolator())
                       .addTarget(R.id.shot))
```

This is much shorter, and I feel it's only slightly worse at conveying how the transitions relate to each other. However
you can see if we were to make this transition more complicated our indentation might start to get out of hand. It's at
this point I wondered how difficult it'd be to make some sort of Kotlin DSL for transitions.

After a quick round of experimentation, it turned out to be surprisingly simple. `TransitionSet` itself is pretty close
to what we need. All it takes is a couple of extension functions:

```kotlin
inline fun transitionSet(block: TransitionSet.() -> Unit): TransitionSet {
  return TransitionSet().apply(block)
}

inline fun TransitionSet.addSet(block: TransitionSet.() -> Unit) {
  addTransition(TransitionSet().apply(block))
}

inline fun TransitionSet.addTransition(transition: Transition, block: Transition.() -> Unit) {
  transition.apply(block)
  addTransition(transition)
}
```

Now we can build the same transition as above like this:

```kotlin
val transition = transitionSet {
  duration = 350
  ordering = ORDERING_TOGETHER
 
  addTransition(ChangeBounds()) {
    interpolator = FastOutSlowInInterpolator()
    addTarget(R.id.background)
  }
  
  addTransition(ShotSharedEnter()) {
    interpolator = FastOutSlowInInterpolator()
    addTarget(R.id.shot)
  }

  addTransition(DarkenImage(0.85f, 1.0f)) {
    interpolator = LinearInterpolator()
    addTarget(R.id.shot)
  }
}
```

Now we have something that's clear, concise, and I feel easier to work with than an XML file. It's always interesting
what you can achieve with extension functions.
