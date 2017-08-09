+++
categories = ["posts"]
date = "2014-11-18T11:08:48+01:00"
title = "Go Interfaces"
+++

## Introduction

Interfaces describe the behavior of an object. An interface is basically a set of methods which can be satisfied by
a concrete type. Typically, Go interfaces are pretty small, most of them contain only one or a few methods. Examples
which are often mentioned are the [`io.Reader`](https://golang.org/pkg/io#Reader) and [`io.Writer`]
(https://golang.org/pkg/io#Writer) interfaces from the [`io`](https://golang.org/pkg/io) package
of the standard library:
```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
```

So every type is a `Reader` that implements the method `Read`, which takes a slice of bytes and returns an integer
and an error. To define an interface which can read and write, both interfaces are combined to a [`io.ReadWriter`]
(https://golang.org/pkg/io#ReadWriter):
```go
type ReadWriter interface {
    Reader
    Writer
}
```

Types like the [`gzip.Reader`](https://golang.org/pkg/compress/gzip#Reader.Read) and [`gzip.Writer`]
(https://golang.org/pkg/compress/gzip#Writer.Write) satisfy these interfaces by reading and writing data of gzip
compressed data.

The `gzip.Writer` compresses the data and writes it to another `Writer`. The `gzip.Reader` does the opposite with
another `Reader`. The [`os.File`](https://golang.org/pkg/os#File) type is a `ReadWriter` and is used to read and write data from disk. So we can
stitch together our own type named `GZIPFile`, which reads and writes gzip compressed data from a file:
```go
type GZIPFile struct {
    filename string
}

func (g GZIPFile) Write(p []byte) (int, error) {
    f, err := os.Create(g.filename)
    if err != nil {
        return 0, err
    }
    defer f.Close()
    w := gzip.NewWriter(f)
    defer w.Close()
    return w.Write(p)
}

func (g GZIPFile) Read(p []byte) (int, error) {
    f, err := os.Open(g.filename)
    if err != nil {
        return 0, err
    }
    defer f.Close()
    r, err := gzip.NewReader(f)
    if err != nil {
        return 0, err
    }
    return r.Read(p)
}

func main() {
    data := []byte("Hello, World!\n")
    f := GZIPFile{"foo"}
    _, err := f.Write(data);
    if err != nil {
        log.Fatal(err)
    }

    b := make([]byte, len(data))
    _, err = f.Read(b)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(string(b))
}
```

The code is also available in the [Go playground](https://play.golang.org/p/cewWFFxIyx).

There might be better ways to read and write gzipped data, but the interesting part of the example is, that the type
`GZIPFile` is again a `ReadWriter`. This gives a feeling about how interfaces are used in Go.

More information on Go interfaces in general can be found in the [spec]
(https://golang.org/ref/spec#Interface_types), the [Effective Go]
(https://golang.org/doc/effective_go.html#interfaces_and_types) guide and in an excellent [article]
(https://talks.golang.org/2012/splash.article#TOC_15.) by Rob Pike.

## Implicitness

Interfaces in Go behave a bit different than interfaces in other languages like C# or Java. As you might notice, in
the example given above, the 'implements' relationship is never declared explicitly.

Interfaces are satisfied implicitly. This let you forget about a type hierarchy, like you would design in C# or
Java. Go has a focus on composition and doesn't use inheritance. In this way, you can adapt your code easily without
massive refactoring of the whole type hierarchy. You can start to write straight-forward code and introduce
interfaces as they show up.

You have a kind of [duck typing](https://en.wikipedia.org/wiki/Duck_typing) available in Go which some might know
from dynamically typed languages like Ruby of Python. But in contrast to these languages, Go's interfaces are
statically checked at compile time. So you get the best out of both worlds.

There is a nice [blog post](https://blog.golang.org/json-rpc-tale-of-interfaces) on how this implicitness simplified
refactoring of code in the standard library.

## The Empty Interface

The `interface{}` type is the empty interface, it has no methods. Because there is no explicit 'implements'
declaration, every type satisfies this interface. So a function like this:
```go
func Foo(x interface{}) {
    fmt.Println(x)
}
```

takes any argument regardless of its type.

One common misunderstanding is that you cannot convert a slice `[]T` to a slice of empty interfaces `[]interface{}` 
([playground code](https://play.golang.org/p/_RHf7bCuxQ)):
```go
func Print(vals []interface{}) {
    for _, val := range vals {
        fmt.Println(val)
    }
}
Print([]int{0, 1, 1, 2, 3, 5}) // fails to compile
```

`[]T` cannot be converted to `[]interface{}` because `[]T` and `[]interface{}` do not have the same representation
in memory. You have to copy the the slice manually to a slice of empty interfaces ([playground code]
(https://play.golang.org/p/rAPuLSVk41)):
```go
src := []int{0, 1, 1, 2, 3, 5}
dst := make([]interface{}, len(src))
for i, v := range src {
    dst[i] = v
}
Print(dst)
```

This issue is covered in the [FAQ](https://golang.org/doc/faq#convert_slice_of_interface) on [golang.org]
(https://golang.org).

## Implementation

An interface value consists of two words of data. One is used to point to the actual data, the other points to a
table with type information. This table, called `itable`, holds, among other things, a list of function pointers
to methods which are used to satisfy an interface.

The compiler doesn't precompute all possible `itable` at compile time. Instead the runtime computes the `itable`
with the list of methods of the concrete types and interfaces provided by the compiler. To avoid the re-computation
on each method invocation, the Go runtime computes the `itable` when the value is stored in the interface.

[Russ Cox's blog post](http://research.swtch.com/interfaces) about interfaces is extremely interesting and provides
 more information about the implementation.

## Caveats

Go's interfaces can have some unexpected properties if you are not used to them. The first one happens in context of
`nil`.

Imagine we have a type `Point`, which satisfies the [Stringer](https://golang.org/pkg/fmt#Stringer) interface:
```go
type Point struct {
    x, y int
}
func (p *Point) String() string {
    return fmt.Sprintln(p.x, p.y)
}
```

The value of an uninitialized pointer is `nil`:
```go
var p *Point
fmt.Println(p, p == nil) // <nil> true
```

And also the value of an uninitialized interface is `nil`:
```go
var s fmt.Stringer
fmt.Println(s, s == nil) // <nil> true
```

But if we assign nil to an interface value:
```go
var p *Point
var s fmt.Stringer = p
fmt.Println(s, s == nil) // <nil> false
```

As mentioned, an interface consists of a pointer to the data and another pointer to the type information. An
uninitialized interface value consists of two `nil` pointers (`nil`, `nil`), because neither its data nor its type
information was initialized.

But in the last example, we assigned a pointer of type `*Point` to the interface value. This changed the underlying
structure to this: (`*Point`, `nil`). So the interface value is therefore not `nil` any more, even if the pointer
inside is `nil`.

The whole code of the example is available in the [playground](https://play.golang.org/p/Q1eq55oolP). The [FAQ]
(https://golang.org/doc/faq#nil_error) on [golang.org](https://golang.org) also explains this issue.

Another accidental problem can occur if the receiver type of a method is a pointer.
To implement the Shape interface:
```go
type Shape interface {
    Area() int
}
```

With type `Rect`:
```go
type Rect struct {
    l, w int
}
```

we can define a method `Area()` which has type `Rect` as receiver ([playground code]
(https://play.golang.org/p/RaseXia6o2)):
```go
func (r Rect) Area() int {
    return r.l * r.w
}
```

If we use a `Rect` value or pointer as `Shape` in a function:
```go
func PrintArea(s Shape) {
    fmt.Println("area:", s.Area())
}
PrintArea(&Rect{1, 2})
PrintArea(Rect{3, 4})
```

it works fine.
But if we use a pointer type as receiver for this method ([playground code](https://play.golang.org/p/cFktdN7T79)):
```go
func (r *Rect) Area() int {
    return r.l * r.w
}

PrintArea(&Rect{1, 2})
PrintArea(Rect{3, 4})
```

The compilation fails. But why?

In Go you can always get the value of a pointer. This is the reason why the first example works. But to take an
address from a value is not always possible. So a value of some type `T` does not satisfy an interface if the method
to implement has a receiver `*T`.

Both caveats were very well explained in a [talk](https://www.youtube.com/watch?v=B-r3Wf_I2Lk) of Francesc Campoy
Flores at dotGo 2014.

## Conclusion

Go interfaces are nice to use and give Go a lightweight feel. If you come from a C# or Java background it might take
some time until you get used to Go interfaces. If you have used dynamic languages, you might already know how to use
this kind of duck-typing and you get static type checking at compile time for free.

The implicitness of Go's interfaces are, for me, one of the greatest features of this language.
