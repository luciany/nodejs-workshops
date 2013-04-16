Workshop 2: Building a Website in Node.js
========

- Recap of our basic web server from [workshop 1](https://github.com/strongloop/nodejs-workshops/blob/master/workshop01.md)
- Creating a more advanced server with express
 - Internals
 - Middleware
- Adding page routes
 - The `render` method
- Setting up static assets
- 404 page
- Authentication

What this Workshop Covers
--------

In [workshop 1](https://github.com/strongloop/nodejs-workshops/blob/master/workshop01.md) we learned about how Node.js works, installed the [StrongLoop Node distribution](http://strongloop.com/products) and built some starter apps.

One of those applications was a simple web server based on the core `http` library:

```javascript
var http = require("http");

http.createServer(function (req, res) {
  res.writeHead(200, { "Content-Type" : "text/plain" });
  res.end("Hello World\n");
}).listen(1337, "127.0.0.1");

console.log("Server running at http://127.0.0.1:1337/");
```

Now we’ll expand our sphere of knowledge to include `express` which builds on the `http` library to provide all of the functionality you will likely need in a web server.


Creating a more advanced server with express
-------

The `slnode` command-line tool ships standard with [StrongLoop Node](http://strongloop.com/products). We can use it to:

- Initialize a new StrongLoop Node project, and create boilerplate code for servers, modules and CLI tools
- Run a specified script
- Print node environment information
- Run tests

We are going to use the command `slnode create` to scaffold a web server, which uses the [express web server](http://expressjs.com/) at its core. Express is one of the most popular web frameworks for Node.js. It handles all the details of serving content, rendering templates (e.g. Jade, EJS) and route handling.

#### Using `slnode create`

`slnode` contains a few subcommands that make it really convenient to create a few different types of Node.js applications. Entering `slnode create` presents the following options:

- cli - creates an empty cli program
- module - creates an empty module in the current application
- package - creates a full node module package
- web - a simple express app with optional mongoose support

We are going to create a web app, so go ahead and type `slnode create web [name]` where [name] is the name of the folder you want to create.

![slnode create](./workshop02/slnode-create.png)

#### Running the Server

To run your freshly minted web app:

```bash
$ cd serverapp
$ slnode install
... bunch of output
$ slnode app
```

Running `slnode install` will result in a lot of output. What this is doing is using NPM to install all of our project's library dependencies &mdash; namely, express and a templating language called EJS.

This is what you should see in your terminal when running `slnode app` and then accessing the server at [http://localhost:3000](http://localhost:3000):

![running the web server](./workshop02/running the server.png)

The Internals
-------

Running `slnode create web serverapp` has scaffolded our folders and files like so:

```
+ node_modules
+ public
+ routes
+ views
  app.js
  package.json
```

The application entry point is `app.js`. Let's take a look inside and break it down piece by piece.

```javascript
var express = require('express')
  , routes = require('./routes')
  , http = require('http')
  , path = require('path');

var app = express();
```

After we load in all our dependencies we create the application with `var app = express();`

```javascript
app.configure(function(){
  app.set('port', process.env.PORT || 3000);
  app.set('views', __dirname + '/views');
  app.set('view engine', 'ejs');
  app.use(express.favicon());
  app.use(express.logger('dev'));
  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(app.router);
  app.use(express.static(path.join(__dirname, 'public')));
});
```

Here we are configuring express. Our configuration options are set with `app.set` &mdash; you can see how this works and the different options on the [express app.configure api page](http://expressjs.com/api.html#app.configure).

`app.use` is the interface for setting up _middleware_, an important concept in the express ecosystem.

Express Middleware
-------

Generally if you use `slnode create` to set up your server then you shouldn't have to touch the middleware. But it's a good idea to know how it works.

The middleware in our application was set in `app.configure` and consists of these functionality:

```
app.use(express.favicon());
app.use(express.logger('dev'));
app.use(express.bodyParser());
app.use(express.methodOverride());
app.use(app.router);
app.use(express.static(path.join(__dirname, 'public')));
```

Each piece of middleware is passed the incoming request from your website visitors, and then either does something with those requests or passes them on to the next piece of middleware.

#### The logger middleware

For instance, the line `app.use(express.logger('dev'));` uses the logging facility provided by express. Remember the output in your terminal after accessing your server for the first time? This is the logger in action.

![running the web server](./workshop02/running the server.png)

#### Ordering middleware

Since middleware can determine whether a request is passed on to the next piece of middleware or not, the order by which you set up your middleware is paramount. For example, these are our first two pieces of middleware:

```javascript
app.use(express.favicon());
app.use(express.logger('dev'));
```

You may have noticed in our logger output we did not see a request for the favicon. This is because the favicon middleware stops the request from reaching the next piece of middleware, the logger. But what happens if we reverse the order of the middleware setup?

```javascript
app.use(express.logger('dev'));
app.use(express.favicon());
```

Look at the logger output again:

```bash
GET / 200 1ms - 212b
GET /stylesheets/style.css 200 1ms - 110b
GET /favicon.ico 200 3ms
```

Yep, the `favicon` middleware was stopping requests from going on to the next piece of middleware, so the logger was never seeing the favicon request (generally this is desirable since favicon requests are not particularly interesting). But once we put the logger at the top of the middleware order, we saw the favicon request the browser was making.

Here is the rest of app.js:

```javascript
app.configure('development', function(){
  app.use(express.errorHandler());
});

var options = {};

routes(app, options);

http.createServer(app).listen(app.get('port'), function(){
  console.log("serverapp listening on port " + app.get('port'));
});
```

You can see here when we actually create the server we are retrieving the 'port' configuration from express with `app.get('port')`. That was set earlier with the line `app.set('port', process.env.PORT || 3000);`


Adding Page Routes
-------

If our website is going to have multiple pages, we need to add page routes. So when the client accesses:

- ourwebsite.com**/products**
- ourwebsite.com**/support**

then the appropriate page ("products" or "support") is retrieved, rendered and returned to the client.

Our routing code exists in `routes/index.js`. But where does that file get loaded in? Here again is `app.js`:

```javascript
var express = require('express')
  , routes = require('./routes')

// later we pass our app and some blank options to the function
// returned from routes.js --
routes(app, options);
```

Refer to our previous discussion of `module.exports` from workshop 1 if you need a refresher.

Let's dive into `routes/index.js`.

```javascript
/*
 * GET home page.
 */
 
function index(req, res){
  res.render('index', { title: 'serverapp' });
};

/**
 * Set up routes
 */
 
module.exports = function(app, options) {
  app.get('/', index);  
}
```

The important starting point of this file is `app.get('/', index);` down at the bottom.

Here we are registering a route by saying "When the client asks for our homepage (i.e. "/"), render the `index` page and return it to them."

The structure of this file can be a bit confusing, though, so let's condense it down a little more. The file can effectively be rewritten like so:

```javascript
module.exports = function(app, options) {
  app.get('/', function(req, res) {
    res.render('index', { title: 'serverapp' });
  };
}
```

When the client asks for the homepage, express calls our callback function with the `req` (request) and `res` (response) variables.


#### The `render` method

Express gives us a function called `render` on the `res` variable. The first two arguments of `render` are the name of the template you want to render and any variables you want to pass to the template.

`res.render('index', { title: 'serverapp' });`

How does express know where the 'index' template is? In `app.js` when we were configuring express, we told it where to look!

```javascript
app.set('views', __dirname + '/views');
app.set('view engine', 'ejs');
```

We also told express that our rendering engine of choice is `ejs`. So in our views folder we should see a file `index.ejs` right?

Sure enough:

```
+ node_modules
+ public
+ routes
- views
    index.ejs
  app.js
  package.json
```