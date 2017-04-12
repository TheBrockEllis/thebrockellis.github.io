---
layout: post
title:  "Injecting mock/real providers in Ionic 3 NgModule"
date:   2017-04-12 11:14:00
author: "Brock Ellis"
categories: code ionic
comments: true
image: "2017-04-12.jpg"
---

tl;dr Making sure you can include mocked providers in a non-cordova environment (i.e. ionic serve)

---

# Back story

The great [Josh Morony](https://twitter.com/joshuamorony) had an amazing blog post not too long ago explaining how you could split up your providers into real or development versions. When using your app in a browser via `ionic serve` the mocked versions would be used, but when bundling your app to be used on a device where `cordova` is present, the real providers would be used.

You can read his blog post [here](https://www.joshmorony.com/automating-mocks-in-ionic-native-3-x/).

# Problem

I was having some issues getting it to work, as were many avid readers of his blog. The error read something like this:

```bash
build prod failed: Error encountered resolving symbol values statically. Calling function 'AppProviders', function calls are not supported. Consider replacing the function or lambda with a reference to an exported function...
```

Essentially, it didn't like me trying to use a function call to declare my providers array. Not to be one to back down from a challenge, I dug into Angular documentation, Github issues, and 4th-page-on-google blog posts. I think I figured it out.

# Solution

In order to dynamically inject the correct provider, I had to create a custom provider that would determine if I was developing locally or natively and return the correct provider and class to use.

## app.module.ts
```javascript
/* Potentially Mocked Providers */
import { SocialSharingProvider, NetworkProvider } from './app.providers';

/* Custom Providers */
import { Auth } from '../providers/auth';
import { User } from '../providers/user';
import { HTTP } from '../providers/http';

...

providers: [
  Auth, User, HTTP, CacheService, SplashScreen, StatusBar, Toast,
  {provide: ErrorHandler, useClass: IonicErrorHandler},
  SocialSharingProvider,
  NetworkProvider
]

```

Please note that the `SocialSharingProvider` and `NetworkProvider` are different from `SocialSharing` and `Network` like you would usually see.

## app.providers.ts
```javascript
/*
* We have this file because we want to include the mock providers for development
* but want to include the real providers in a cordova.js environment
*/

import { Network } from '@ionic-native/network';
class NetworkMock extends Network {
  get type(): string {
    return navigator.onLine ? 'wifi' : 'unknown';
  }
}

import { SocialSharing } from '@ionic-native/social-sharing';
class SocialSharingMock extends SocialSharing {
  canShareViaEmail(){
    return Promise.reject('This is a mocked provider');
  }

  shareViaEmail(){
    return Promise.reject('This is a mocked provider');
  }
}

export const SocialSharingProvider = [
  { provide: SocialSharing, useClass: (document.URL.includes('https://') || document.URL.includes('http://')) ? SocialSharingMock : SocialSharing },
];

export const NetworkProvider = [
  { provide: Network, useClass: (document.URL.includes('https://') || document.URL.includes('http://')) ? NetworkMock : Network }
]
```

You can see if this file the first two 'groups' of code are defining both the "real" and "mock" classes for a provider. I `import` the real provider and create a new class for the mocked provider that extends the real class.

The magic happens at the bottom.

I export a `const` array that `provide`s the correct Provider but has a ternary operator to specify the correct class. If my URL includes http(s), I include the mocked provider. If not, I use the standard one that was imported above.

# Disclaimer

Once again, I don't 100% fully understand what's going on underneath the hood (who does with this much abstraction going on???) but it seems to work fine. I am able to `ionic serve` my project and my promises are rejected, but I'm also able to `ionic build ios -prod` and the app sends emails via the `SocialSharing` plugin perfectly on the device.

Hit me up if I missed anything or it doesn't work. =)
