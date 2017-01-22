---
layout: post
title: 'Go: Bandwidth limit for multiple Reader and Writer'
tags:
- go
- bandwidth
- concurrency
---

Bandwidth limit is really important for stabilizing systems. Core business logic stops if low priority job consume entire bandwidth of the system. When writing bandwidth limit for `Reader` or `Writer` in Go, I found [go-flowrate](https://github.com/mxk/go-flowrate). This library enables easy way to limit bandwidth like the code below.

```go
func main() {
    f, _ := os.Open("data.dat")

    // Create wrapper with 100 bytes per second
    f2 := flowrate.NewReader(f, 100)

    // Biz logic
    // ...
}
```

APIs are pretty simple and reliable. But `go-flowrate` can limit bandwidth only for single `Reader` or `Writer`.
In my case, I was working on file uploader for Dropbox which has concurrent upload feature. This code require limiting bandwidth for multiple I/O. So I decided create new small library for bandwidth limit.

# `bwlimit`

`bwlimit` is the name of my library. You can see or clone from github [bwlimit](https://github.com/watermint/bwlimit). Here is code example of `bwlimit`.

```go
func main() {
    // Limit 100 bytes per second
    bwlimit := NewBwlimit(100, false)

    // Prepare multiple readers
    f1, _ := os.Open("data1.dat")
    f2, _ := os.Open("data2.dat")

    // Create wrapper
    fr1 := bwlimit.Reader(f1)
    fr2 := bwlimit.Reader(f2)

    // Biz logic
    // ...

    // Wait for all Reader to be closed or reach EOF
    bwlimit.Wait()
}
```

1) Prepare bandwidth limiting object (`bwlimit` in above code) first. Second argument is the flag for block(true) or unblock(false) for `Read` or `Write` operation. 2) Then, create wrapper for each `Reader` or `Writer`.

# Concept

The concept of `bwlimit` comes from Toyota's Production System (TPS). In TPS, [Takt time](https://en.wikipedia.org/wiki/Takt_time) is the core concept for leveling production. Define time unit of ship defined quantity of products.

For example; 1) Takt time is 100ms, 2)  bandwidth limit is 1,000 bytes per second. 100 bytes is maximum transferrable data size per takt time. If `bwlimit` object has two `Readers`, that mean 50 bytes per `Reader` per takt time.

# Flow control

`bwlimit` does not carry bandwidth window to next takt time. If the Reader have 50 bytes window per takt time, and the Reader read nothing. Then, window size of next takt time will be 50 bytes. This is because if `bwlimit` carry unused window to next takt time. `bwlimit` may allow burst IO for certain takt time. That sometime cause buffer overflow of routers. To prevent incident caused by burst IO.

