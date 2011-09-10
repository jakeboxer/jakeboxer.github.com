---
layout: post
title: "A method with no unit tests is a broken method"
date: 2009-11-17 20:00
comments: true
categories: 
---

If you write software, you need to write unit tests. If you've written a method/function, and you haven't written a unit test for it, it's safe to assume that it's broken (even if it compiles and your other tests pass).

I'm not necessarily advocating full-fledged test-driven development. I'm just saying, if you release code into "the wild," and there are methods you haven't unit tested, your customers will almost certainly run into multiple bugs in each one of them.

That's an atomic point. Separate from that, I'd like to mention that this isn't always a bad thing. For a startup that wants to iterate as quickly as possible (and is writing non-life-critical software), writing the code with no unit tests, releasing it, and reproducing each customer-discovered bug with a unit test before fixing it is a totally reasonable model. These startups just shouldn't operate under the illusion that their software "works". In the hours after they make one of these releases, they should feel blessed if a single customer is able to register or log in.