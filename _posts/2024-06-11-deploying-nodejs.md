---
layout: post
title: Making a search engine | deploying node.js app
slug: connecting solr with frontend node.js
comments: true
tags: [solr, node.js]
---

Now we have an indexed database powered by solr, that contains the data gathered by nutch. Next step of the projet is to connect solr with frontend so we can make queries. I chose to test out node.js first, as it is known to handle multiple concurrent connections well, it is fast and efficient. Using Just-In-Time compilation to make optimizations to the code, single threaded event loop to handle all requests asynchronously.

<h2><u>Environment setup</u></h2>

From `https://nodejs.org/en/download/package-manager/current` select desired version and os, run the commands in command line.
```
# installs nvm (Node Version Manager)
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# download and install Node.js (you may need to restart the terminal)
$ nvm install 22

# verifies the right Node.js version is in the environment
$ node -v # should print `v22.3.0`

# verifies the right NPM version is in the environment
$ npm -v # should print `10.8.1`
```
Npm is node.js package manager and it is included in node.js installation. I will be using solr-node package to connect establish connection between solr and node.js app, in order to make queries. Create and cd into a working directory, install solr-node, as well as express and log4js.
```
$ mkdir site
$ cd site
$ npm install solr-node
$ npm install express
$ npm install log4js
```
Create a server.js file