---
layout: post
title:  "Using Github Pages To Host Ionic Apps Automatically "
date:   2017-05-05 15:35:00
author: "Brock Ellis"
categories: code ionic github
comments: true
image: "2017-05-05.jpg"
---

tl;dr Setting up a Cordova hook to copy build files to `/docs` for Github Pages

---

# Problem

I am building an app with Ionic (surprise) and am doing some rapid iterations with the design. In order to get folks to be able to see it early and often, I decided to go with a progress web app (or PWA for you cool kids). The problem was that I had to find someplace to host the code online for the stakeholders to see it. Manually copying or uploading it was not an option.

# Solution

Github has this amazing feature where they allow you to host static assets (JS, HTML and CSS) for free on their site using a feature called 'Pages'. Since Ionic apps are made up of 'static assets' this would be a perfect way to host the app online for free and easily. The only problem is that Github gives you 2 choices for location of the Pages:

1. The entire `master` branch will be the content of the page, or
2. The `/docs` directory at the root of the project will become the content for the page.

Well, an Ionic project has multiple platforms and a bunch of files that do not pertain to the PWA, so the entire master branch was out of the question. The `/docs` folder was the only other option. I had to figure out how to move my PWA code to the `/docs` folder after building the app so that on `git push origin master` the PWA would be updated.

# Implementation

I accomplished this my creating a custom Cordova hook that runs after the build process completes. The hook checks to make sure that we're being for the `browser` platform, and if we are, it copies all compiled assets to `/docs` automatically.

First, start by creating a `/hooks` directory at the root of the Ionic project if you don't have one. Inside of that directory, you can have many sub-directories for the different actions, i.e. `after_build`, `after_platform_add`, `before_compile`, etc. I wanted to make sure the code was copied only after the build process was done, so I chose after_build. See [here](https://cordova.apache.org/docs/en/latest/guide/appdev/hooks/) for more details on the hooks.

```shell
$ cd my_ionic_project
$ mkdir hooks
$ mkdir after_build
```

Hooks are actually just javascript files (run via node) so we can access all of the npm packages that we have installed if needed. I created a file and made sure the file was executable:

```shell
$ touch 010_copy_browser_to_docs.js
$ chmod 775 010_copy_browser_to_docs.js
```

Here is the content of that file:

```javascript
#!/usr/bin/env node

// Copy Browser Code To Docs for Github Pages
// v1.0
// Automatically copies the platform/browser/www/ contents to /docs so that
// the repository can automatically be hosted with Github pages

var shell = require('child_process').execSync;
var fs = require('fs');
var path = require('path');

const src= 'platforms/browser/www';
const dist= 'docs';

if(process.env.CORDOVA_PLATFORMS == "browser"){
  shell('mkdir -p ${dist}');
  shell('cp -r ${src}/* ${dist}');
}

```

Credit to [Abdennour TOUMI](http://stackoverflow.com/a/41030035/1814019) on stackoverflow for that technique.

You want to make sure you have the hashbang at the top that specifies node. The script itself is pretty self explanatory. Check to see if we're building for `browser` and copy some files if we are.

P.S. make sure that `/docs` is also 775 so the script can write the files to it.

The major downside to this method is that you have to check in `/docs` to git which inflates the sites of your repo a bit since all of the source code for the app is there. But for development purposes, I think it works alright.
