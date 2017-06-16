---
layout: post
title:  "Better Dynamic Colors with Ionic"
date:   2017-06-16 10:15:00
author: "Brock Ellis"
categories: code ionic
comments: true
image: "2017-06-16.jpg"
---

tl;dr Revising the concept of changing app colors at runtime in an Ionic app

---

# Backstory

A while back, [I wrote a blog](http://thebrockellis.com/2016/11/03/Custom-titlebar-color-in-Ionic2) post about how to customize the colors of Ionic components using a custom directive. The method used in that blog posts works but it's highly fragile since it's completely dependent on the HTML elements of the components and not the CSS classes. With the help of [Simon's blog post at Devdactic](https://devdactic.com/dynamic-theming-ionic/) and [David Walsh's blog post about dynamic CSS](https://davidwalsh.name/add-rules-stylesheets) I was able to achieve the same results with much less fuss.

# Problem

In our app, the users can choose what color they want to use for the menu. We use that primary color in a bunch of different locations. It helps give the user a level of ownership if they can choose how things look. In the mobile app for the product, built with Ionic, I wanted to recreate that feature but wasn't sure how to change CSS colors at runtime by injecting new rules into our stylesheets based on the results of a network request to our servers.

# Solution

Start with a blank Ionic app: `ionic start DynamicColorsApp blank`.

Open up the main app component at `cd DynamicColorApp/src/app/`.

Once there, let's add a property to the class to hold the custom background color we're going to give the app:

```
@Component({
  templateUrl: 'app.html'
})
export class MyApp {
  rootPage:any = HomePage;
  public customBgColor:any;
```

Now at this point, we should be ready to make an AJAX request back to the server to fetch the users actual background color, but that's outside the scope of this post. For now, let's just assume we have the color string. Let's store that in a property we just created and then call a function to add the necessary CSS rules to our stylesheet

```
constructor() {

  // Some async request that will fetch data that will include the custom color goes here
  this.customBgColor = "#F0F";

  this.createCSSRules();
```

Our `createCSSRules()` method looks like this:

```javascript
createCSSRules(){
  this.addCSSRule(document.styleSheets[0], ".header .toolbar-background,  .button", `background-color: ${this.customBgColor}  !important;`, 1);
}
```

This method does 2 things:
1. targets the `.header.toolbar-background` and `.button` elements and sets their background-color property to our custom color.
2. Sends all of that information to another method, `addCSSRule()` to inject the new rule into the stylesheet. Which looks like this:

```javascript
addCSSRule(sheet, selector, rules, index) {
  if("insertRule" in sheet) {
    sheet.insertRule(selector + "{" + rules + "}", index);
  }else if("addRule" in sheet) {
    sheet.addRule(selector, rules, index);
  }
}
```

Now I shamefully stole this code from David Walsh's blog. Check out the link at the top of the post for more information.

What this does is create a new CSS rule, targeting our selector (in our case, the toolbar and button backgrounds) and using the color we've loaded from a remote server. It then inserts that rule into the stylesheet of our app. The `addCssRule()` checks to see if the `insertRule` method is available, and if not, uses `addRule` instead, so we get some cross-browser compatibility to boot!

End result should look something like this:

![dynamic colors](/blog/img/2017-06-16-1.png)

I also did some additional work on making sure the font color would be visible on any color of background. I used the npm package contract (`npm install contract --save`) and used it (`import * as contrast from 'contrast';
`) to test whether the `this.customBgColor` was light or dark and then change the font color to suit. You can see my full [test file here.](https://gist.github.com/TheBrockEllis/ebfa3741c07d6d586f02d4a44af608e8)

This all works in the browser, I have not deployed this code to devices yet so your milage may vary.
