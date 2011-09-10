---
layout: post
title: "Installing Erlang on Snow Leopard"
date: 2010-01-01 20:14
comments: true
categories: 
---

Here's another in my series of "Installing X on Snow Leopard". These aren't official, well-tested guides; they're just documentations of my attempts to compile and install various things on my personal computer. My last one ([Installing MySQL on Snow Leopard](http://jboxer.com/2009/09/installing-mysql-on-snow-leopard/)) is my most popular post to date (aside from a couple that have been on Reddit). Erlang is less popular than MySQL, but hopefully this will still help a few people.

Downloading and unpacking
-------------------------
Go to http://erlang.org/download.html and download the Source for the newest version (when I was writing this, that was **[R13B03](http://erlang.org/download/otp_src_R13B03.tar.gz)**. After downloading, extract it to somewhere that's convenient to get to with the Terminal.

Configure
---------
Open the Terminal and `cd` into the directory you extracted Erlang to (mine was `/Users/jake/src/otp_src_R13B03`. Then run the following command:

```
./configure \
    --prefix=/usr/local/ \
    --enable-smp-support \
    --enable-threads \
    --enable-darwin-64bit
```

**Note:** You will probably get **three errors**. Read about them in the Configuration Errors section coming up.

The first three configure options are the defaults according to the README. However, I've had experiences where supposed defaults aren't really the defaults when compiled on OS X, so I don't like to take chances. `--enable-darwin-64bit` enables experimental support for the 64bit x86 Darwin binaries. This may not be necessary, but in general, 64-bit stuff has fewer problems on Snow Leopard, so I figured this was a good idea.

Configuration Errors
--------------------

I got the following configuration errors:

```
jinterface    : No Java compiler found
wx            : Can not combine 64bits erlang with wxWidgets on
                MacOSX, wx will not be useable
documentation : fop is missing. The documentation can not be built.
```

These aren't a problem. If you get any errors besides these, you're in trouble. Leave a comment, and I'll see if I can help.

Making and installing
---------------------

These two commands shouldn't give you any trouble:

`make`

And then, after `make` is done:

`sudo make install`

If you get any errors at either of these stages, leave a comment and I'll try to help.

Making sure it works
--------------------

**Note:** This canonical test is gratefully borrowed from [erlang.org](http://erlang.org/quick_start.html).

Put the following into a text file:

``` erlang
-module(test).
-export([fac/1]).

fac(0) -> 1;
fac(N) -> N * fac(N-1).
```

Save it as `test.erl` in a directory that's easy to get to with the Terminal. Then, from the Terminal, `cd` into that directory and type `erl` (which, if everything worked right, should start the Erlang command-line interpreter). From the interpreter, run the following commands: 

```
1> c(test).
{ok,test}
2> test:fac(20).
2432902008176640000
3> test:fac(40).
815915283247897734345611269596115894272000000000
```

**Note:** Lines starting with `N>` (where N is a number) are lines you should type (but just type the stuff coming after `N>`). The other lines represent output.

`c(test).` compiles test.erl (assuming it's in the directory you `cd`'ed into). `test:fac(20).` and `test:fac(40).` runs your factorial function.

So, that's what worked for me. If anyone has any problems along the way, leave a comment and I'll try to help.