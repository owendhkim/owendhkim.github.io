---
layout: post
title: Making a search engine | researching and planning
slug: search engine web crawler
comments: true
tags: [search engine, crawler, spider]
---

This summer, I'm given a task to ship a search engine for my university's website.

After doing some research, I was able to learn the following.

<div style="width: 700px; height: 150px; overflow: auto; border: 1px solid transparent; padding: 10px;">

    1. When you search something on the search bar, it is not actually searching through the web and delivering you the result, it is bringing up the relative information from the database full of pre-crawled information.

    2. That being said, searching something on the search bar is like making a query, and web crawler/spider's job is to crawl the web and store the information in a db(preferably indexed, to deliver the result faster). So when someone seraches, query is made to the db and results are brought up.
</div>


To sum it up in few main points...

    1. Crawler crawls the web and stores its contents to db

    2. When search function is used, query is made to db.

    3. Goes through some sort of algorithm to decide what results to display in what order.

Things to consider

---VM spec---

Rocky linux 9.3 x86-64
120GB disk storage 8GB ram

Expecting about 8000 unique and valid links total after a thorough crawl

When crawling, what should I extract and store? Entire html? Particular div? href? title? text content?

Compatibility/integration with front-end

What is more important, execution time or relevance/quality of the result?