---
layout: post
title:  "Password-less SSH Login Permission Issue"
date:   2014-07-02 12:16:00
author: "Brock Ellis"
categories: code work
comments: true
image: "2014-07-02.jpg"
---

tl;dr Settings up password-less login with SSH keys in faster and more secure. Here's my journey.

Background
-------------------
I had been using Shuttle (an SSH manager for Mac) but wanted to get rid of the application dependency. I wanted some geek cred so I set up a .ssh/config file with the 5 or 6 servers I log into daily.

Problem
---------------------
I created a SSH config file in `~/home/.ssh/config` with the following entry in for each server I wanted to access.

```bash
Host Server1
	HostName 555.555.555.555
	User joecool
```

From there, a simple `ssh Server1` would get me SSH’d into the server. However, the server was still asking me for my password, so I set out to get password-less login working.

Run this command if you haven’t already to generate a private key pair

```
ssh-keygen -t rsa
```

Then, assuming you already have a .ssh directory with the keypair created, you can run the following to copy your public key to the `authorized_keys` file in the remote server.

```bash
cat ~/.ssh/id_rsa.pub | ssh user@123.45.56.78 "cat >>  ~/.ssh/authorized_keys"
```

In reality, that should have worked, but it still didn’t…

Solution
-------------------------
To debug, I ran `ssh -v name@host` to get more verbose output. Pro tip: adding more v's adds more verbosity. I.e. `-vvv` is more verbose than `-v`.

Found out that it was trying 3 types of authentication and failing the first two (key based) and failing over to asking me for my server password again.

At that point, I did some googling and ran into [this post](https://bbs.archlinux.org/viewtopic.php?id=103954).

The final answer in that thread stated that in the authorization error logs for the server, it was stated that there was a permission error of the .ssh directory. I wanted to check my servers auth.log file but it wasn’t where they stated it would be (`var/log/auth.log` for Ubuntu). Of course, I was on a CentOS server and the ssh auth logs are located in `/var/logs/secure`. Looking there, I found this log:

```
Jul  2 14:48:19 CS00X sshd[3257]: Authentication refused: bad ownership or modes for directory /home/brock
```

I followed the post's advice and set the permissions of my .ssh files on the remote server like so:

```bash
chmod go-w ~/
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

And viola, it worked like a charm. No more entering passwords!
