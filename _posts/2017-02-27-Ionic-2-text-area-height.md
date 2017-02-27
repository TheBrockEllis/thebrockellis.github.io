---
layout: post
title:  "Ionic 2 Text Area Height"
date:   2017-02-22 14:23:00
author: "Brock Ellis"
categories: ionic code
comments: true
image: "2017-02-27.jpg"
---

tl;dr Making an ion-textarea grow to match containers height

---

# Back story

I have a popover that appears over a page where users can enter custom notes that will get saved in `storage`. The popover is really just a page with a header, footer and a content area with an `ion-textarea`. The problem is that the `ion-textarea` was creating a native `textarea` input after initialization and that textarea wasn't filling up all of the available vertical space.

# No rows for me

I know that usually you set the height of a `textarea` by settings the `cols` and `rows` attributes, but I just wanted the `textarea` to be as large as can be in the given space. After playing around and seeing how Ionic took the `ion-textarea` and masked the native `textarea`, I knew it was time to get into the weeds: a custom directive.

This directive is super simple. It grabs the `nativeElement` after the view has been initialized, finds the `textarea` that it created and sets the height style to 100%.

The directive:

```typescript
import {ElementRef, Directive} from '@angular/core';

@Directive({
  selector: '[elastic]'
})
export class Elastic {

  constructor(public element:ElementRef){
    this.element = element;
  }

  ngAfterViewInit(){
    this.element.nativeElement.querySelector("textarea").style.height = "100%";
  }

}
```

And it is used like so:

```html
...
</ion-header>

<ion-content>

  <ion-textarea elastic [(ngModel)]="noteContent"></ion-textarea>

</ion-content>

<ion-footer>
...
```

The hardest part to figure out was that I had to use the `ngAfterViewInit` lifecycle event. My other custom directives have all used the `ngOnInit` so it was a good time to learn the rest of the lifecycle events that Angular offers.

Also, remember to add your custom directives to your module import statements! I initially forgot that and for 5 minutes I was very confused as to why it wasn't working... :D
