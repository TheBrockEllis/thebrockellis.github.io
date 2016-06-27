---
layout: post
title:  "Doctype ruins $.height()"
date:   2014-04-26 20:16:27
author: "Brock Ellis"
categories: code work
comments: true
image: "2014-04-26.jpg"
---

tl;dr Missing DOCTYPE makes jQuery unable to calculate the window's height properly

Background
-------------------
We have a few different products that we support at [Sycamore Education](http://www.sycamoreeducation.com) (formerly known as Sycamore Leaf Solutions). We try as hard as we can to keep the code as similar as possible but they server completely different markets so they've been drifting further and further apart. The other day, I was trying to sync up the code on both products for the login screen. Both served the same purpose and had the same layout, and thus the code should be the same.

Problem
---------------------
After doing the initial work, I realize that one the products had a perfect centered (vertical and horizontal) div that contained all of the content and the other products div was only centered horizontally. I spent a few seconds looking back and forth between the code to see if there were any styles I had missed copying or if there was some uncommented code that was causing the hiccup. We were using jQuery to find the available height of the viewport upon page load and then dynamically add top and bottom margins (a little hack-y, but it worked). For the life of me though, I couldn't figure out what the differences between the code was.

Solution
-------------------------
Like most of the problems I encounter that can't be fixed in 10 minutes, I went straight to Google. I tried "jQuery .height() not working" and the first answer that came up was from none other than.... *drum roll* stackoverflow!

Side tangent: Stackoverflow is simply amazing. I have probably learned more from looking over answers and comments there than I have from any other location.

The first answer [to this question](http://www.stackoverflow.com/questions/12787380/jquery-window-height-function-does-not-return-actual-window-heightt) hit the nail on the head.

There was probably one line of code that was not present in both files, and it was probably the most important one: the Doctype. That line was missing from the culprit file and was causing the browser to render the page in Quirks mode which causes all kinds of havoc on a page. I'm actually surprised that the height issue was the only problem that manifest itself.

Needless to say, adding the doctype declaration to the top of the page fixed the problem.
