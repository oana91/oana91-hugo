+++
date = "2016-12-22T09:32:16+01:00"
title = "Writing negroni middleware in golang"

+++

Negroni is a great middleware framework for golang that comes with a lot of
functionallity (logging, panic recovery, cors) that one needs to create for almost
each web application. It is very straightforward to use
negroni throughout the development phase of a project, then replace it with custom 
code in production. 
While coding [bago](https://github.com/MrToBe/bago), a small basic authentification
middleware for negroni, I could not find any tutorials on how to write a
negroni framework. So there we go:

## What is a (negroni) middleware?

First things first. A middleware is just some code that is executed before a
http request reaches its handler on a server. For example in an online banking
application a user requests the server to show his balance. Before showing the
balance the server needs to authenticate the user. Therefore one could write a
authentification middleware that checks the username and password. 

Doing authentification in a middleware instead of the handler has the huge
advantage of re-use. In a typical RESTful API we would need to implement the
authentification code once for each http method get, post, put, delete. By using
a middleware we can handle authentification before the request hits the handler.

## Setup

To get going you need to create a project folder under your go/src directory. To
start coding it only requires one .go file.

First it is necessary to name the package of the file:

``` go
package middleware
```

Here I named the package middleware, but you can pick any name you like. After
this it is necessary to create a struct that implements the negroni middleware
interface. In golang this means we need to create a struct and attach some
functions to it.

``` go
type Middleware struct {
    variable string
}
```

I picked the name Middleware, but you could choose whatever you like. The only
restriction is, that the name needs to be capitalized. If your middleware
requires some configuration or in-memory storage you can setup some variables
within the struct. For an authentification software you could for example hold the 
correct username or password.

After that you would typically setup a function that creates a new instance of
your Middleware struct. Therefore I use the golang convention of a New()
function that returns a pointer to the new instance. This allows you for example
to validate the input before setting the variables.

``` go
func New(variable string) *Middleware {
       return &Middleware{variable}
}
```

## The middleware

After these setup steps you can focus on the actual middleware logic. This is
done by attaching a ServeHTTP function to the Middleware struct:

``` go
func (m *Middleware) ServeHTTP(w http.ResponseWriter, r *http.Request, n http.HandlerFunc) {
    // This code is executed before the next handler
    n(w,r)
    // This code is executed after the next handler
}
```

This function allways needs to be named ServeHTTP to implement the negroni
middleware interface. Then the magic happens with the n(w, r) part of the
function. What negroni does is just passing one handler to another to build a
chain of functionallity that is executed on the http.ResponseWriter w and 
*http.Request r. In an authentification middleware one would for example check the
credentials in the header before running the next handler via n(w, r). This next
handler could be an endpoint handler or another middleware. 

What is great about this setup is that you can execute code before and after the
handler. That way you could for example write a middleware that checks the
request and then validates if the response actually makes sense.

## Conclusion

Even so the golang community tries to stay away from frameworks, for me it makes
sense to use frameworks for middleware as this functionallity is usally pretty
generic. Therefore negroni is a lot of fun and it fits perfect into the standard
lib without limiting you from using other libraries. Writing middleware for
negroni is straightforward and simple. There is no magic involved.
