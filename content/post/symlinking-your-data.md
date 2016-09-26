---
title: Symlinking Your Data
author: Jason P. Becker
date: 2014-04-02
tags: rstats, symlink, data, encryption, privacy
---

I frequently work with private data. Sometimes, it lives on my personal machine rather than on a database server. Sometimes, even if it lives on a remote database server, it is better that I use locally cached data than query the database each time I want to do analysis on the data set. I have always dealt with this by [creating encrypted disk images](http://support.apple.com/kb/HT1578?viewlocale=en_US&locale=en_US) with secure passwords (stored in [1Password](https://agilebits.com)). This is a nice extra layer of protection for private data served on a laptop, and it adds little complication to my workflow. I just have to remember to mount and unmount the disk images.

However, it can be inconvenient from a project perspective to refer to data in a distant location like `/Volumes/ClientData/Entity/facttable.csv`. In most cases, I would prefer the data "reside" in `data/` or `cache/` "inside" of my project directory.

Luckily, there is a great way that allows me to point to `data/facttable.csv` in my R code without actually having `facttable.csv` reside there: symlinking.

A [symlink](http://en.wikipedia.org/wiki/Symbolic_link) is a **symbolic link** file that sits in the preferred location and references the file path to the actual file. This way, when I refer to `data/facttable.csv` the file system knows to direct all of that activity to the actual file in `/Volumes/ClientData/Entity/facttable.csv`. 

From the command line, a symlink can be generated with a simple command:

```bash
ln -s target_path link_path
```

R offers a function that does the same thing:

```r
file.symlink(target_path, link_path)
```

where `target_path` and `link_path` are both strings surrounded by quotation marks.

One of the first things I do when setting up a new analysis is add common data storage file extensions like `.csv` and `.xls` to my `.gitignore` file so that I do not mistakenly put any data in a remote repository. The second thing I do is set up symlinks to the mount location of the encrypted data.