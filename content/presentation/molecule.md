---
title: "Molecule: Using Compose for presentation logic"
date: 2022-10-06T15:40:00+09:00
summary: "A collection of throughts around building user interfaces for Android."
conference: "DroidKaigi 2022"
conferenceLink: "https://droidkaigi.jp/2022/en/"
location: "Tokyo"
youtube: "https://www.youtube.com/embed/q9p4ewk-9E4"
speakerdeck: "https://speakerdeck.com/player/5606f5b012cd43d7992ddd2903009586"
draft: false
---

Jetpack Compose can be used for more than just emitting a user interface. Molecule is a library that allows `Flow` or `StateFlow` streams to be built using Compose.

This talk covers:

- Comparing Compose to Rx and Flow APIs for presentation logic
- The benefits Compose can have on readability
- How to write presentation logic with Compose
- How Cash App uses Molecule to build and test presenters
