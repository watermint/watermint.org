---
layout: post
title: 'Cross compiling Go app with go-sqlite3 on Docker'
tags:
 - docker
 - glide
 - go
 - sqlite3
 - go-sqlite3
---

Part of my Go app in my project, [toolbox](https://github.com/watermint/toolbox), using [go-sqlite3](https://github.com/mattn/go-sqlite3). I spent couple of hours to setup cross compiling environment for this.

`go-sqlite3` using CGO for binding sqlite3 to go library. That requires cross compilers for each target platform. First, I tried to setup mingw64, etc for each platform. But it was easy to implement if you are familiar with [xgo](https://github.com/karalabe/xgo).

`xgo` is a Docker images that pre configured for each Go version and target platforms.
If you are using [glide](https://glide.sh) for packaging system. Prepare Dockerfile like below.

```
FROM karalabe/xgo-1.7.x

RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install -y zip git curl

ENV GOBIN=/usr/local/go/bin
ENV PATH=$PATH:/usr/local/go/bin
RUN curl https://glide.sh/get | sh
```

Then, just build with `xgo` command like below.

```
xgo --ldflags="$LD_FLAGS" -targets "windows/*,linux/*,darwin/*" github.com/watermint/toolbox/tools/$t
```
