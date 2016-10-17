---
layout: post
title:  "Ionic 2 RC1 Pipe Build Error"
date:   2016-10-17 00:12:00
author: "Brock Ellis"
categories: code Ionic Angular
comments: true
image: "2016-10-17.jpg"
---

tl;dr Fun error with ngc when building RC1 app solved!

Just upgraded my RC0 app to RC1 and was adding some new code. Wanted to throw an update to a buddy of mine via TestFlight for iOS. I went to go run `ionic build ios` and ran into the following error:

```
[00:06:01]  Error: Error at /Users/brockellis/Code/cpr/mobile-guidebooks/.tmp/pages/chapter/chapter.ngfactory.ts:224:47
[00:06:01]  Supplied parameters do not match any signature of call target.
[00:06:01]  ngc failed
[00:06:01]  ionic-app-script task: "build"
[00:06:01]  Error: Error
```

A little digging shows that it has nothing to do with the file being named (`chapters` in my case).

I was using a pipe in that page template. That pipe was set up like so:

```typescript
import { Injectable, Pipe } from '@angular/core';
import { DomSanitizer, SafeHtml } from '@angular/platform-browser'

@Pipe({
  name: 'sanitize'
})
@Injectable()
export class Sanitize {
  constructor(public sanitizer:DomSanitizer) {}

  transform(value: string, args: any[]) {
    return this.sanitizer.bypassSecurityTrustHtml(value);
  }
}
```

The problem was that the pipe was accepting an `args` param but I was not actually sending or use that param. I don't fully grok what the Angular ngc is trying to do, but according to [this Ionic App Script issue](https://github.com/driftyco/ionic-app-scripts/issues/150), it's very picky.

I removed the `args: any[]` since I wasn't using them and things work perfectly.

Woot.
