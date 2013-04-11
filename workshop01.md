Workshop 1: Intro to Node.js
========

- What is Node.js?
- From Ruby, PHP and Python to Node.js
- How Node Works
	- Callbacks
	- The Asynchronous Model
- Why Use Node.js?
- Setting up your computer for Node development
- First Node.js apps:
	- Hello World
	- Opening a file
	- Creating a web server
- For your Curiosity: What Powers Node.js?

What is Node.js?
--------

Simply explained, Node.js is a programming language for building server-side applications. It is especially good for network I/O applications like web apps, chat servers, real-time games, and so forth.

Is it really a programming language? Only in part. It is server-side JavaScript, but it is also a platform layer with functionaliy to interact with the operating system: to read and write files, do network operations, spawn child processes, etc.

So the way you write Node.js code is just like writing client-side JS, but under the hood is a powerful layer with a lot of high-level OS and network functionality. Get a cursory glance at what Node.js offers on [the Node.js API page](http://api.nodejs.org).


From Ruby, PHP and Python to Node.js
--------

> _Note: because this topic can be quite controversial, it should be emphasized that we are comparing these languages only for the sake of bridging your previous experience to Node.js._

> _Conceptually, Node.js differs greatly from the way these other languages work. So our goal here is to show-by-example, not to raise judgments about the value, language speed or design of one approach over another -- at least, not yet :-)._

Reading a file in PHP:

```php
echo file_get_contents("./README.md");
```

Reading a file in Ruby:

```ruby
File.open("./README.md", "r").each_line do |line|
  puts line
end
```


Reading a file in Python:

```python
print open("./README.md", "r").readlines()
```

Reading a file in Node.js:

```javascript
var fs = require("fs");

fs.readFile("./README.md", "utf-8", function(err, fileContents) {
    console.log(fileContents);
});
```

Many developers initially struggle with the way this is coded and how the file data from `readFile` is returned back to the application. This is because we have been trained to think like a **procedural** programmer. That is, we expect a function call like `readFile` to _return_ the contents of the file.

But what is actually happening here? The `readFile` method takes 3 arguments; the first two are obvious enough, the location of the file and the encoding of the file. But what is that 3rd parameter? That's called a _callback_ and it's what Node.js "calls back" when the file is done being read.


### Callbacks

How many are familiar with this jQuery AJAX code?

```javascript
$.post("/supportrequest", { name : "Bob", request : "I need help!" }, function(res) {
	console.log(res);
});
```

If you are: congratulations, you've seen callbacks before and may not have known about it. What makes this example easier to digest is we expect that a network call will take some time to complete. Only when it is done will that 3rd argument `function(res)` be "called back" with the results from the server.

Node.js takes this concept and applies it to every way you interact with the operating system.


### The Asynchronous Model

Callbacks are part and parcel of the _asynchronous model_ of development. Our procedural bias (that is, expecting data to be _returned_ from a `readFile` operation) also means we expect that the next line of code in our application won't execute until that first line of code is done. In Node.js, this is not the model. And for good reason. Let's look at an illustrative example.

Here is our `readFile` example again, only this time with a console.log statement at the end of the code. Which output will we see first, the console.log or the contents of the file?

```javascript
var fs = require("fs");

fs.readFile("./README.md", "utf-8", function(err, fileContents) {
    console.log(fileContents);
});

console.log("Hello");
```

Remember: the _callback_ passed as the 3rd argument to `readFile` will only be called when the file contents are done loading. So what gets output first? The `"Hello"` from the last line in the file. Subsequently when Node.js is done reading in the file from disk, the file data is output from inside the callback.



OK. Why would we Develop Apps this Way?
-------

Let's imagine an analogy from everyday life. You walk into a restaurant and take a seat. At the next table over is a group of 20 who have been there for a while and already ordered. You order a hamburger. Meanwhile, the host is seating other people, other waiters are taking orders and delivering food to their patrons. In short: this is an asynchronous model of functioning. Generally speaking, no one at the restaurant is "blocking" anyone else from doing work. Case in point? You get your hamburger before the larger party. Your request is smaller and doesn't need to come out at the same time as 19 other people.

Now imagine a synchronous model of running a restaurant. You enter the restaurant behind 5 other people in line. Before you can be seated, the first patron must be seated, their drinks and food delivered, they have to sign the check and then leave. Then the 2nd person in line is seated. How happy would you be under these circumstances?

