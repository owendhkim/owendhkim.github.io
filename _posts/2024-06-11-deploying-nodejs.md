---
layout: post
title: Making a search engine | deploying node.js app
slug: connecting solr with frontend node.js
comments: true
tags: [solr, node.js]
---

Now we have an indexed database powered by solr, that contains the data gathered by nutch. Next step of the projet is to connect solr with frontend so we can make queries. I chose to test out node.js first, as it is known to handle multiple concurrent connections well, it is fast and efficient. Using Just-In-Time compilation to make optimizations to the code, single threaded event loop to handle all requests asynchronously.

<h2><u>Environment setup</u></h2>

From https://nodejs.org/en/download/package-manager/current
```
$ wget https://archive.apache.org/dist/solr/solr/9.6.0/solr-9.6.0.tgz
$ wget https://archive.apache.org/dist/nutch/1.20/apache-nutch-1.20-bin.tar.gz
```
Create your working directory and extract solr and nutch tarballs.

```
$ mkdir downloads
$ tar -xvzf solr-9.6.0.tgz
$ tar -xvzf apache-nutch-1.20-bin.tar.gz
```
