---
layout: post
title: Guidance For New Software Developers
---

I've enjoyed my career in software development and I reguraly get asked where new devs should spend their time and focus. Software development is complicated and changes fast. As a developer, your best bet is to rise above the rapid pace and spend some time on key fundementals that will help you grow no matter what path you've chosen to follow. I spend a lot of time guiding members of my team to try to help them build some intuition around what actions they can take that will help them to level up quickly. Here are my thoughts, right or wrong, on what I believe makes the best developers.

## Read
Read! Read books. Read tweets. Read RFCs. Read articles. Read newsletters. Read, Read, Read! Try to expand your horizons. Read about stuff that might interest you in a couple of years. If you keep hearing about some new technology or best practice, spend the time to quickly learn about the subject and why it might be important. Reading helps to broaden your vision so you don't get siloed in one tech stack or way of thinking.

## Act
Reading is worthless if it's just knowledge in your head. And I don't care how much you read, you won't fully understand a technology unless you try it out. You'll be able to speak about it, and may know what you're talking about. You may have read so much, you can even talk your way through an interview. But the moment you're asked to follow through on your knowledge, you'll be stuck. Practicing is just as important as reading. It's what builds muscle memory so you know where to start when you're given a task. Start working on your github profile. Write a react app. Write a node app. Write a python app. Start playing with machine learning or building bots or whatever you've decided may interest you. Your github projects don't have to be perfect. I would say screw perfection and shoot for quantity. Expose yourself to as many technologies as you can. Have fun with it. Don't get bogged down in making something beautiful and perfect. Do enough to build a real insight into what it takes to use that 

```
var router = express.Router()
routes.get("/sub*", (req: Request, res: Response) => {
  res.sendFile(path.resolve("index.html"));
});
```

The goal is to replace the route with a subdomain when deployed to production. The way I'll build the routing will avoid the situation where both the subdomain and the route show up in the URL. I don't want `sub.host.test/sub`. I want `sub.host.test` for prod and `localhost/sub` for development.

So far, none of this groundbreaking. Here's where things start to get a bit interesting. Basically, the routes above will handle the local development case perfectly. You could setup a reverse proxy to point to the two routes and build the subdomains into the reverse proxy routes and you might be happy with the result. What you'd get is the duplication of the subdomain in the route like `sub.host.com/sub`. That's not what I want. I want the routes to look nice!

OK, so enter the magic middleware. The middleware identifies the subdomains and routes to the appropriate index page. It's routing within routing. 

```
var subdomainMiddlware = (req: Request, res: Response, next: NextFunction) => {
  if (req.hostname.match(/sub\./g)) {
    res.sendFile(path.resolve("index.html"));
  } else {
    next();
  }
};
```

The middleware works by checking the `hostname` property on the request. If the hostname begins with a valid subdomain, the appropriate html file is sent on the response. Otherwise, the request passes through the middleware.

Here's how the middleware is appplied to the routes. This isn't the most ideal way of adding the middleware, but for this example, it works just fine.

```
routes.get("/sub*", subdomainMiddleware, (req: Request, res: Response) => {
  res.sendFile(path.resolve("index.html"));
});
```

## Setting Up React Router
Now that the server can handle both subdomains and routes to serve up the same pages, React Router needs a little help to build the correct routes.
This is what a typical routing setup looks like for React Router. I've included the `sub` route to reflect the base route. 

```
import { Router, Route, browserHistory } from 'react-router'

render((
  <Router history={browserHistory}>
    <Route path="/sub" component={App} />
  </Router>
), document.getElementById('root'))
``` 

However, I'm interested in changing the base route when the app is running on a production machine or during development. Luckily enough, react-router supports setting a base route using the `createHistory` method from the `history` package. Once setup, the `basename` can be set to change based on the current environment.

```
import { Route, Router, IndexRoute, useRouterHistory } from "react-router";
import { createHistory } from "history";

// Change the base route based on the hostname
const isLocalhost = location.hostname.match(/localhost|127\.0\.0\.1/g);
const history = useRouterHistory(createHistory)({
  basename: isLocalhost ? "/sub" : "/",
});

export default (
  <Router history={history}>
    <Route path="/" component={App}>
      <Route path="/sub" component={App} />
    </Route>
  </Router>
);
```
I've removed the `browserHistory` module from the code above and replaced it with the `useRouterHistory` module which accepts the `createHistory` method as an argument. The whole point is to the change the `basename` if the site is running in localhost or in production. The `basename` value has to match what Express routes on the server side.

By following this pattern, I'm able to publish my site to a remote server and use the subdomain to access the site.