_Not very happy_. On more technical terms: the traditional model of, say, serving HTTP requests from an Apache server means an entire restaurant is spawned for every patron. How efficient is that? Terribly inefficient! Very quickly you run out of real estate (RAM).

#### The Asynchronous Model is Superior

Although you are running under just one process, you get the benefits of being able to serve many simultaneous requests without the messiness of threads. The potential for a high level of concurrency is much more feasible using this model.

In summary there are three reasons that Node is really compelling:


1. Unlike the Apache web server, which spawns a thread every time a request comes in, Node.js doesn't use threads when serving network requests. This offers some sizable performance advantages, especially as the volume of requests grows.

2. Your Node.js applications won't "block" while retrieving data from a database, doing a file operation, or serving an HTTP request. This means many more requests can be served from one instance of your application.

3. Many developers already know client-side JavaScript. They can use the same mental understanding of the structure and semantics of client-side JS, now on the server.


Setting up your Computer for Node Development
-------

The first step is to download [StrongLoop Node](http://strongloop.com/products). This is the StrongLoop distribution of Node.js containing Node, NPM (the Node Package Manager for userland modules), `slnode` (a swiss-army CLI tool with commands for scaffolding, testing and documenting Node.js source code) and a set of supported NPM modules.

StrongLoop Node is and will always remain free, but if you ever decide to use Node.js in production you can pursue [StrongLoop support plans](http://strongloop.com/products/support).

### Use your Favorite Editor

The great thing about Node.js is most editors already support JavaScript syntax highlighting. Simply create a file, edit it, save it with the ".js" extension and you're good to go.

### Running Node.js Apps

Running a Node.js app is as simple as opening a terminal to the directory where your file is, and running `$ node app.js` or simply `$ node app` (you don't need to include the ".js" extension).

If you want to go into interactive mode, start node like so: `$ node` and you will get the Node.js REPL.

```sh
$ node
> var x = 3;
undefined
> x * 4;
12
```

Your First Node.js Apps
-------

In all these examples we simply take the code and run it with `$node [filename]` in our terminal.

### Hello World

Note here how we get the same `setTimeout` function as client-side JavaScript. This piece of code first prints "Hello, World" and then 2000ms later prints "Hello, Callback".

```javascript

setTimeout(function() {
	console.log("Hello, Callback");
}, 2000);

console.log("Hello, World");
```

### Opening a File

Try running this without the "README.md" file first to see the error. Then create the "README.md" file in the same directory as your application code to see its contents.

```javascript
var fs = require("fs");

fs.readFile("./README.md", "utf-8", function(err, fileContents) {
	if (err) {
		console.log("Error reading file", err);
		return;
	}

    console.log(fileContents);
});
```

### Simple Web Server

In just 6 lines of code here we can use Node.js' built-in HTTP module for creating an HTTP server. The first argument to `createServer` is the function that is called every time a new request comes to our server.

The variables passed to our callback are `req` and `res` -- that is, the _request_ object, which gives us details about the incoming request from the web browser, and the _response_ object, which provides methods for sending data back to the browser.

Once the server is running you can access it by putting "127.0.0.1:1337" into your browser. Unlike the previous two examples, this application will continue to run indefinitely until you manually kill it.

```javascript
var http = require("http");

http.createServer(function (req, res) {
  res.writeHead(200, { "Content-Type" : "text/plain" });
  res.end("Hello World\n");
}).listen(1337, "127.0.0.1");

console.log("Server running at http://127.0.0.1:1337/");
```

For your Curiosity: What Powers Node.js?
-------

Node can be broken down into three core technologies:

* The foundation of Node is [Chrome's JavaScript runtime](http://code.google.com/p/v8/), known as the "V8 JavaScript Engine" or just "V8". V8 is the same engine used by Google's Chrome Browser to execute JavaScript on any page you visit while browsing, and it's what allows Node developers to use JavaScript at all in the first place.

* The next major component of Node is the [CommonJS](http://www.commonjs.org/) module system - it's a standard which allows developers to define modules that can be reused and shared throughout their own applications and others. Without CommonJS' standards we wouldn't have systems like [npm](http://www.npmjs.org/) (Node Package Manager) which allow Node developers to share reusable packages with each other and provide a standard for handling dependencies in our Node applications.

* The final major component of Node is a powerful underlying layer called [libuv](https://github.com/joyent/libuv) - this is what abstracts away all of the underlying network/file system/etc functionality on both Windows and POSIX-based systems like Linux and OS X.

