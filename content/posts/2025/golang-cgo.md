---
title: "Crosscompiling golang with CGO"
date: 2025-07-03T16:55:00+02:00
draft: false
tags: ['go', 'golang', 'server', 'linux']
description: How can you crosscompile from macos apple silicon to linux amd64
---

This story started about 3 years ago, when I got a macbook with an M1 processor.


Up until then, I have been compiling my golang apps in a amd64 linux, and deploying to the same architecture.

> but cross compiling in golang is so simple, all you have to do is to set 2 environment variables!

well, yeah, it was super easy.

```bash
ENV GOOS=linux
ENV GOARCH=amd64
```

Till I tried to use CGO.


## The problem


When you have an application using cgo, it has to be compiled with that flag.

```bash
ENV CGO_ENABLED=1
```

And then it fails.

At the time I tried to go after a solution for it, but couldnt make it work.

It even made me question why I am using golang.

With dotnet, or java I never had this type of issue.

But then I realised
> That is the cost of native build

Of course dotnet and java dont deal with this issue, they have a vm runing the code.
Sure, it is easier to compile, but then you are running your code on this VM and not your server, and there is a high cost for that.


## But why CGO?


CGO is golang integration with C.

Not always needed, but for some libraries (like sqlite), it is important


## SOLUTION 1: But why dont you use another libraries?

Yes I did.

For a while I was using alternatives.

But they are not as good as direct access to sqlite.

On top of that, sometimes you find an ideal lib that uses CGO, and you should not have to ignore that.

## SOLUTION 2: Docker

This is the solution I found last year.

I did implement it, but was not happy about it.

Until yesterday, when I decided to dig further and found a great alternative.

At first I got a way:
```Dockerfile
FROM --platform=linux/amd64 golang:1.24.4-alpine AS builder

RUN apk add --no-cache gcc musl-dev

ENV GOOS=linux
ENV GOARCH=amd64
ENV CGO_ENABLED=1

WORKDIR /build

COPY ./src .
RUN go mod tidy
RUN go build .
```

```bash
docker buildx build -t cross-compiler .
CONTAINERID=$(docker ps -alq)
docker cp $CONTAINERID:/build/pnorcoid .
docker stop $CONTAINERID
```

But that would mean I have to run a container after the build.

So I looked into further and got this other solution.

Create this dockerfile in the root folder:
```Dockerfile
FROM --platform=linux/amd64 golang:latest AS builder

RUN apt-get update && apt-get install -y --no-install-recommends gcc libc6-dev

WORKDIR /build

ENV GOOS=linux
ENV GOARCH=amd64
ENV CGO_ENABLED=1

COPY ./src .
RUN go mod tidy
RUN go build -o pnorcoid .

FROM --platform=linux/amd64 scratch
COPY --from=builder /build/pnorcoid .
```
and then run it with this params:

```bash
DOCKER_BUILDKIT=1 docker build --target artifact --output type=local,dest=. .
```

Now we compile in the architecture and platform we want and with cgo enabled.
All good right?

Well...

## SOLUTION 3: Make it work in macos silicon

When I ran my app, I got an error message, and after such a long time investigating this compilation erros, I just assumed it was still the same.

I had no idea what else to do, so I decided to ask gemini (it belongs to google after all, just like golang)

>  Dear gemini, how do I compile golang from apple silicon to linux/amd64 when I need cgo_enabled?

And indeed it recommended to use docker to compile.

But it gave me an alternative: Native cross-compilation on macos.

I was already tired, and about to give up, but it was so simple I decided to try.

```bash
brew tap messense/macos-cross-toolchains
brew install x86_64-unknown-linux-gnu

env CC="x86_64-linux-gnu-gcc" GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go build .
```
and it works.

## CONCLUSION

Now the question is: why is the docker solution recommended?

I am not sure.

I understand with an amd64/linux compilation, it makes more sense, but I rather not have to use docker unless I have to.

So for now, I will keep using this native build.

If it stops working, we have another solution just above.
