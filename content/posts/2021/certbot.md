---
title: "Certbot"
date: 2021-09-08T10:23:00+02:00
draft: false
tags: ['devops', 'server', 'certbot']
featured_image: https://certbot.eff.org/assets/certbot/Certbot-solid-c4e500f9953fc8ee1d38cb0b22778163602a82cb2b39a5bc89211315c5c877c9.svg
description: I <3 certbot. Managing a server? not so much
---

## my certificate is expired and certbot 

![OMG](http://www.evilenglish.net/wp-content/uploads/2014/05/1851270709_OMG_83811072751_xlarge.jpg)

This gives me a message "your current client, does not support ACME v2 at all. You will need to migrate to a different one", but there is nothing I can do to update my certbot.

So I did some googling around, and found nothing that helped me.

It is worth to note that this was in an old server with ubuntu 17.

Now there is one thing I can do: remove it and install it again!

step 1: removing
```bash
sudo apt remove certbot
...
certbot removed!
```

step 2: install it again
```bash
sudo apt install certbot
...
...
error!
```
![Oh noes](https://cdn.iconscout.com/icon/premium/png-256-thumb/oh-no-2123278-1787781.png)
fuuuuuuuuuuuuuuuuuuu!!!!

ok, no panic. lets check the [documentation](https://certbot.eff.org/lets-encrypt/ubuntuxenial-nginx).

```bash
sudo snap install --classic certbot
```

Well, I am not a fan of snap, usually try to keep as much as possible with flatpack, but let's try

hey, it worked!

so I run
```bash
sudo systemctl stop nginx.service
sudo certbot renew
sudo systemctl start nginx.service
```

open the website and
...
bob is your uncle!

![success](/resources/4a8d9ed773cdd5b623416a0549f7d6e8.png)