---
layout: post
title: Making a search engine | integrating nutch and solr
slug: web crawler integrating nutch and solr
comments: true
tags: [nutch, solr, crawler, spider]
---

I'm choosing to test nutch and solr first as they are both from the same company, Apache, built for each other to be paired. After testing out the crawler, I needed to look for some sort of management system in between crawler and a db so I can connect to front end and make queries anyway, it is great that there is an option well compatible with each other.

Nutch is great for crawling in large scale, and has built in indexer that can be pipelined into solr. Both open source.

<h2><u>Environment setup</u></h2>

From `https://solr.apache.org/downloads` and `https://nutch.apache.org/download` get the download mirror links for the desired version.
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

cd into the folder and copy pwd.

```
[cvm_owenk@search-dev apache-nutch-1.20]$ ls
bin   crawl  lib             licenses-binary  logs           NOTICE.txt  urls
conf  docs   LICENSE-binary  LICENSE.txt      NOTICE-binary  plugins
[cvm_owenk@search-dev apache-nutch-1.20]$ pwd
/home/cvm_owenk/downloads/apache-nutch-1.20
```
Export PATH variable to your shell configuration by editing `.bashrc` or `.zshrc` and set what you copied to be `$NUTCH_HOME`. HOME directory of a program should be a directory one upper level from bin directory. Do the same for solr, and reload `.bashrc`.
```
$ vi ~/.bashrc
```
```
export NUTCH_HOME="/home/cvm_owenk/downloads/apache-nutch-1.20"
export PATH=$NUTCH_HOME/bin:$PATH
```
```
$ source ~/.bash_profile
```
After downloading and exporting PATH variable. Try running nutch and solr to verify it was done correctly, you should see help options showing like below.
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
If you get `JAVA_HOME not set` message, run the command `$ which java` to grab the location where java is installed and export it as variable JAVA_HOME on your shell profile, just like other HOME variables, it should be one upper directory from the /bin directory.

```
[cvm_owenk@search-dev2 apache-nutch-1.20]$ which java
/usr/bin/java

export JAVA_HOME="/usr"
export PATH=$JAVA_HOME/bin:$PATH
```

<h2><u>Creating Solr core for Nutch</u></h2>

For faster query Solr will index the crawld results into a db. If you are planning to shard the db you may want to look into Solr collection. Otherwise, creating a core will be sufficient.

Create path for nutch core, copy default config into it.

```
$ mkdir -p ${SOLR_HOME}/server/solr/configsets/nutch/
$ cp -r ${SOLR_HOME}/server/solr/configsets/_default/* ${SOLR_HOME}/server/solr/configsets/nutch/
```

Check if schema.xml file exists in this path `.../src/plugin/indexer-solr/schema.xml`<br>
If it exists copy it to `${SOLR_HOME}/server/solr/configsets/nutch/conf/`<br>
If not, download one from solr github repo.
```
$ curl -o schema.xml https://raw.githubusercontent.com/apache/nutch/release-1.20/src/plugin/indexer-solr/schema.xml
```

