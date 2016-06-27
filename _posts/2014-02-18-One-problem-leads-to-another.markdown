---
layout: post
title:  "One problem leads to another"
date:   2014-02-18 21:19:27
author: "Brock Ellis"
categories: code work
comments: true
image: "2014-02-18.jpg"
---

Isn't it amazing how one simple, little problem can lead to a whole heap of issues?

A few years ago, I was a super junior programmer at Sycamore Leaf Solutions. I was learning on the job and having a ton of fun doing it. I was at a point where I wanted to take on a real meaningful project as opposed to the smaller jobs I was doing at the time. I set my sights on creating a mobile app for our system.

I did some research and found jQuery Mobile. It looked awesome. I could use simple HTML markup and just use their CSS files and have a touch optimized website. Still being fairly green, I had no idea what I was getting myself into. I threw together a few pages written in PHP and hosted on our servers and used the jQM assets. What I had at the end was a 'mobile web app'. We had a heck of a time trying to explain to our customers that this new 'app' can't be downloaded- you could only view it on the web. It wasn't the huge hit that I wanted.

So I doubled down. They wanted a downloadable app and I was going to give it to them. I looked further into jQM and found that it was really meant to be used via jQuery on mobile apps as a single page application (SPA). I set up the SPA and found that, because it was purely client-side code, I had to provide some type of webservice to consume and provide the data.

That lead me to look into creating an API for our service. I researched how to build out an API and how to use RESTful practices. I found a framework to use and made all of the endpoints as perfect as possible. In my naiveity, I chose API key security. However, as soon as we decided to open this API up to 3rd parties, I had to figure out how to let clients authenticate against the API without end users providing their login credentials.

This train of thought led to implemnting OAuth for our API. I read up on the differences between OAuth 1.0a and 2.0. I can almost do the "OAuth Dance" in my sleep at this point. We eventually settled on OAuth 2 and worked toward building a whole developer portal and an access token system for all of our users. What would happen if we get massive traction and a lot of developers start building on our API? That'd be awesome, but it could mean a lot of extra overheard as far as database calls goes...

That conversation lead to utilizing some middleware like Memcache or Redis as a simple store for crucial information that is needed to access data behind the API. We use information like unique School Year or Quarter ID's that is needed for almost every database call, but we would rather not hit the database every time to look it up. This is what I am currently working on.

Wow. It started with such a simple goal: build a mobile app. Because I was so inexperienced, I could not understand everything that was going to be required in pursuit of that goal. If I had to do it all over again I wouldn't change a thing. By slowing getting grasping more and more complex topics, I was able to learn how they interconnect and why it is important to have them. I will, however, be a bit more careful when taking on my next 'meaningful' project.

"Eat that elephant one step at a time."
