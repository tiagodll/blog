---
title: "server update"
date: 2021-09-18T10:23:00+02:00
draft: false
tags: ['Server', 'Ubuntu', 'Blog']
featured_image: https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Logo-ubuntu_no%28r%29-black_orange-hex.svg/500px-Logo-ubuntu_no%28r%29-black_orange-hex.svg.png
description: Keep your ubuntu up to date. Upgrading too late can be a pain...
---

# God dammit ubuntu!

![ubuntu](https://upload.wikimedia.org/wikipedia/commons/thumb/3/3a/Logo-ubuntu_no%28r%29-black_orange-hex.svg/500px-Logo-ubuntu_no%28r%29-black_orange-hex.svg.png)

I have to start by admitting one thing: I use to hate servers. Every time I even had to open a remote connection, that would ruin my mood. Granted, I am a web developer, my work depends on it. But still…

All that is just to explain why I haven’t updated my ancient server for a few years

Ok, time to roll up my… well, I don’t really wear long sleeves… Anyway, I take a deep breath and try to upgrade it.

```bash
sudo do-release-upgrade
```

[Enter]

```bash
Reading cache

Checking package manager

Cannot upgrade 

An upgrade from 'zesty' to 'bionic' is not supported with this tool. 
=== Command terminated with exit status 1 (Sat Sep 18 16:30:40 2021) ===
```

![sad](https://www.jing.fm/clipimg/full/96-960582_art-depressed-white-easy-sad-person-drawing.png)

Ooops Apparently this upgrading tool doesn’t support my version anymore

I tried ducking around (yes, I use duck duck go), but none of the results was working for me

Got in contact with scaleway and they said: - ubuntu is only upgradeable for 2 major versions. Now you have to destroy your instance, and make a new one.

Dammit, now I will have a new instance with new price. My old instance was super cheap because it was that old.

Ok, it was my fault. I will not complain. lets go!

So I got my new instance and started working on it:

1. Create my own user & secure it
2. install and configure nginx and certbot
3. install docker
4. git clone my apps, run devops script
5. spend days and days fixing…

… wait a minute …

it works! 😎

But wait, how is this working? 🤔

it took me months to set up the old server

Simple:
![Docker](https://duckduckgo.com/i/def4b5e6.png)

You see, the old server I had to set up app by app

but when I got 2 more developers to work on my side project, I decided to have it all running on docker, so it would be easy to have it working for everyone

I was expecting it would be easier, but having it all working in minutes was unbelievable.