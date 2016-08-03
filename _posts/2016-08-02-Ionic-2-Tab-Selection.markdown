---
layout: post
title:  "Ionic 2 Multiple Tab Dynamic Selection"
date:   2016-08-02 23:40:00
author: "Brock Ellis"
categories: work ionic2 tabs
comments: true
image: "2016-08-02.jpg"
---

tl;dr Two sets of Ionic 2 tabs and two hours later...

Situation
-------------------
I am working on an Ionic 2 application that has two Tab components. Each tab component has 2 page components. Let's try and come up with a visual, shall we?

```
Tab 1
----------------------------------
|            |                   |
|  Work (A)  |    Personal (B)   |
|            |                   |
----------------------------------

Tab 2
----------------------------------
|             |                  |
|  Inbox (C)  |    Outbox (D)    |
|             |                  |
----------------------------------
```

Problem
---------------------
A user logs in and selects "Personal (B)". The app changes to `Tab 2` and allows them to peruse their "Inbox (C)" and "Outbox (D)" as their leisure. When they are done checking their email, they want to get back to `Tab 1` which is the main navigation for the app.

I had added the following to the header of "Inbox (C)" and "Outbox (D)" to allow navigation back to the previous tabs (`Tab 1`):

```html
<ion-header>
  <ion-navbar primary>
    <ion-buttons left>
       <button (click)="navigateToMainMenu()">
         <ion-icon name="arrow-back"></ion-icon>
       </button>
     </ion-buttons>
```

```typescript
    navigateToMainMenu(){
      //have to reset the parent's navigation back to the main TabsPage
      //look here for more info: https://webcake.co/exploring-nav-hierarchy-in-the-ionic-2-tabs-page/
      this.nav.rootNav.setRoot(MainTabsPage);
    }
```

As you can see in my comments, I had to access the rootNav before setting the root to the `MainTabsPage` (which is `Tab 1` from the diagrams above). The issue with this is that it would always direct the user back to the "Work (A)" component. The user got to the `Tab 2` from "Personal (B)" so taking them back to A is not a super UX.

I spent way too many hours Googling and cruising the Ionic forums for answers, when I stumbled upon this little gem in the documentation for setRoot:

<img src="/blog/img/2016-08-02-1.jpg" class="img-responsive img-thumbnail" style='width: 100%;' />

When setting the root page, you can pass along parameters!

So I quickly adjusted my code to send through an index that would be used in `Tab 1` to change the tab to "Personal (B)" like so:

```typescript
  navigateToMainMenu(){
    this.nav.rootNav.setRoot(MainTabsPage, {tabIndex: 1});
  }
```

And then inside of the MainTabsPage component, I import NavParams and set it in the constructor and fetch the `tabIndex`. If `tabIndex` is present, I use that set which tab is currently selected using property binding.

```typescript
  export class MainTabsPage {
    private tab1Root: any;
    private tab2Root: any;
    public tabIndex:Number = 0;

    constructor(public params:NavParams) {
      let tabIndex = this.params.get('tabIndex');
      if(tabIndex){
        this.tabIndex = tabIndex;
      }
```

Here is the attribute binding in the template:

```html
  <ion-tabs [selectedIndex]='tabIndex' #myTabs id="myTabs">
    <ion-tab [root]="tab1Root" tabTitle="Work" tabIcon="home"></ion-tab>
    <ion-tab [root]="tab2Root" tabTitle="Personal" tabIcon="person"></ion-tab>
  </ion-tabs>
```

Boom. Navigating from `Tab 2` takes you back to "Personal (B)". Phew.

Hacker ethos, FTW
---------------------
Tonight was supposed to be so productive. Changing what tab was selected when navigating back to `Tab 1` was supposed to be a 5 minute fix before I could get to the **real** work for tonight. Guess the coding gods had other plans.

In my head it didn't seem like that hard of a concept, but there didn't seem to be any resources on the interwebz addressing this same scenario. This feels hacky and somewhere, deep down in my bones, I feel there is a better way to do this (or a better way to architect the app so this isn't an issue) but my hope is that this blog post helps someone else out in their hour of need.
