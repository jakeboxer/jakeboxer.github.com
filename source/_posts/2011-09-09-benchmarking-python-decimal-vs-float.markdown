---
layout: post
title: "Benchmarking Python decimal vs. float"
date: 2009-03-17 17:19
comments: true
categories:
---

I'm writing a web app that includes, among other things, a good amount of (rational) non-integer numbers.  Whenever I'm in this situation, and I'm using a language that supports Decimals (as opposed to just floats and doubles), I always wonder which one I should use.

If you're a programmer, you understand the difference and the dilemma.  Floats/doubles are very fast, as all computers (built within the last 15 years) have hardware specifically made to deal with them.  However, they're not perfectly accurate; because of binary representation, numbers that we use a lot (like 1/10 or 0.1) cause the same problems that 1/3 (0.33333...) cause in base 10.

Decimals, on the other hand, are slow.  They are handled entirely in software, and thus take hundreds of instructions to do things that would take less than 10 with floats/doubles.  The upside is that they're perfectly accurate; 0.1 is 0.1 is 0.1.

So the question becomes twofold:

* Do I really need my numbers to be perfectly accurate?
* How much slower are decimals than floats/doubles?

In my case, the accuracy would be nice, but not completely necessary. And thus, the latter question becomes important. I'm not writing a large application, and I don't expect it to get too popular too quickly, so if the slowdown is only moderate, I'll take the accuracy.

To learn what the slowdown was, I wrote two quick Python test programs:

``` python Decimal test
from decimal import Decimal

a = 0

for i in range(0, 20000):
  a = Decimal('%d.%d' % (i, i))
  print(a)
```

``` python Float test
from decimal import Decimal  # kept this in on the float version
                             # so they'd have the same overhead

a = 0

for i in range(0, 20000):
  a = float('%d.%d' % (i, i))
  print(a)
```

When I ran each of these with /usr/bin/time (which I just learned about a couple weeks ago, and has replaced counting seconds on my fingers as my favorite benchmarking tool), the decimal version took an average of about 1.5 seconds (over 10 runs), while the float version took an average of 0.5.  Just to make sure no overhead was getting in the way, I upped the limit to 40000 and ran it again. Decimal took 3.0 seconds, float 1.0. I can now confidently say that **Python floats are about 3x the speed of Python decimals**.

Or are they? While this tests the creation and printing of decimals and floats, it doesn't test mathematical operations.  So, I wrote two more tests.  I'm going to be doing a lot of division on these numbers, and that's definitely the most expensive mathematical operation to compute, so I made sure to do it in the tests (along with some subtraction).

``` python Decimal version
from decimal import Decimal

a = 0

for i in range(2, 20002):
  a = Decimal('%d.%d' % (i, i)) / Decimal('%d.%d' % (i - 1, i - 1))
  print(a)
```

``` python Float version
from decimal import Decimal

a = 0

for i in range(2, 20002):
  a = float('%d.%d' % (i, i)) / float('%d.%d' % (i - 1, i - 1))
  print(a)
```

This time, the float version averaged about 0.6 seconds (1.15 with 40,000 iterations instead of 20,000), while the decimal version averaged over 11 seconds (23 with 40,000 iterations instead of 20,000). So while Python float creation and printing is merely 3x as fast as Python decimal creation and printing, **Python float division is almost 20x as fast as Python decimal division**.

So what did I choose? Decimals. In the context of these tests, the decimal slowdown may seem significant, but if I finished my app using decimals and profiled it, I can almost guarantee (based on the speeds here) that the bottleneck would not be decimal division performance. If I was running an app that was handling hundreds of simultaneous requests, I may consider switching (I may also spring for better hardware, but that's a different topic). However, for my purpose, 1/20th the speed of floats is more than fast enough.

P.S. As my very late discovery of /usr/bin/time should suggest, I'm extremely new to benchmarking. If anyone has any suggestions for me, or criticisms of my method, please leave your thoughts. This is something I'd like to get better at.