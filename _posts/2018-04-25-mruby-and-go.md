---
layout: post
title: 'mruby and go: value mapping library'
tags:
 - mruby
 - go
---

I'm recently working on my hobby project that requires DSL on Go language. From my little research of DSL on Go; [go-mruby](https://github.com/mitchellh/go-mruby/) looks attractive to implement DSL. [mruby](https://github.com/mruby/mruby) is the lightweight version of Ruby. That is embeddable into an app.
With go language, an app can be compilable into a single binary. 

# go-mruby

`go-mruby` have set of functions that enable essential integration with mruby code and Go. For example, `go-mruby` can define mruby class or method from Go.
But current functions are it's too primitive for dealing with real-world applications.

Like mapping values from mruby to Go, or vice versa. It's a bit tiresome for getting/putting each struct field. For example below, just for two fields of struct require 9 lines of code.

```go
mrb := mruby.NewMrb()
defer mrb.Close()

type Planet struct {
    Id   int
    Name string
}

rv, err := mrb.LoadString(`
class Planet
  attr_accessor :id
  attr_accessor :name
end

p = Planet.new
p.id   = 3
p.name = "Earth"
p
`)
if err != nil {
    panic(err.Error())
}

p := Planet{}
if rId, err := rv.Call("id"); err != nil {
    panic(err.Error())
} else {
    p.Id = rId.Fixnum()
}
if rName, err := rv.Call("name"); err != nil {
    panic(err.Error())
} else {
    p.Name = rName.String()
}
fmt.Printf("Planet: %v\n", p)
```

For more productivity, I wrote mapping functions. Functions are open sourced on GitHub in MIT license. [grb](https://github.com/watermint/grb). 

# `grb`

## Unmarshal Ruby values into Go struct data

Func `Unmarshal` fetch values for every field in Ruby object that defined by `mruby:"METHOD_NAME"` tag. 

```go
type Planet struct {
    Id         int            `mruby:"id"`
    Name       string         `mruby:"name"`
    Radius     float64        `mruby:"radius"`
    HasMoon    bool           `mruby:"has_moon"`
    Moon       []string       `mruby:"moon"`
    MoonRadius map[string]int `mruby:"moon_radius"`
}
```

Define each value accessor in Ruby class.

```ruby
class Planet
  attr_accessor :id
  attr_accessor :name
  attr_accessor :radius
  attr_accessor :has_moon
  attr_accessor :moon
  attr_accessor :moon_radius
end
```

Retrieve ruby value `*ruby.MrbValue`, then unmarshal it into the Go struct.

```go
rv, _ := mrb.LoadString(`
p = Planet.new
p.id          = 3
p.name        = "Earth"
p.radius      = 6371.0
p.has_moon    = true
p.moon        = ["Moon"]
p.moon_radius = {"Moon" => 1737}
p
`)

p := Planet{}
Unmarshal(rv, &p)

fmt.Printf("Planet: %v\n", p)
// Planet: {3 Earth 6371 true [Moon] map[Moon:1737]}
```

## Decode/encode between Ruby values and Go values

Similar to func `Unmarshal`. Func `DecodeMrbValue` and `EncodeMrbValue` enable mapping values between mruby and Go.
`DecodeMrbValue`/`EncodeMrbValue` have limitation for a type of value.
This supports JSON equivalent types like below.

* nil
* bool
* int
* float
* string
* Array / slice
* Hash / map (Hash/map key must be a string)

```go
mrb := mruby.NewMrb()
defer mrb.Close()

// Mapping to mruby value
rv, err := EncodeMrbValue(mrb, map[string]interface{}{"Earth": 6371, "Moon": 1737})

// Mapping from mruby value
gv, err := DecodeMrbValue(rv)
```
