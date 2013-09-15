---
layout: post
title:  "Beginning Node.js"
date:   2013-05-25 14:33:39
tags:   coding javascript node.js
---

> Quintessential blog application using Node.js and Express  

## JavaScript on Server

Node.js takes JavaScript from browser to the server. Node platform is built on Chrome’s V8 engine which makes it pretty fast.It uses an event-driven, non-blocking IO model which is disruptively different from regular web development paradigm.

## Installing Node

Node.js packages are available at the official site. Once Node is installed, open terminal and run node —version or node —help to verify that everything is working fine.

## Writing your first Node.js program

Node provides several modules which can be included in your programs. Following is the “Hello World” equivalent in node world:

```javascript
var http = require("http");
var server = function(req, res) {
	res.writeHead(200, {"Content-Type": "text/plain"});
	res.write("Hello World");
	res.end();
}
http.createServer(server).listen(8888);
```

Save this file as, say, helloworld.js and then from terminal run this program as node helloworld.js. Congratulations, you just wrote your first web server.

## Enter Express

Express is a web application framework inspired by Sinatra. Node has this concept of packages or modules similar to ruby’s gems. To get started with express, define a package for your app which is essentially a package.json file in the app directory. Think of gemfile. Define express as one of the dependencies for your app:

```javascript
{
	"name": "express",
	"description": "express blog",
	"version": "0.0.1",
	"private": true,
	"dependencies": {
		"express": "3.2.5"
	}
}
```

To check the latest version of express, use `npm info express version`.  
Now that we’ve defined the package, we can use NPM (Node Package Manager) to install the dependencies, a la Bundler. Just run

`npm install`

Once npm finishes, all dependencies are available in `./node_modules` directory. (Since express is itself a package, it has its own package.json and all dependencies rolled into node_modules folder.)

Ok, let’s start using express. Require express and define an application instance:

```javascript
var express = require('express');
var app = express();
```

Now you can use app.VERB() to define a route. Also, you can use app.listen() to start the server.

```javascript
app.get('/', index);
app.listen(8888);
```

Using our earlier code, here’s our first complete express app.

```javascript
var express = require('express');
var app = express();

var index = function(req, res) {
	res.writeHead(200, {"Content-Type": "text/plain"});
	res.write("Hello World");
	res.end();
}

app.get('/', index);
app.listen(8888);
```

Express augments node objects to add some extra syntactic sugar. For example, above code can be made more succinct by using res.send():

```javascript
var express = require('express');
var app = express();

var index = function(req, res) {
	res.send("Hello World");
}

app.get('/', index);
app.listen(8888);
```
  
##Resources

[Node.js Homepage](http://nodejs.org/)  
[The Node Beginner Book](http://www.nodebeginner.org/)  
