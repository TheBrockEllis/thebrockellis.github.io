---
layout: post
title:  "Ionic PWA as Chrome Extension"
date:   2017-02-10 14:40:00
author: "Brock Ellis"
categories: ionic code
comments: true
image: "2017-02-10.jpg"
---

tl;dr Stuffing an Ionic progressive web app into a Chrome extension...because we can

---

# Back story

Recently I was trying to clear out old projects from my Github account when I ran across an old Google Chrome extension I had written almost 4 years ago. The extension was an experiment in Sycamore School's first foray into APIs. It allowed a student to login and check their grades. No muss, no fuss.

For some reason, I was compelled to update the app to work with all of the new changes that Sycamore's API had seen (true OAuth2 support, primarily). I took a look at the source code and got sick. jQuery and bootstrap. Lots of random functions and not much cohesiveness. It really made me appreciate how I've grown as a developer. After adjusting the `$.ajax` requests and getting it working, I thought about how I could improve it.

That led me to google 'angular 2 in Chrome extensions'. This got me no where. Lots of help resources *for* angular as chrome extensions but nothing about using angular *in* an extension. Then I remember reading some of Justin Willis's old tweets about how "progressive web apps" are just plain web sites on steroids. Well, Chrome extensions are just html/js files, so why couldn't I just make an Ionic PWA into a chrome extension. Turns out, you can.

# Building a PWA from an Ionic app

Building a PWA from an Ionic app is super simple. First, you need to run `ionic platform add browser` inside of your project directly. This adds the "browser" platform to your app, right alongside Android and iOS.

Then you run `ionic build browser --production`. This mimics the same build process that you'd use for other native platforms. The difference is, instead of placing the transpiled and platform optimized code in the `platforms/android` or `platforms/ios` directories, it places your app in the `www` directory.

This is what my `www` directory looked like through this process:

```
www/
  - assets/
  - build/
  - index.html
  - service-worker.js

```

(Now, I haven't touched service workers yet. They still seem like black magic to me so I have ignored them to this point.)

# PWA as Chrome extension

Once you have the PWA built, you can start to add basic Chrome extension features. [The biggest piece you need to add is a `manifest.json` file](https://developer.chrome.com/extensions/getstarted#declaration) in the `www/` directory. This manifest tells Chrome the details about your extension- similiar to a `package.json` file does for NPM packages. Click the link for more particulars.

At this point, there were a few hacks I needed to accomplish to get the Ionic PWA to load:

1. Go into `index.html` and remove the `cordova.js` include statement. Since we're not going to run this on a device, cordova will never be present. Nuke it.
2. Need to relax the security policy for the extension. I'm not going to lie, I don't know 100% what that does in terms of security, and before you try this on a production app you'd probably want to research it, but it was necessary for the app to boot.

I did this by adding the following entry to the `manifest.json` file:

```
...
"content_security_policy": "script-src 'self' 'unsafe-eval'; object-src 'self'",
...
```

3. This is going to sound odd, but I had to adjust the size of the Chrome Extension window. The app just runs in whatever container it's given (usually determined by a device's resolution) so I had to add some very basic CSS to the `index.html` to make the window size larger.

```html
<head>
  ...
  <style>
    html {
      width: 300px;
      height: 500px;
    }
  </style>
  ...
</head>
```

With those small hacks in place, I get a wonderful Ionic application running in as a pure Chrome extension.

# Disclaimer

I did this out of curiousity. Not sure it'll ever see the light of day beyond a tweet and this blog post. There are probably things I am forgetting, so please use caution and your best judgement before blinding following these steps.

<img src="/blog/img/2017-02-10-1.png" style='display:block;margin: 0 auto;width: 75%;'>

But damn it looks good though.
