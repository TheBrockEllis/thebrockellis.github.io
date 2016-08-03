---
layout: post
title:  "Ionic 2 Promise.all() with potentially failing HTTP requests"
date:   2016-08-03 14:12:00
author: "Brock Ellis"
categories: work ionic2 http
comments: true
image: "2016-08-03.jpg"
---

tl;dr Multiple HTTP requests that may or may not fail with Promise.all

Situation
-------------------
I am building a type of messaging interface in an Ionic 2 application. The user has a `textarea` for sending a message and then they get to select their recipients to send the message to.

Problem
-------------------
The way the app is structured, messages can be sent to 3 different user types: student, parents or staff. When selecting recipients for the message, I wanted to include all 3 user types in one unified list. The required making 1, 2 or even 3 rest API calls to get the list of users (each user type had their own endpoint).

I created my HTTP requests like so:

```ts
  //employee search
  let url = `https://www.service.com/api/v1/User/${me.UserID}/Thing/Search?filter=${val}&type=3`;
  this.promises.push( this.http.get(url).map(res => res.json()).toPromise() );

  //family search
  url = `https://www.service.com/api/v1/User/${me.UserID}/Thing/Search?filter=${val}&type=2`;
  this.promises.push( this.http.get(url).map(res => res.json()).toPromise() );
```

Then I set up a Promise.all to wait for all promises to resolve before compiling a list of users to display in the template, like so:

```ts
  Promise.all(this.promises).then( values => {
    var users = [];
    users = Array.prototype.concat.apply([], values);
  });
```

My issue was that I was requiring the user to give me a string to use as a filter for the search (the `?filter=${val}` in the HTTP request). This string is used by the API to narrow search results based on last name matching.

If a user typed in "ell" it would return an employee named "Brock Ellis" but there was no families that had the last name that contained "ell". The second HTTP request would then return a 204 No Content and the promise would be rejected, causing the Promise.all() to fail and no data would be displayed at all.

Solution
----------------
The solution was to manually listen to the `.then()` and `.catch()` of each HTTP request. On a successful request, we'll have data returned and we can just `Promise.resolve(results)` and be happy. If no users were matched, we'll still return a resolved promise, but we'll just return an empty array. This way, the Promise.all() still works as intended and I attempt to merge two or more arrays (if they happen to be empty, so be it).

The above HTTP requests end up looking like this:

```ts
  //employee search
  let url = `https://www.service.com/api/v1/User/${me.UserID}/Thing/Search?filter=${val}&type=3`;
  let employeeSearchPromise = this.http.get(url).map(res => res.json()).toPromise()
    .then( result => Promise.resolve(result))
    .catch( error => Promise.resolve( [] ));
  this.promises.push( employeeSearchPromise );

  //family search
  url = `https://www.service.com/api/v1/User/${me.UserID}/Thing/Search?filter=${val}&type=2`;
  let familySearchPromise = this.http.get(url).map(res => res.json()).toPromise()
    .then( result => Promise.resolve(result))
    .catch( error => Promise.resolve( [] ));
  this.promises.push( familySearchPromise );

```

Credits for this fix go to [lxrec from programmers.stackexchange.com](http://programmers.stackexchange.com/questions/315329/error-handlers-inside-promise-all) with an amazing write up that gives more detail.
