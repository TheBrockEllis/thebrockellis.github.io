---
layout: post
title:  "Trouble cURLing Myself"
date:   2014-03-04 21:59:27
author: "Brock Ellis"
categories: code work
comments: true
image: "2014-03-04.jpg"
---

tl;dr Code, debug, research, debug some more and then get help

Background
-------------------
A few days ago I was working with our soon-to-be-public-facing API trying to finish our integration with OAuth2. Moving to OAuth means that it would now be impossible to test the API via a web browser. To help out the new testing, I had downloaded a little Java PC program that allowed me to set up requests, set custom headers and change the HTTP methods used. The application worked awesome, but it was still a little clunky and not as nice as using the address bar in my browser. So I set out to do what every web dev would do: build a web interface to the API.

I had seen the idea a few other places: build a 'console' where users can set their access token and test out API calls without actually having to code anything. My biggest inspiration was the Google OAuth Playground which I used quite a bit early on.

The basic layout of the console (which I named 'the Sandbox') was a simple web form on the left hand side of the page and the results of the test calls on the right hand side. The form would submit the data and I would use cURL to send the test request and then display the results. I set up all the UI stuff and started on hooking up the backend and ran into some problems.

Problem
---------------------
I was trying to cURL the '/Me' endpoint on our API which just returns some basic information about your specific user. I had written some very similar code a few weeks ago but despite my best efforts, I could not get it to work. There was some inconsistency afoot. When using the PHP cURL library, it failed. Request didn’t even make it out the door. When I tried via a web browser it came back with HTTP 401, which was exactly what I was expecting. Two methods of access and two different results- let the debugging commence.

I started on the command line, trying to curl the endpoint. Nothing. I tried adding the verbose flag (--verbose) to the curl command line request and it came back with the following:

{% highlight shell-session linenos %}

* About to connect() to dev.sycamoreeducation.com port 80
* Trying 198.101.164.71... connected
* Connected to xxx.redacted.com (555.555.555.555) port 80
> GET /api/v1.0/Me HTTP/1.1
> User-Agent: curl/7.15.5 (x86_64-redhat-linux-gnu) libcurl/7.15.5 OpenSSL/0.9.8b zlib/1.2.3 libidn/0.6.5
> Host: xxx.redacted.com
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Wed, 26 Feb 2014 17:01:09 GMT
< Server: Apache
< Content-Length: 0
< Content-Type: text/html; charset=UTF-8
* Connection #0 to host xxx.redacted.com left intact
* Closing connection #0

{% endhighlight %}

Ok, I can now see a response (HTTP 200 OK) which is nice even though it wasn’t what I was expecting. Then I wondered if our API was doing some sort of redirect that the curl request couldn’t handle so I set the location flag (-L) on the curl request. Still nothing.

At this point I had been banging my head against this for a few hours and I decided that I was best served moving onto something else. I let it rest for a day and moved onto more productive things.

Solution
-------------------------
A few days went by and I brought it up to one of our sysadmins: 'Any idea why I wouldn't be able to execute a cURL call on the same domain it originates from?'. He sat down, typed a few characters and asked me to try it again. It worked. Son of a bitch, it worked. The problem was one of the host files for the server was pointing to the wrong virtual IP address. A little reconfiguration and bippity-boppity boo it all worked.

I know it would have been nice to glean some piece of knowledge out of this whole situation, but maybe what I can pass on is that there is definitely value in having an experienced and knowledgeable sysadmin on your team. Try to code something every way you know how, exhaust all debugging options, and then find someone who may know something you don’t and ask them.
