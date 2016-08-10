---
layout: post
title:  "Easy Android Device Screenshots via CLI"
date:   2016-08-10 15:39:00
author: "Brock Ellis"
categories: Ionic bash tips
comments: true
image: "2016-08-10.jpg"
---

tl;dr Nifty bash function for capture screenshots from the command line for Android devices

Problem
---------------
When developing my Ionic 2 apps, I always had issues capturing screenshots. I was either using the Genymotion emulator which does not offer screen captures in the free version or I was using an attached device and had to physically take a screenshot and then move it to my computer.

Solution
---------------
I ran into [this](http://blog.shvetsov.com/2013/02/grab-android-screenshot-to-computer-via.html) blog post via a Stack Overflow question that explains you how you can use `adb` to take a screen shot from an attached device and save it locally. I added the command into a bash function in my `.bash_profile` that displays some feedback and also tacks on a random number to the end of the filename so I can grab 3 or 4 in quick succession without having to rename them as I take them.

```bash
function screenshot {
  NAMEZ=~/screen_${RANDOM}.png
  adb shell screencap -p | perl -pe 's/\x0D\x0A/\x0A/g' > "$NAMEZ"
  echo "Screenshot taken: "$NAMEZ""
}
```

This allows me to do two things:

1. Continue using a **real** Android device to test my Ionic apps on which is what you should be doing as much as possible. There really is NO substitution for real, device-in-hand testing.

2. Not have to fiddle with moving screenshots from device to computer or having to pay for the Indie plan of Genymotion (not that Genymotion isn't worth paying for, but I constantly have issues with lag that make it near un-usable).
