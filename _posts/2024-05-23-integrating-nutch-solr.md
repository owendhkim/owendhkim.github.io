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

From https://solr.apache.org/downloads and https://nutch.apache.org/download/ get the links for the desired version.
```
$ wget https://www.apache.org/dyn/closer.lua/solr/solr/9.6.0/solr-9.6.0.tgz
$ wget https://www.apache.org/dyn/closer.lua/nutch/1.20/apache-nutch-1.20-bin.zip
```
Extract the tarball/zip file, cd into the folder and copy pwd.

```
[cvm_owenk@search-dev apache-nutch-1.20]$ ls
bin   crawl  lib             licenses-binary  logs           NOTICE.txt  urls
conf  docs   LICENSE-binary  LICENSE.txt      NOTICE-binary  plugins
[cvm_owenk@search-dev apache-nutch-1.20]$ pwd
/home/cvm_owenk/downloads/apache-nutch-1.20
```
Export PATH variable to your shell configuration by editing .bashrc or .zshrc and set what you copied to be $NUTCH_HOME. HOME directory of a program should be a directory one upper level from bin directory. Do the same for solr.
```
export NUTCH_HOME="/home/cvm_owenk/downloads/apache-nutch-1.20"
export PATH=$NUTCH_HOME/bin:$PATH
```
After downloading and exporting PATH variable. Try running nutch and solr to verify it was done correctly.
```
[cvm_owenk@search-dev ~]$ solr

Usage: solr COMMAND OPTIONS
       where COMMAND is one of: start, stop, restart, status, healthcheck, create, create_core, create_collection, delete, version, zk, auth, assert, config,
...

[cvm_owenk@search-dev ~]$ nutch
nutch 1.20
Usage: nutch COMMAND [-Dproperty=value]... [command-specific args]...
where COMMAND is one of:
  readdb            read / dump crawl db
  mergedb           merge crawldb-s, with optional filtering
...
```
## Creating Solr core for nutch
For faster query Solr will index the crawld results into a db. If you are planning to shard the db you may want to look into Solr collection. Otherwise, creating a core will be sufficient.

Create path for nutch core, copy default config into it.

```
$ mkdir -p ${SOLR_HOME}/server/solr/configsets/nutch/
cp -r ${SOLR_HOME}/server/solr/configsets/_default/* ${SOLR_HOME}/server/solr/configsets/nutch/
```

Check if schema.xml file exists in this path .../src/plugin/indexer-solr/schema.xml<br>
If it exists copy it to ${SOLR_HOME}/server/solr/configsets/nutch/conf/<br>
If not, download one from solr github repo.
```
$ curl -o schema.xml https://raw.githubusercontent.com/apache/nutch/release-1.16/src/plugin/indexer-solr/schema.xml
```

Delete managed-schema file if it exists
```
$ rm ${SOLR_HOME}/server/solr/configsets/nutch/conf/managed-schema
```
Create nutch core
```
${SOLR_HOME}/bin/solr create -c nutch -d ${SOLR_HOME}/server/solr/configsets/nutch/conf/
```
Start solr and go on to the admin page (http://localhost:8983/solr/) to verify that solr gui is running and nutch core is created

```
$ solr restart
```
If you are running solr on vm like me, you might have to look into ssh tunneling https://www.ssh.com/academy/ssh/tunneling-example<br>
I had to use the command below to forward server's localhost:8983 to client's(me) localhost:8983.
```
$ ssh -L <local URL>:<local Port>:<remote URL>:<remote Port> <username>@<hostname>
```
## Before crawling
We are almost ready to crawl, before crawling we must add the agent name to our .xml file and create a seeds list.

In $NUTCH_HOME/conf directory there should be a nutch-site.xml file, properties added in this file will overwrite the nutch-default.xml file which contains the default setting.<br>
You are requried to add the http.agent.name value to your nutch-stie.xml file in order to crawl using nutch.

```
<configuration>

<property>
  <name>http.agent.name</name>
  <value>Your agent name here</value>
  <description>'User-Agent' name: a single word uniquely identifying your crawler.

  The value is used to select the group of robots.txt rules addressing your
  crawler. It is also sent as part of the HTTP 'User-Agent' request header.

  This property MUST NOT be empty -
  please set this to a single word uniquely related to your organization.

  Following RFC 9309 the 'User-Agent' name (aka. 'product token')
  &quot;MUST contain only uppercase and lowercase letters ('a-z' and
  'A-Z'), underscores ('_'), and hyphens ('-').&quot;

  NOTE: You should also check other related properties:

    http.robots.agents
    http.agent.description
    http.agent.url
    http.agent.email
    http.agent.version

  and set their values appropriately.
  </description>
</property>

...

</configuration>
```
To create seeds list, cd into $NUTCH_HOME and create a .txt file inside a urls directory, make sure it is formatted to one url per line.
```
$ cd $NUTCH_HOME
$ mkdir urls
$ cd urls
$ touch seed.txt
```
## Crwaling
