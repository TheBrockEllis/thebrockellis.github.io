---
layout: post
title:  "Updating Ionic beta.11 to RC0"
date:   2016-10-10 09:11:00
author: "Brock Ellis"
categories: code Ionic Angular
comments: true
image: "2016-10-10.jpg"
---

tl;dr My notes from upgrading a large-ish Ionic 2 app from beta.11 to RC.0

Huzzah! Ionic 2 Release Candidate 0 is out in the wild!!!

<img src="/blog/img/2016-10-10-1.png" class='img-responsive img-thumbnail' style='width:50%;display:block;margin:0 auto;'>

This is super exciting news. I've been using Ionic for a few months now full time and it's an amazing framework. I love just about everything about it. However, it is still officially in beta so the constant breaking changes is difficult to deal with.

The biggest hurdle I've had with Ionic (and my extension, Angular, since Angular powers the framework) is how to do forms correctly. One of the apps I've been building uses forms fairly extensively and it's been a lot of trial and error to get things working.

I'm working through updating this app to RC0 and found some issues with the new forms modules from Angular. I want to use this blog post as a living list of notes that I can refer to later (and maybe help someone else out in the future).

---

## app.module.ts

The App Module is a new concept for Ionic and I'm still trying to grok it. I do know that to get forms to work properly, you'll need to import the FormsModule into the `imports` section of the App Module.

```typescript
import { NgModule } from '@angular/core';
import { FormsModule }   from '@angular/forms';
import { IonicApp, IonicModule, Nav } from 'ionic-angular';
import { MyApp } from './app.component';

@NgModule({
  declarations: [
    MyApp
  ],
  imports: [
    IonicModule.forRoot(MyApp),
    FormsModule
  ],
  bootstrap: [IonicApp],
  entryComponents: [
    MyApp
  ],
  providers: [Nav, TokenService, MeService, HttpService, SchoolService, IssueService, ClassService, ErrorService]
})
```

## updateValue()

I used to update a value within a FormGroup with the following code:

```typescript
(<FormControl>this.assignmentForm.controls['subject']).updateValue(assignment.SubjectID);
```

I would need to cast the control to a FormControl instead of an AbstractControl to get access to the `updateValue()` method. However, in Angular 2 Final updateValue() has been deprecated and split into two separate methods: `patchValue()` and `setValue()`. You can read more about the difference and reasonings [here](https://github.com/angular/angular/pull/10537). The code I'm using now is:

```typescript
this.assignmentForm.get('subject').patchValue(assignment.SubjectID);
```


## Moment import

My previous import statements for using Moment.JS looked like this:

```typescript
import * as Moment from 'moment';
```

The compiler kept throwing odd errors about not being the `unix` not being defined. I looked around and found a quick answer posted just a few hours earlier on the [Ionic Framework Forums](https://forum.ionicframework.com/t/error-from-rollup-when-importing-moment-js-with-rc0/64902). Changing the import to the following structure seemed to help:

```typescript
import moment from 'moment';
```

## No directive declared for [pipe]

I use 4 or 5 custom pipes for manipulating data in my app. However, after following the directions for upgrading a beta.11 app to RC.0, I was getting an error about no directive declared for one of my pipes. I know it had nothing to do with a specific pipe, because when I removed the first pipe I declared in the `app.module.ts` file, the error would just complain about the next one.

After re-reading the directions, it looks like I had pasted my Pipes into both the `declarations` AND `entryComponents`. Turns out, pipes are not really components, so they needed to be removed from the `entryComponents` array. Once removed, never saw the error again.

Remove pipes from the entryComponents of the NgModule. Keep them in the declarations.

## No Provider for NavController

This was working prior. I have a provider called `ErrorProvider` that acts as a universal error reporter. After every HTTP request, I catch errors and send them to that provider. The provider then extracts error data and shows an `Alert` component. I know, its bad practice to mix a provider with UI but it works, so sue me. Also, I have yet to find a good tutorial about how to share UI elements like this. As soon as I find something, I'll sure as hell post a blog about it.

The issue was that the `ErrorProvider` was accessing NavController directly (`constructor(public nav:NavController)` and then `this.nav.pop()`), when it can't anymore.

In order to access the active nav from inside of a provider, you have to access it via the App helper.

```typescript
  constructor(public app:App){
    this.nav = this.app.getActiveNav();
  }
```

## "Cannot read property 'parentInjector' of undefined"

This one was a doozy. I had two TabPages (`Tabs1Page` and `Tabs2Page`) that I wanted to switch back and forth between. Each TabPage is a root element that should have it's own navigation stack.

I was attempting to find the root nav element of `Tabs1Page` and set it to the new `Tabs2Page` using:

```typescript
  this.app.getRootNav().setRoot(TabsPage);
```
This is now the recommended way of resetting root nav (instead of `this.nav.setRoot()`). Nothing new there. Except it was no longer working in RC.0. It ws giving me this error, which, by the way, has 0 documentation anywhere on the web. I guess I may be the first person ever to see it.

<img src="/blog/img/2016-10-10-2.png" class='img-responsive img-thumbnail' style='width:100%;'>

The solution was super simple. I had forgotten to remove the `providers` on my `app.component.ts`. All of those dependencies should have been moved to the `app.module.ts`, which I did. I just neglected to clean up the root component. Commenting out that one line made things work like a charm now.

```typescript

@Component({
  templateUrl: 'app.template.html',
  //providers: [Nav, TokenService, MeService, HttpService, SchoolService, StudentService, IssueService, ClassService]
})
export class MyApp {
  ...

```
