---
layout: post
title: Making a search engine | integrating nutch and solr
slug: web crawler integrating nutch and solr
comments: true
tags: [nutch, solr, crawler, spider]
---

I'm choosing to test nutch and solr first as they are both from the same company, Apache, built for each other to be paired. After testing out the crawler, I needed to look for some sort of management system in between crawler and a db so I can connect to front end and make queries anyway, it is great that there is an option well compatible with each other.

Nutch is great for crawling in large scale, and has built in indexer that can be pipelined into solr. Both open source.

## Environment setup

```
$ wget https://www.apache.org/dyn/closer.lua/solr/solr/9.6.0/solr-9.6.0.tgz
$ wget https://www.apache.org/dyn/closer.lua/nutch/1.20/apache-nutch-1.20-bin.zip
```
