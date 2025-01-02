---
layout: post
title:  "Creating your own SPA library"
description: "Recreating the wheel in a world full of frameworks. Let's learn how-to create a SPA."
keywords: "javascript, js, vanilla js, spa, tutorial"
date:   2025-01-01 22:20:51 +0800
categories: frontend how-to spa
preview: https://res.cloudinary.com/sudoaldrich/image/upload/v1735741173/blogs/thumbnails/SPA_tuxf1r.png
---

# Learning SPA by building one

JavaScript ecosystem is pretty loaded with front-end frameworks that allows us to create SPAs.

You might be thinking, "Why bother?" or "Don't reinvent the wheel" -- I'd like to learn, to put it simply.

## SPAs

Single Page Application (SPA) is a type of web application that loads a single document
and updates it with the data coming from a web server  or an API instead of reloading the entire page.

The idea is to provide a rich user interface.

## The first problem

The routing. In conventional MPAs, we're using URL path to identify the resource we're trying to load while 
in SPAs we are using it for identifying the component to show in the document.

For this implementation, we'll be using History API's [pushState](https://developer.mozilla.org/en-US/docs/Web/API/History/pushState) function.

```js
history.pushState({}, '', url);
```

With this, we could update what's in our browser's address bar.

Then, we'd like to have an object to store the callbacks for our routes.
A route being just another function that would execute updates on our page.

## Basic routing

What I would like to have is an object wherein we could register a path and it's callback function.

Something like this:
```js
// our home page. by default you could define routes using string
app.add('/', Home);

// or, you could also define a route using Regular Expression with or without named groups
app.add(/\/pages\/(?<id>\d+)/i, Page);
```

Here is the foundation for our application:

1. **Constructor**. Pass down a configuration object that would contain the root document of our SPA application as
  well as a function to handle all 404 pages.
2. **Add**. Register a route using only a path (either string or RegEx) and it's callback function.
3. **Get**. A function to retrieve the function from our `routes` storage.

```js
class SPA {
  routes = [];

  constructor(config = {}) {
    this.context = {
      root: config?.root || document.getElementById('app'),
    };

    this.defaultRoute = {
      key: '*',
      callback: (config?.defaultRoute || (() => { })).bind(this.context),
    };
  }

  add(path, cb) {
    this.routes.push({
      key: path,
      callback: cb.bind(this.context),
    });
  }

  get(path) {
    const route = this.routes.find(r => (r.key instanceof RegExp && r.key.test(path)) || r.key === path);
    return route || this.defaultRoute;
  }
}
```

## Executing a route

Let's us now add function that would parse the matching route. To keep this simple, we'll be
adding support to named groups to easily map the values from the defined route.

By calling this function, we'll be getting the route callback using `get` function and 
execute it while passing the matches from our regular expression.

```js
class SPA {
  // ...
  execute(path) {
    const route = this.get(path);
    let params;

    if (route?.key && route?.key instanceof RegExp) {
      params = route.key.exec(window.location.pathname);

      if (params?.groups && Object.keys(params?.groups).length > 0) {
        params = params.groups;
      } else {
        params = Array.from(params);
        params?.shift();
      }
    }

    route?.callback(params);
  }
  // ...
}
```

## The clicks

How should we handle navigation? When do we decide when to  whether it is time to call `pushState` function?

In this case, I'd like our application to have it only on hyperlinks.
For us to ensure that all links would have the route change callback, we'd like to use
[MutationObserver](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) to observe changes in
our document.
```js
const observer = new MutationObserver((mutationList) => {
  mutationList.forEach((mutation) => {
    mutation?.addedNodes?.forEach(e => {
      if (e.nodeName.toLowerCase() === 'a') {
        // if the node is an actual link
        e.addEventListener('click', handleClick);
      } else {
        // else, find any links in the container
        if (typeof e === 'object' && typeof e.getElementsByTagName !== 'undefined') {
          const as = e.getElementsByTagName('a');

          for (let i = 0; i < as.length; i++) {
            as[i].addEventListener('click', handleClick);
          }
        }
      }
    })
  })
});

observer.observe(document, { attributes: true, childList: true, subtree: true });
```

`handleClick` being the function to execute `pushState` and callback from our routes storage.

It is important to note that for `handleClick` function we should cover:
1. External links
2. Scroll on document if link has fragment.


## Building it together

Here is a working demo of our application.

<iframe src="https://stackblitz.com/edit/vitejs-vite-1zqfcjcy?embed=1&file=src%2Fmain.js"></iframe>

## Conclusion

There's a lot of improvement we could do in this practice application. 
We're only touching the surface of SPA development. There are more concepts to uncover.
This is nowhere near the frameworks we are using all along.

I hope that this tutorial helped you to kick-off your project in mind.


