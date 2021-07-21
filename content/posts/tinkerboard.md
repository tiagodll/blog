---
title: "Tinkerboard"
date: 2017-05-20T07:30:29Z
draft: false
---

Finally my Asus tinkerboard arrived.

![Thinkerboard](https://www.distrelec.ch/Web/WebShopImages/landscape_large/5-/01/Asus_TinkerBoard_30083025-01.jpg)


Is it awesome?
not sure yet...

Yes, it packs more power than raspberry pi, but it falls behind in so many things

The software is basically inexistent
You get a standar linux and nothing more
With no further ado, lets get it going.

step 1: ssh to it
	{{< highlight bash >}}
	ssh linaro@your_ip
	{{< /highlight >}}
	
	pwd: linaro

step 2: install go

	{{< highlight bash >}}
	wget https://storage.googleapis.com/golang/go1.8.1.linux-armv6l.tar.gz
	tar -C /usr/local -xzf go1.8.1.linux-armv6l.tar.gz
	export PATH=$PATH:/usr/local/go/bin
	{{< /highlight >}}

... aaaaaaannnd I am out of space :'(

something is not right here, I have a 32gb sd card,
I run fdisk and check that indeed my partition is 29gb
but somehow my /root is locked to 2gb

well, anyway, lost a day playing around with that,
in the end formatted my sd card before loading the image
and it works.