Delete managed-schema file if it exists
```
$ rm ${SOLR_HOME}/server/solr/configsets/nutch/conf/managed-schema
```
Start solr and create nutch core
```
$ solr start
$ ${SOLR_HOME}/bin/solr create -c nutch -d ${SOLR_HOME}/server/solr/configsets/nutch/conf/
```
Restart solr and go on to the admin page (http://localhost:8983/solr/) to verify that solr gui is running and nutch core is created

```
$ solr restart
```
If you are running solr on vm like me, you might have to look into ssh tunneling https://www.ssh.com/academy/ssh/tunneling-example<br>
I had to use the command below to forward server's localhost:8983 to client's(me) localhost:8983.
```
$ ssh -L <local URL>:<local Port>:<remote URL>:<remote Port> <username>@<hostname>
```
<h2><u>Before crawling</u></h2>
We are almost ready to crawl, before crawling we must add the agent name to our .xml file and create a seeds list.

In `$NUTCH_HOME/conf` directory there should be a `nutch-site.xml` file, properties added in this file will overwrite the `nutch-default.xml` file which contains the default setting.<br>
You are requried to add the `http.agent.name` value to your `nutch-stie.xml` file in order to crawl using nutch.

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
To create seeds list, cd into `$NUTCH_HOME` and create a .txt file inside a urls directory, make sure it is formatted to one url per line.
```
$ cd $NUTCH_HOME
$ mkdir urls
$ cd urls
$ touch seed.txt
```
<h2><u>Crawling</u></h2>

There are two binary executables inside the bin directory, you can use bin/nutch to crawl step by step or you can use bin/crawl to set up an automative crawl.

#### Crawling step by step:
From NUTCH_HOME directory
1. inject seeds `$ bin/nutch inject crawl/crawldb urls`
2. generate fetch list `$ bin/nutch generate crawl/crawldb crawl/segments`
3. set target segment directory ``$ s1=`ls -d crawl/segments/2* | tail -1` ``
4. fetch `$ bin/nutch fetch $s1`
5. parse fetched data `$ bin/nutch parse $s1`
6. update db(local db in nutch home) `$ bin/nutch updatedb crawl/crawldb $s1`
7. invert links in order to index `$ bin/nutch invertlinks crawl/linkdb -dir crawl/segments`
8. index `$ bin/nutch index crawl/crawldb/ -linkdb crawl/linkdb/ -dir crawl/segments -filter -normalize -deleteGone`

<br>

#### Using crawl script:
```
Usage: crawl [options] <crawl_dir> <num_rounds>

Arguments:
  <crawl_dir>                           Directory where the crawl/host/link/segments dirs are saved
  <num_rounds>                          The number of rounds to run this crawl for

Options:
  -i|--index                            Indexes crawl results into a configured indexer
  -D                                    A Nutch or Hadoop property to pass to Nutch calls overwriting
                                        properties defined in configuration files, e.g.
                                        increase content limit to 2MB:
                                          -D http.content.limit=2097152
                                        (distributed mode only) configure memory of map and reduce tasks:
                                          -D mapreduce.map.memory.mb=4608    -D mapreduce.map.java.opts=-Xmx4096m
                                          -D mapreduce.reduce.memory.mb=4608 -D mapreduce.reduce.java.opts=-Xmx4096m
  -w|--wait <NUMBER[SUFFIX]>            Time to wait before generating a new segment when no URLs
                                        are scheduled for fetching. Suffix can be: s for second,
                                        m for minute, h for hour and d for day. If no suffix is
                                        specified second is used by default. [default: -1]
  -s <seed_dir>                         Path to seeds file(s)
  -sm <sitemap_dir>                     Path to sitemap URL file(s)
  --hostdbupdate                        Boolean flag showing if we either update or not update hostdb for each round
  --hostdbgenerate                      Boolean flag showing if we use hostdb in generate or not
  --num-fetchers <num_fetchers>         Number of tasks used for fetching (fetcher map tasks) [default: 1]
                                        Note: This can only be set when running in distributed mode and
                                              should correspond to the number of worker nodes in the cluster.
  --num-tasks <num_tasks>               Number of reducer tasks [default: 2]
  --size-fetchlist <size_fetchlist>     Number of URLs to fetch in one iteration [default: 50000]
  --time-limit-fetch <time_limit_fetch> Number of minutes allocated to the fetching [default: 180]
  --num-threads <num_threads>           Number of threads for fetching / sitemap processing [default: 50]
  --sitemaps-from-hostdb <frequency>    Whether and how often to process sitemaps based on HostDB.
                                        Supported values are:
                                          - never [default]
                                          - always (processing takes place in every iteration)
                                          - once (processing only takes place in the first iteration)
```
In my case the command looked like this, -i and -D swtich to turn on auto indexing and its destination, 10 rounds of crawl.
```
bin/crawl -i -D solr.server.url=http://localhost:8983/solr crawl/ 10
```
