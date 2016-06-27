---
layout: post
title:  "The Checkbox Conundrum"
date:   2014-03-19 16:50:27
author: "Brock Ellis"
categories: code work
comments: true
image: "2014-03-19.jpg"
---

tl;dr Disabled form elements do not get submitted with the form and readonly doesn't work as you would think for checkboxes

Background
-------------------
Today I was working on a bugfix request. Nothing out of the usual. I usually knock out 3 or 4 of those a day, but this one was a bit of a doosey.

We were displaying a form with a few checkboxes. In one of the checkboxes (a very important one) we didn't want the user to be able to uncheck it if some condition was met (for very good reasons, I assure you). When we queried the database, if a value came back true, we would add a 'disabled' attribute to the checkbox. Awesome! The checkbox couldn't be changed and we were happy.

Problem
---------------------
The problem was when the user submitted the form, but database record for that value went in as false. I couldn't understand what was happening. The value came out of the database false, the use submits the form and the value goes back into the database as true. Wha?!


Solution
-------------------------
I did some [digging](http://www.w3schools.com/tags/att_input_disabled.asp) and found out that checkboxes with the 'disabled' attribute are not actually submitted with the rest of the form data. Not only is this the case for checkboxes, but for **all input elements**. When we process the data, we check to see if that value is present and if it is not, we default the value to false.  Wow. Noobie error. I should have known that but now I do, so moving on.

So I went ahead and applied the HTML atttribute “readonly” to my checkbox. Sounded like a reasonable idea, right? I just want the user to be able to see it and not change it. Made the change and tested it out. Firefox automatically changes my cursor to a red circle letting me know the checkbox isn't to be messed with...but, I can still check or uncheck it!?

Well, upon [further research](http://stackoverflow.com/questions/155291/can-html-checkboxes-be-set-to-readonly) the 'readonly' attribute doesn't work for checkboxes like you would first think. “Readonly” is meant to prevent the use from editing the value of an input. With checkboxes, you're never editing the value of the input, just the **state** of the input (checked vs non-checked). I'm sure some very smart people implemented checkboxes to work that way, but it goes against my first gut instinct.

The final fix was to just display a dummie checkbox and include a hidden input if that value was true and the normal checkbox if that value was false. The accomplishes the goal of having the user see, but NOT edit, the input when that value is true but allows them to change it when it is false. 
