---
layout: post
title: NGINX Static Routing
---

I ran into a problem with routing between NGINX and my React Router. In this app, I was running a prebuilt static JS file. The problem I ran into was with NGINX and how it handles static routing. A typical static location in the NGINX config looks like this:

```
location / {
    root /my/app/;
}
```

Using localhost as an example, the route would look like this `http://localhost:8080/` and would serve up the `index.html` file in the `/my/app/` folder. But what happens when I want the route `http://localhost:8080/page`? React router will handle that route, but NGINX throws a 404 because no `page` file exists in `/my/app/`. I needed a way to send any requests through NGINX and to the same html file in the `/my/app/` folder.

## Enter `try_files`

`try_files` is NGINX way of defaulting to a file if no match is found. To default to the `index.html` in the example above, I just add the `try_files` config to route to `index.html` if the path isn't found.

```
location / {
    root /my/app/;
    try_files $uri /index.html;
}
```

Now, when any route comes into the root route, it gets sent to `index.html` and that's where React Router can handle the route appropriately. But that's not the end of the story. Only adding the `try_files` config threw an error:

```
rewrite or internal redirection cycle while internally redirecting to "/index.html"
```

To solve this problem, I added a second route to handle the `/index.html` route.

```
location / {
    root /my/app/;
    try_files $uri /index.html;
}

location = /index.html {
    root /my/app/;
    expires 30s;
}
```
That stops the redirection cycle. You can find out more about all this in the [NGINX documentation](http://nginx.org/en/docs/http/ngx_http_core_module.html#try_files).