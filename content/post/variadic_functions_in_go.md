+++
categories = ["misc"]
date = "2014-12-03T10:22:35+01:00"
title = "Variadic Functions in Go"
+++

## Introduction

Go has the notion of [variadic functions](https://golang.org/ref/spec#Function_types). Such a function has a
parameter with a type prefixed by `...`. Variadic functions take zero or more arguments of that type
([playground code](https://play.golang.org/p/dPPaiaDP8Q)):
```go
func Variadic(args ...string) { /* ... */ }

Variadic()
Variadic("one", "two", "three")
```

In the function called, the parameter of type `...string` is equivalent to `[]string`. The arguments passed are
copied into a new slice ([playground code](https://play.golang.org/p/Ksdz7slchm)). If you call the function with
zero arguments, the slice is `nil` ([playground code]
(https://play.golang.org/p/hz1IZIHusV)).

You can also pass a slice to the variadic function. Therefore you append `...` to the slice in the function call
([playground code](https://play.golang.org/p/6YeJIFdR33)):
```go
vars := []string{"one", "two", "three"}
Variadic(vars...)
```

In this case the slice does not get copied ([playground code] (https://play.golang.org/p/lLIVMIswoO)).

There is one limitation in regard to variadic functions. There must be only one parameter with a type `...T` and it
 must be the final one. That means you can have a function like this ([playground code]
(https://play.golang.org/p/Ls_wKaZ8uN)):
```go
func Variadic(prefix string, args ...string) { /* ... */ }
```

But a function like that will not compile ([playground code](https://play.golang.org/p/x2QwAuMbt2)):
```go
func Variadic(a ...string, b ...string) {}
```

The reason is, that there is no way to distinguish between `a` and `b` in a function call like this:
```go
Variadic("one", "two", "three")
```

`"one", "two", "three"` could all belong to `a` or `b` or be distributed in any way between these two.

## Usage

One nice usage of variadic functions is to create new instances of a type. Consider the following function:
```go
func NewThing(config *Config) *Thing
```

This API is useful to create a configured thing. But how can you create a default thing? Pass `nil`, pass
`&Config{}`?

If the API looks like this:
```go
func NewThing(config ...Config) *Thing
```

you can get the default thing by calling the function with no arguments. This feels way more natural.

This only scratched the surface of an idea I discovered in a [blog post by Dave Cheney]
(http://dave.cheney.net/2014/10/17/functional-options-for-friendly-apis) which explains this usage of variadic
functions in much more detail. He demonstrates the usage of variadic functions in connection with functions as
parameters to create user-friendly APIs. He also gave a [talk](https://www.youtube.com/watch?v=24lFtGHWxAQ) about
the topic of functional options at dotGo 2014. I would encourage you to read the blog post or watch the video.
