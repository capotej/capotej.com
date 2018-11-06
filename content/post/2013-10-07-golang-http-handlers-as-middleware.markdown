---
title: "Golang http handlers as middleware"
date: 2013-10-07T08:52:00Z
comments: true
tags: ["go", "http"]
---

Most modern web stacks allow the "filtering" of requests via stackable/composable middleware, allowing you to cleanly separate cross-cutting concerns from your web application. This weekend I needed to hook into go's ```http.FileServer``` and was pleasantly surprised how easy it was to do.

<!--more-->

Let's start with a basic file server for ```/tmp```:

```go main.go
func main() {
    http.ListenAndServe(":8080", http.FileServer(http.Dir("/tmp")))
}
```

This starts up a local file server at :8080. How can we hook into this so we can run some code before file requests are served? Let's look at the method signature for ```http.ListenAndServe```:

```go
func ListenAndServe(addr string, handler Handler) error
```

So it looks like ```http.FileServer``` returns a ```Handler``` that knows how to serve files given a root directory. Now let's look at the ```Handler``` interface:

```go
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

Because of go's granular interfaces, any object can be a ```Handler``` so long as it implements ```ServeHTTP```. It seems all we need to do is construct our own ```Handler``` that wraps ```http.FileServer```'s handler. There's a built in helper for turning ordinary functions into handlers called ```http.HandlerFunc```:

```go
type HandlerFunc func(ResponseWriter, *Request)
```

Then we just wrap ```http.FileServer``` like so:

```go main.go
func OurLoggingHandler(h http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    fmt.Println(*r.URL)
    h.ServeHTTP(w, r)
  })
}

func main() {
    fileHandler := http.FileServer(http.Dir("/tmp"))
    wrappedHandler := OurLoggingHandler(fileHandler)
    http.ListenAndServe(":8080", wrappedHandler)
}
```

Go has a bunch of other builtin [handlers](http://golang.org/pkg/net/http/#Handler) like [TimeoutHandler](http://golang.org/pkg/net/http/#TimeoutHandler) and [RedirectHandler](http://golang.org/pkg/net/http/#RedirectHandler) that can be mixed and matched the same way.
