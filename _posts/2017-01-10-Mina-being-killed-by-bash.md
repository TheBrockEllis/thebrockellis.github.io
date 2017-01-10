---
layout: post
title:  "Mina Deploy Being Killed By Bash"
date:   2017-01-10 15:37:00
author: "Brock Ellis"
categories: code rails mina
comments: true
image: "2017-01-10.jpg"
---

tl;dr Vague error that killed mina deployments mid-asset pre-compilation

---

For one of the ruby on rails side projects I am working on, we use Mina for a deployment orchestration. We develop code locally with Docker and use Mina to deploy changes to a Digital Ocean cloud server. This worked perfectly for many weeks until last night I ran into a very persistent bug.

When running `mina deploy` the process would end prematurely. Here is the output that we saw:

```
Using sass-rails 5.0.6
Using devise 4.2.0
Using dateslices 0.0.4
Bundle complete! 39 Gemfile dependencies, 87 gems now installed.
Gems in the groups development and test were not installed.
Bundled gems are installed into ./vendor/bundle.
-----> DB migrations unchanged; skipping DB migration
-----> Precompiling asset files
I, [2017-01-10T04:26:13.609282 #23582]  INFO -- : Writing /var/www/filmit/tmp/build-148402234431163/public/assets/tinymce-17efbbe86f02b5fd13671079384552216aa633ba6b99fb99eeeb422d9656db88.js
I, [2017-01-10T04:26:13.615337 #23582]  INFO -- : Writing /var/www/filmit/tmp/build-148402234431163/public/assets/tinymce-17efbbe86f02b5fd13671079384552216aa633ba6b99fb99eeeb422d9656db88.js.gz
I, [2017-01-10T04:26:13.631241 #23582]  INFO -- : Writing /var/www/filmit/tmp/build-148402234431163/public/assets/android_app_store-43878635807d52c36e10ba17e6eac080320f2edbdd94721e84a7a9268fde01ce.png
I, [2017-01-10T04:26:13.640985 #23582]  INFO -- : Writing /var/www/filmit/tmp/build-148402234431163/public/assets/ios_app_store-3b9b9555a5776afb1533e32469388658d4f8fdddb1989a035bd678e914ecc187.png
I, [2017-01-10T04:26:13.653709 #23582]  INFO -- : Writing /var/www/filmit/tmp/build-148402234431163/public/assets/logo-f43f5ea5ee65c8ce7b4f28e7d0119cbe85a56996d6cfc8a791813e2cf97e8c10.png
bash: line 223: 23582 Killed                  RAILS_ENV="production" bundle exec rake assets:precompile RAILS_GROUPS=assets
! ERROR: Deploy failed.
-----> Cleaning up build
Unlinking current
OK
Connection to 104.131.58.228 closed.

 !     Command failed.
       Failed with status 1 (4864)
```

Ok, we were building a new version, symlinking correctly, bundler was doing its thing properly. The problem happened when we were precompiling the assets. (A change to a JS file caused this issue to surface. If no JS, CSS or images change, pre-compilation doesn't run, I believe). The asset complilation was taking place because we had `invoke :'rails:assets_precompile'` listed in our `deploy.rb` file. This makes the most sense, because you realy shouldn't be storing pre-compiled assets in version control. Compile them on-the-fly when you need to, amirite?

There were two issues on the main Mina repository ([#282](https://github.com/mina-deploy/mina/issues/282) and [#370](https://github.com/mina-deploy/mina/issues/370)) but both of them were of no help. I looked toward Stack Overflow hoping for more general advice and found [this gem of an answer](http://stackoverflow.com/questions/22656935/mina-deploy-error-deploy-failed):

<img src="/blog/img/2017-01-10-1.png" class='img-responsive img-thumbnail' style='width:50%;display:block;margin:0 auto;'>

Thank you so much, Giri!

I took a look at `top` and indeed, I was pushing my little $5/month VPS's RAM to the max. I followed the [wonderful tutorial](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-16-04) by Digital Ocean on creating a swapfile for Ubuntu 16.04 and things worked perfectly after that.

> As an aside: major kudos to those technical writers at Digital Ocean. Every tutorial I've ever read has been  top notch).
