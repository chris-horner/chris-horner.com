---
title: "Persona IM"
date: 2024-09-13T17:20:00+09:00
summary: "Recreating the in-game messaging app of Persona 5."
icon: "persona-launcher.png"
screens: []
draft: false
---

Built as part of a [presentation given at DroidKaigi 2024](../../presentation/persona-im), this app aims to recreate the in-game messaging UI seen in Persona 5 as closely as possible.

Here's a reference of what it looks like in game:

<video
    controls
    loop
    style="max-width: 100%">
    <source
     src="../../video/persona_reference.mp4"
     type="video/mp4"
     />
</video>

And here's a demo of what I ended up with:

<video
    controls
    loop
    width="360px"
    style="float: right; margin-left: 24px; max-width: 100%; display: inline;">
    <source
     src="../../video/persona_demo.mp4"
     type="video/mp4"
     />
</video>

Persona 5's UI has been lauded for its innovation - from [Game Maker's Toolkit](https://youtu.be/4Bv45aPMGyI?t=531) to [Masahiro Sakurai](https://www.youtube.com/watch?v=UjW_TTNtXEM). It's had [several pieces](https://www.siliconera.com/what-makes-the-ui-and-menus-in-persona-games-so-good/) [written on it](https://ridwankhan.com/the-ui-and-ux-of-persona-5-183180eb7cce), and was made by a group of folks who knew how to break the rules in all the right ways.

Because of this, I was inspired to see if I could recreate part of this interface for Android using Compose UI.

All source code for the app is [available on GitHub](https://github.com/chris-horner/Persona-IM).

©ATLUS ©SEGA