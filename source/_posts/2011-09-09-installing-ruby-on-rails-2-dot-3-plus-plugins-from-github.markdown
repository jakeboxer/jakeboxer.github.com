---
layout: post
title: "Installing Ruby on Rails 2.3+ plugins From Github"
date: 2010-01-13 20:21
comments: true
categories: 
---

I've been banging my head against this wall for quite awhile now, and I just finally figured out the answer. Like I've done in other posts, I'll just post what worked for me, and hopefully it'll help other people.

I'm running Ruby 1.9 and Ruby on Rails 2.3.3 on Snow Leopard. I've been trying to install plugins (specifically, [Authlogic](http://github.com/binarylogic/authlogic) and [Carmen](http://github.com/jim/carmen)) for a couple days now using the following two commands (as taken from the main github pages):

``` bash
script/plugin install git://github.com/binarylogic/authlogic.git
script/plugin install git://github.com/jim/carmen.git
```

In return, I received the following errors:

``` bash
Plugin not found: ["git://github.com/binarylogic/authlogic.git"]
Plugin not found: ["git://github.com/jim/carmen.git"]
```

After a lot of poking around, it turns out you need to make two changes in order for this to work on Rails 2.3 or higher: change the `git://` at the beginning of each URL to `http://`, and add a trailing slash to the end of each URL. So instead, run these:

``` bash
script/plugin install http://github.com/binarylogic/authlogic.git/
script/plugin install http://github.com/jim/carmen.git/
```

They both worked perfectly for me, so hopefully they'll work for you. If not, leave a comment and I'll try to help.