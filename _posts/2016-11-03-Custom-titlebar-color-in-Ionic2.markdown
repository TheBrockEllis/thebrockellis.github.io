---
layout: post
title:  "Custom titlebar color in Ionic 2"
date:   2016-11-03 15:56:00
author: "Brock Ellis"
categories: code Ionic Angular
comments: true
image: "2016-11-03.jpg"
---

tl;dr Going beyond SCSS variables and getting user-defined colors in an Ionic 2 app

---

Ionic 2 is an amazing framework. Coupled with the immense power of Angular 2 (and there is no doubt that Angular 2 is powerful) you can do some pretty darn awesome things. This is an example of one of those awesome things.

In my app, I give the users the ability to assign a primary and secondary color for their organization. When users from this organization log into the app, the Rails-based API sends back their authentication information along with all of the organization's information. This data is stored locally via `Storage`.

My goal was to try and get the header's on the app to reflect the user's organization's primary/secondary colors. I knew that I could add custom colors using SCSS variables defined in `src/theme/variables.scss`. But the problem was I didn't want to update this one file (and resubmit the entire application) each time a new customer signed up with a unique color combo.

It was then that I looked into using custom directives for Angular 2. I knew that Angular had the ability to add these things called "attribute directives" that change the appearance or behavior of an element. I just wanted to adjust the background color of an element to a different color dynamically. Easy enough, right?

I created a `directives` directory in the `/src` folder and added a `custom-background-color.ts` file. This is the content of the file:


```typescript
import { Directive, ElementRef } from '@angular/core';

import { User } from '../providers/user'; // my user provider that will allow us to access the data stored for the user

@Directive({
  selector: '[custom-background-color]'
})
export class CustomBackgroundColor {

  constructor(public element: ElementRef, public user:User){
    this.element = element;
  }

  ngOnInit(){
    this.user.getUser().then( (userData:any) => {
      this.change(userData.primary_color); //primary_color is stored locally and is what we want to change the background to
    });
  }

  change(color){
    this.element.nativeElement.children[0].style.backgroundColor = color;
  }
}
```

I had to get a little elaborate with the selector in the `change()` method because Ionic does so much manipulation with their own custom directives.

Then, I imported the custom directive to the `declarations` bock in `src/app/app.module.ts` like so:

```typescript
/* Directives */
import { CustomBackgroundColor } from '../directives/custom-background-color';

@NgModule({
  declarations: [
    MyApp,
    HomePage,
    LoginPage,
    RegisterPage,
    Sanitize,
    CustomBackgroundColor
  ],
  ...
```

Then it can be used in any HTML file.

```HTML
<ion-header>
  <ion-navbar custom-background-color>
    <ion-title custom-font-color>Page Title</ion-title>
  </ion-navbar>
</ion-header>

<ion-content class="home">
  ...
```

Overall, this feels very hacky to me. It has a high level of ["code smell"](https://blog.codinghorror.com/code-smells/) but it has seemed to work pretty well for me so far. If anyone has a better way of accomplishing this, I'd be open to learning!

The end result is quite nice though. ;-)

<img src="/blog/img/2016-11-03-1.png" class='img-responsive img-thumbnail' style='width:50%;display:block;margin:0 auto;'>
