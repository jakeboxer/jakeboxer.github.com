---
layout: post
title: "Convert an int array to a comma-separated string with C"
date: 2009-11-16 19:55
comments: true
categories: 
---

I'm much less experienced in C than I am in higher-level languages. At Cisco, we use C, and I sometimes run into something that would be easy to do in Java or Python, but very difficult to do in C. Now is one of those times.

I have a dynamically-allocated array of unsigned integers which I need to convert to a comma-separated string for logging. While the integers are not likely to be very large, they could conceptually be anywhere from 0 to 4,294,967,295. In Python, that's one short line.

``` python
my_str = ','.join([str(num) for num in my_list])
```

How elegantly can people do this in C? I came up with a way, but it's gross. If anyone knows a nice way to do it, please enlighten me.