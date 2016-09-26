---
title: A PostgreSQL Cheat Sheet for OSX and R
author: Jason P. Becker
date: 2015-01-23
tags: rstats, posgresql, osx 
---

I keep this on my desktop.


**Install:**

```bash
brew install postgresql
initdb /usr/local/var/postgres -E utf8
gem install lunchy
### Start postgres with lunchy
mkdir -p ~/Library/LaunchAgents
cp /usr/local/Cellar/postgresql/9.3.3/homebrew.mxcl.postgresql.plist ~/Library/LaunchAgents/
```

**Setup DB from SQL file:**

```bash
### Setup DB
lunchy start postgres
created $DBNAME
psql -d $DBNAME -f '/path/to/file.sql'
lunchy stop postgres
```
**Starting and Stopping PostgreSQL**

```bash
lunchy start postgres
lunchy stop postgres
```
may run into trouble with local socket… try this: 

```
rm /usr/local/var/postgres/postmaster.pid
```

**Connecting with R**

```r
# make sure lunch start postgres in terminal first)
require(dplyr)
db <- src_postgres(dbname=$DBNAME)
```

Inspired by seeing [this post](http://altons.github.io/r/2015/01/22/an-easy-way-of-installing-rpostgresql-on-mac/) and thought I should toss out what I do.
