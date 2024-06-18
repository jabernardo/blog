---
layout: post
title:  "Creating an HTTP server in Go"
description: "This is a tutorial on how-to create an HTTP server in Go. Specifically, Go 1.22 as it introduced enhanced pattern routing."
date:   2024-06-18 10:05:51 +0800
categories: go-lang how-to http server
---

## Introduction

This tutorial would show you how to create a basic http server in Go lang.
For this we'll need [Go 1.22](https://tip.golang.org/doc/go1.22) as it offers built-in enhanced routing that would save us time in thinking what framework should we use.

#### Folder Structure

- `go.mod`
- `go.work`
- `main.go`
- handlers/
  - `home.go`
  - `articles.go`

### Initilizing Project

#### Create project.

{% highlight bash %}
$ mkdir http-server; cd http-server;
{% endhighlight %}

#### Initialize project.

{% highlight bash %}
$ go mod init http-server;
$ go work init .; # enable workspaces
{% endhighlight %}


#### Creating the router

`NewServeMux` would return a `ServeMux` that would serve us our router for the application.
Let's create barebone http server first to try it out.

{% highlight go %}
package main

import (
	"fmt"
	"log"
	"net/http"
)

func HomeRoute(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello World!")
}

func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("/{$}", HomeRoute) // <--- `/{$}` only matches `/`

	err := http.ListenAndServe(":3000", mux)

	if err != nil {
		log.Fatalln("Cannot run the server", err)
	}
}
{% endhighlight %}

#### Method Specific Route

To filter specific request method to a route, we could use a prefix to identify the target request method.
{% highlight diff %}
- mux.HandleFunc("/{$}", HomeRoute)
+ mux.HandleFunc("GET /{$}", HomeRoute)
{% endhighlight %}

[Learn more about ServeMux patterns](https://pkg.go.dev/net/http#hdr-Patterns)

Testing the application...

> GET method has response
{% highlight bash %}
$ curl localhost:3000
Hello World!
{% endhighlight %}


> POST method throws an error
{% highlight bash %}
$ curl -XPOST localhost:3000
Method Not Allowed
{% endhighlight %}

#### Getting Path Value

One of the biggest game changer introduced in [Go 1.22](https://tip.golang.org/doc/go1.22) is the ability to read url paths.

{% highlight go %}
func ArticlesRoute(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Reading article", r.PathValue("article"))
}
{% endhighlight %}

Register the path:

{% highlight go %}
mux.HandleFunc("/articles/{article}", ArticlesRoute)
{% endhighlight %}

What do we have now?

{% highlight diff %}
package main

import (
	"fmt"
	"log"
	"net/http"
)

func HomeRoute(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello World!")
}

+ func ArticlesRoute(w http.ResponseWriter, r *http.Request) {
+	fmt.Fprintln(w, "Reading article", r.PathValue("article"))
+ }

func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("GET /{$}", HomeRoute)
+	mux.HandleFunc("/articles/{article}", ArticlesRoute)

	err := http.ListenAndServe(":3000", mux)

	if err != nil {
		log.Fatalln("Cannot run the server", err)
	}
}
{% endhighlight %}

Testing the new route

{% highlight bash %}
$ curl localhost:3000/articles/hello-world
Reading article hello-world
{% endhighlight %}

#### Organizing our project

To better organize our code, we'll create a new module.

{% highlight bash %}
mkdir handlers;
{% endhighlight %}

Then move our route to each separate module inside `handlers`

{% highlight go %}
// ./handlers/home.go
package handlers

import (
	"fmt"
	"net/http"
)

func HomeRoute(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello World!")
}
{% endhighlight %}

{% highlight go %}
// ./handlers/articles.go
package handlers

import (
	"fmt"
	"net/http"
)

func ArticlesRoute(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Reading article", r.PathValue("article"))
}
{% endhighlight %}


Update our `main.go` to read the `handlers` submodule.

{% highlight go %}
package main

import (
	"http-server/handlers"
	"log"
	"net/http"
)

func main() {
	mux := http.NewServeMux()

	mux.HandleFunc("GET /{$}", handlers.HomeRoute)
	mux.HandleFunc("/articles/{article}", handlers.ArticlesRoute)

	err := http.ListenAndServe(":3000", mux)

	if err != nil {
		log.Fatalln("Cannot run the server", err)
	}
}
{% endhighlight %}

## Conclusion

Here we have learned about how to create a basic HTTP server using Go and how to utilize.
Hoping that this tutorial helped kickstart your dream project in Go.

To see the full project in this tutorial [click here](https://github.com/jabernardo/tuts/tree/main/http-server).
