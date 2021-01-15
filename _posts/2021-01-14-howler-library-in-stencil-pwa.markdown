---
layout: post
title:  "Howler Library in Stencil PWA"
date:   2021-01-14 23:31:00
author: "Brock Ellis"
categories: code
comments: true
image: "2021-01-14.jpg"
---

tl;dr Import core Howler library w/o plugins to fix prerendering error

---

## Backstory

I'm building a PWA with Stencil. I chose Stencil because I love the team at Ionic and I tend to like thier take on JSX/rendering.
It feel (to me, at least) like the best mix of Angular and React. This specific PWA is a game that involves sound, so 
I chose to use the Howlerjs for cross-browser compatibility issues. I am hosting this PWA on Firebase.

## Problem

My deployment process is usually running `stencil build --prerender` inside of the Stencil application and the `firebase deploy`.
Super simple. However, the Howler library was giving me all sorts of grief. It was producing a scary compiler bug.

Importing it into the component:

`import { Howl } from 'howler';`

Error on prerendered build:

```bash
> stencil build --prerender

[34:53.1]  @stencil/core
[34:53.4]  v2.3.0 ⛵️
[34:56.3]  build, app, prod mode, started ...
[34:56.5]  transpile started ...
[34:58.9]  transpile finished in 2.43 s
[34:58.9]  copy started ...
[34:58.9]  generate hydrate app started ...
[34:58.9]  generate lazy started ...
[35:00.5]  copy finished (1314 files) in 1.60 s
[35:11.2]  generate hydrate app finished in 12.29 s
[35:21.4]  generate lazy finished in 22.52 s
[35:21.8]  generate service worker started ...
[35:22.2]  generate service worker finished in 421 ms
[35:22.3]  build finished in 25.91 s

[35:23.0]  prerendering started ...
[35:23.9]  prerendering failed in 862 ms

[ ERROR ]  Hydrate Error
           ReferenceError: HowlerGlobal is not defined at
```

It keeps complaining about this global not being defined. I tried what felt like 100 different solutions;

- Installing from a CDN

- Playing around with rollup config

- Downgrade to older Howler version

None of them seemed to work.

## Solution

Then I dove into the hydrated app that was being compiled by Cmd + Clicking on the files in
the error output. I saw that the Howler error was being encountered in one of their plugins; specifically
the 'spatial' audio plugin. No my knowledge, I was not using that plugin. A quick search
through the Github issues for the repo led me to this fix: only import the code .

`import { Howl } from 'howler/dist/howler.core.min'`

That did it. No more error when building a prerendered application. So simple. So many wasted hours.

Hopefully this saves one lonely soul a few minutes. Use those minutes to be with your family or
take care of your own mental health.