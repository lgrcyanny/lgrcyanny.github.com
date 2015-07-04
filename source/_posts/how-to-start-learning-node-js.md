title: How to start learning Node.js
tags:
  - Learning
  - nodejs
id: 130
categories:
  - Nodejs
date: 2013-11-10 20:36:45
---

I have always wanted to do some meaningful projects, not the homework project at college, not the debug working at company. I just wanted to start a meaningful project for interest, or for fun. Maybe it will become a big success. and I believe in that. And recently I am busy with a spark prject based on Node.js.

Node.js is a platform built on Chrome's JavaScript V8 engine. The event-driven, non-blocking I/O model makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.

In short, I think the advantages is:[more...]

*   Same language on both server side and front side
*   Event-driven and callback feature gains perfect performance
*   With WebSocket, you can built great real-time app, like online chat room
*   Worth to learn, it's new, and will give you a new perspective of web development

## Beginner Guides

[1\. Felix's Node.js Beginners Guide](http://nodeguide.com/beginner.html)

This a short blog that can help you overview Node.js, learn some basics of Node.js, you can finish it within 30 min.
The Module system of node is worth to dig, read the [api](http://nodejs.org/api/modules.html), especially the concept of **exports** and the **module.exports**

[2\. Node style guide](http://nodeguide.com/style.html#class-names)

Before programming, learn the programming style of node.

[NPM coding style](https://npmjs.org/doc/coding-style.html)
This is the npm coding style. At the beginning, indentation with 2 spaces make me uneasy. But now I am used to it.

[3\. The Node Beginning Book](http://www.nodebeginner.org/)

The beginning book is really great, Even though it has more than 70 pages, I finished it and learned a lot. I followed the example in the book, build a simple blog demo. And the code in the book had some tiny issue. I have uploaded the demo to my github, [node-simle-blog-demo](https://github.com/lgrcyanny/nodedemo).
BTW, the book will teach you how to run linux command by node [Child Processes](http://nodejs.org/api/child_process.html), sounds interesting.

## Learning more details

[4\. Express guide](http://expressjs.com/guide.html)

After learning how to build an application stack from the beginner book, you can start express now, finish the guide, understand the basics of the great web framework express. BTW, in node community, there are large amount of web framework. Like Tower.js, meteor.js, derby and so on. But express is the most popular and is supported by many people. I recommend it.

[5\. Node.js in Action](http://www.manning.com/cantelon/)

The book recommended by express guide has more than 400 pages. Don't withdraw, insist on it, you can follow the book example, and learn more details about node. I haven't finish it, because I want to start my project more quickly, I just walked to Chapter4.
The **callback** model and **event driven** explained in the book is great. Especially the muti-chat room by socket.io gave me a deep impression.

[6\. Build a express mongo blog](http://howtonode.org/express-mongodb)

Follow the blog and build a simple blog with mongodb+express+jade, the code of jade for the blog has a tiny issue [here](http://www.devthought.com/code/use-jade-blocks-not-layouts/), my demo is here [express blog](https://github.com/lgrcyanny/expressblog)

7\. Now I am fed up with tutorials, I decided to build something by myself. I build a file uploader/downloader with express. [FileManager](https://github.com/lgrcyanny/ExpressFileManager)

## Build the Web project

[8\. Node Express mongoose demo](https://github.com/madhums/nodejs-express-mongoose-demo)

I build PartyTime based on the demo. The demo is a blog with passport and authentication function. Worth to learn

[9\. Mongoose](http://mongoosejs.com/)

Although mongodb provide nodejs api, I think the middleware is more better than the origin mongodb API

[10\. Mongodb Manual](http://docs.mongodb.org/manual/)

Lean basics about mongodb, especially the Data Modeling method.

## Others

[11\. Node github](https://github.com/joyent/node)

[12\. A guide for api usage](http://docs.nodejitsu.com/)

[13\. Node Cloud](http://www.nodecloud.org/)
This is a resource directory gathering sites related to Node.js.

[14\. NPM](https://npmjs.org/)  Finding and uploading npm package.

15\. Some api worth to read: Modules, Child Processes, Crypto, File System, HTTP, Events, Path, QueryString, URL.

## Good Luck!

Many tutorials, no matter how to lean, enjoy the learning process and the final sucess! Don't waiting for finishing every details before starting your project.