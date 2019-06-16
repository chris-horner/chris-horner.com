---
title: "Intent Filters and Custom Tabs"
date: 2019-06-15T09:12:22+10:00
summary: "Custom Tabs have some interesting properties when interacting with intent filters."
draft: false
---

Lately I've been working on a project that uses
[intent filters](https://developer.android.com/guide/components/intents-filters) to open deep links in an app. I'd never
done this before, but after reading through
[the documentation](https://developer.android.com/training/app-links/deep-linking) on how it should work it seemed
pretty straightforward.

The feature was implemented, tested, and everything seemed to work perfectly. I could tap on one of our predefined URLs
and our app would open at the correct screen. I smugly informed our testers that the feature was ready to rock and
moved on to other tasks...

Obviously, it wasn't going to be quite so simple. After giving the feature a crack, one of our testers got in touch and
informed me that deep linking wasn't working as expected. Apparently after tapping on a URL it was simply opening the
browser and never launching our app. I asked the standard set of questions any Android developer might ask:

- What device are you using? Is it a Samsung?
- What version of Android is it running?
- Did you tell the OS to always open these links with the browser?

After looking into the issue, it turns out the URL the tester was using wasn't itself one of the URLs we handled.
Rather, it was a URL that would redirect to one we did handle with the expectation that after the redirect it would
open our app.

I grabbed a copy of this URL for myself and gave it a tap. Frustratingly, it worked perfectly. Tapping the URL
first opened Chrome, which then performed the redirect, which then opened our app. Wondering what the difference could
be, I had a look at the tester's device. I tapped the link and sure enough the browser didn't call into our app after
the redirect. During this I noticed something fishy; this browser didn't look like any I'd seen before.

Digging into the device's settings, I noticed someone had downloaded and set the default browser to Microsoft Edge.
"Ah ha!", I exclaimed. I changed the default browser to Samsung Internet (assuming it might also suffer from similar
issues) and gave the link another tap. Curiously, it worked perfectly with Samsung's browser...

I had a chat with the product owner, hoping that we could avoid dealing with this issue for now. Surely the percentage
of users with Edge would be so small it wouldn't be worth dealing with. Alas, the product owner informed me that he'd
suffered the same issue, and he'd never installed Edge. Something else was afoot.

<img
    src="../../img/intent_filter_chrome.png"
    style="width: 300px; max-width: 100%; display: inline; float: right; margin-bottom: 8px;"
    />

With a hunch, I asked him what app he was using when he opened the link. "Gmail, like many users will", he told me.
This whole time I'd been opening the link via Google Keep for convenience. I knew that Gmail made use of
[Custom Tabs](https://developer.android.com/reference/androidx/browser/customtabs/package-summary.html) to open links
inside of itself. Perhaps this was causing problems? Quickly sending myself an email with the link I opened it on my
device...

...only to find it _still_ worked perfectly.

I couldn't see any major differences between my device and those experiencing issues. At least, none that would
cause a problem like this to occur. In desperation I uninstalled and reinstalled our app, hoping a fresh start would
help.

After this, I was finally able to see the redirect fail to open the app.

It seemed odd to me that reinstalling the app would suddenly make the issue rear its head. What could possibly be
persisted that would make a redirect succeed inside of a Custom Tab? I decided to reopen the link from Keep again, and
noticed it worked. Not making use of Custom Tabs, Keep would open Chrome and then pop up a dialog asking which app
I'd like to use to handle the link.

<figure style="float: right; margin-top: 0px; display: inline; max-width: 100%; ">
  <img
      src="../../img/app-chooser-dialog.png"
      width="354px"
      style="max-width: 100%;"
      />
  <figcaption>Example app chooser dialog.</figcaption>
</figure>

While testing this feature, I remember tapping the `Always` button on the app chooser dialog. (You know the one I'm
talking about, it's everyone's favourite). I was tapping quite few links and didn't want to bother seeing that
dialog every time. Were the two somehow linked?

I tapped on `Always` again and watched our app open. I closed it, navigated back to Gmail, and tapped on the cursed
redirect link...

And voila, our app opened successfully.

<strong>TL, DR</strong>: Intent filters will only work in Custom Tabs if the user has _already_ said they'd like your
app to be the default handler for a particular type of link. If they haven't yet committed to that kind of relationship
then redirects to your URL inside a Custom Tab will fail.

To circumvent the issue I ended up creating an additional intent filter for the redirect URL. That URL now opens our
app, checks if it redirects to a URL we actually want to handle, and then either consumes it or passes it on to a
browser. Not the most elegant solution, so if anyone out there knows of something better, please get in touch!
