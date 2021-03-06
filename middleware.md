# Middleware

**Quick view**

```go
// First mount static files
iris.StaticWeb("/assets", "./public/assets")

// Then declare which middleware to use (custom or not)
iris.Use(myMiddleware{})
iris.UseFunc(func(ctx *iris.Context){})

// declare any finish middleware/ runs always at the end of the request using .Done/.DoneFunc
iris.DoneFunc(executeLast)

// Now declare routes
iris.Get("/myroute", func(ctx *iris.Context) {
    // do stuff
})
iris.Get("/secondroute", myMiddlewareFunc, myRouteHandlerfunc)

// Now run the server
iris.Listen(":8080")


// executeLast func middleware

func executeLast(ctx *iris.Context){
    println("before close the http request")
}

// myMiddleware will be like that

type myMiddleware struct {
  // your 'stateless' fields here
}

func (m myMiddleware) Serve(ctx *iris.Context){
  // ...
}

```

Middlewares in Iris are not complicated to implement, they are similar to simple Handlers.

They implement the Handler interface as well:

```go
type Handler interface {
    Serve(*Context)
}
type Middleware []Handler
```

Handler middleware example:

```go

type myMiddleware struct {}

func (m myMiddleware) Serve(ctx *iris.Context){
    shouldContinueToTheNextHandler := true

    if shouldContinueToTheNextHandler {
        ctx.Next()
    }else{
        ctx.Text(403,"Forbidden !!")
    }

}

iris.Use(&myMiddleware{})

iris.Get("/home", func (ctx *iris.Context){
    ctx.HTML(iris.StatusOK,"<h1>Hello from /home </h1>")
})

iris.Listen(":8080")
```

HandlerFunc middleware example:

```go

func myMiddleware(ctx *iris.Context){
    ctx.Next()
}

iris.UseFunc(myMiddleware)

```

HandlerFunc middleware for a specific route:

```go

func mySecondMiddleware(ctx *iris.Context){
    ctx.Next()
}

iris.Get("/dashboard", func(ctx *iris.Context) {
    loggedIn := true
    if loggedIn {
        ctx.Next()
    }
}, mySecondMiddleware, func (ctx *iris.Context){
    ctx.Writef("The last HandlerFunc is the main handler, everything before that is middleware for this route /dashboard")
})

iris.Listen(":8080")

```

> Note that middlewares must come before route declarations.

Make use of the [middleware](https://github.com/iris-contrib/middleware) package, view practical [examples here](https://github.com/iris-contrib/examples).

```go
package main

import (
 "github.com/kataras/iris"
 "github.com/iris-contrib/middleware/logger"
)

type Page struct {
    Title string
}

iris.Use(logger.New())

iris.Get("/", func(ctx *iris.Context) {
    ctx.Render("index.html", Page{"My Index Title"})
})

iris.Listen(":8080")
```

**Done\/DoneFunc**

```go
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
    ctx.Writef("1. This is the first middleware, before any of route handlers \n")
    ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
    ctx.Writef("2. This is the second middleware, before the '/' route handler \n")
    ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
    ctx.Writef("3. This is the 3rd middleware, after the '/' route handler \n")
    ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
    ctx.Writef("4. This is ALWAYS the last Handler \n")
}

func main() {

    iris.UseFunc(firstMiddleware)
    iris.DoneFunc(lastAlwaysMiddleware)

    iris.Get("/", secondMiddleware, func(ctx *iris.Context) {
        ctx.Writef("Hello from / \n")
        ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
    }, thirdMiddleware)

    iris.Listen(":8080")

}


```

**Done\/DoneFunc with Parties**

```go
// Package main same as middleware_2 but with party
package main

import "github.com/kataras/iris"

func firstMiddleware(ctx *iris.Context) {
    ctx.Writef("1. This is the first middleware, before any of route handlers \n")
    ctx.Next()
}

func secondMiddleware(ctx *iris.Context) {
    ctx.Writef("2. This is the second middleware, before the '/' route handler \n")
    ctx.Next()
}

func thirdMiddleware(ctx *iris.Context) {
    ctx.Writef("3. This is the 3rd middleware, after the '/' route handler \n")
    ctx.Next()
}

func lastAlwaysMiddleware(ctx *iris.Context) {
    ctx.Writef("4. This is ALWAYS the last Handler \n")
}

func main() {

    // with parties:
    myParty := iris.Party("/myparty", firstMiddleware).DoneFunc(lastAlwaysMiddleware)
    {
        myParty.Get("/", secondMiddleware, func(ctx *iris.Context) {
            ctx.Writef("Hello from /myparty/ \n")
            ctx.Next() // .Next because we 're using the third middleware after that, and lastAlwaysMiddleware also
        }, thirdMiddleware)

    }

    iris.Listen(":8080")

}


```

> Done\/DoneFuncs are just last-executed handlers, like Use\/UseFunc the children party inheritates these 'done\/last' handlers too.
