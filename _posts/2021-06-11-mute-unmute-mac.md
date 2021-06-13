---
author: robertwang
layout: post
title:  "Quickly mute and unmute mic on Mac"
date:   2021-06-11


---

 
During #pandemic you’re #WFH , are you sick of constantly mute and unmute your microphone in the meeting? Keep looking for mute button in app after app? #MicrosoftTeams ? Or #GoogleMeet ? #CiscoWebEx ?

Don’t bother anymore. Here’s a quick setup then you’re ONE hotkey away from mute or unmute your microphone **across entire Mac**.

You only need system’s Automator app.

1. Create a Quick Action workflow
2. Set “Workflow receives current” option to “no input”
3. Add “Run Shell Script” action
4-1. Paste: `osascript -e "set volume input volume 0`"
4-2. (Paste: `osascript -e "set volume input volume 100`") to another workflow file called “Unmute systematically”
5. Save this Quick Action as “Mute systematically”
6. Set a global shortcut you want in  > System Preferences > Keyboard > Shortcuts > Services

Done. 

Enjoy your peace of mind and remember to unmute when someone starting to talk!

I’m enjoying my peace of mind. Hope it works for you too! 

Wondering if there’s a hotkey for iPad?