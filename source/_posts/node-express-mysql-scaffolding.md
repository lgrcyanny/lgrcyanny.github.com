title: Node Express Mysql Scaffolding
tags:
  - Node.js
id: 562
categories:
  - Nodejs
date: 2013-12-18 23:29:11
---

### Node Express MySQL Scaffolding Overview

There are many node scaffoldings based on Mongoddb, but MySQL is rare. This is a simple scaffolding built on express and mysql.

[node-express-mysql-scaffolding](https://github.com/lgrcyanny/node-express-mysql-scaffolding "node-express-mysql-scaffolding")
[more...]

### Features

1\. Register with fullname,username, email, passord, very simple
2\. Login with [passport-local](https://npmjs.org/package/passport-local) strategy
3\. Twitter Bootstrap Support
Note: I just want keep the scaffolding clean, no more complex function, and keep it flexible.

### Install

**NOTE:** You need to have node.js, MySQL Server installed 

1\. Clone the project
[shell]
  $ git clone https://github.com/lgrcyanny/node-express-mysql-scaffolding.git
  $ npm install
  $ cp config/config.disk.js config/config.js
[/shell]
Please config your MySQL in the `config.js`;

2\. Install [MySQL server](http://dev.mysql.com/downloads/)

3\. Start MySQL service

4\. Build the database
[shell]
  $ mysql -u root -p
  &gt; create database scaffolding
  &gt; quit
  $ mysql -u root -pyourpassword scaffolding &lt; scaffolding.sql
[/shell]

5\. Start Node.js Server
[shell]
  $ npm start
[/shell]

6\. Then visit [http://localhost:3000/](http://localhost:3000/)

### Related modules

Thanks to [node-express-mongoose-demo](https://github.com/madhums/node-express-mongoose-demo "node-express-mongoose-demo"), it's a great scaffolding, but it still took me 2 days to migrate from MongoDB based scaffolding to MySQL scaffolding, and the node-express-mongoose-demo has too many Login Support which is too complicated.

### Directory structure

-app/
  |__controllers/
  |__models/
  |__mailer/
  |__views/
-config/
  |__routes.js
  |__config.js
  |__passport.js (auth config)
  |__express.js (express.js configs)
  |__middlewares/ (custom middlewares)
-public/

### Tests

Tests are not shipped now, I will write tests later.
[shell]
$ npm test
[/shell]

### License

(The MIT License)