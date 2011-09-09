---
layout: post
title: "Array Slicing Is Messed Up"
date: 2011-09-09 15:56
comments: true
categories: 
---

The mechanics of array-slicing (and, by extension, string-slicing) have bothered me since my very first Computer Science class. For those of you who aren't programmers but are braving the programmer warning, let me explain a little.  Arrays in programming are essentially lists of data. For example, I could keep an array of test grades, and it might look something like this:

``` python
array = [99, 72, 85, 85, 100, 61, 88, 32]
```

Arrays throw beginners off a little bit, because indices are zero-based. This means that if I want to access specific elements in an array, the first element is considered element 0, the second is element 1, etc. An example using the above array:

``` python
print array[0]  # prints '99' (the 1st element in the list)
print array[4]  # prints '100' (the 5th element in the list)
print array[7]  # prints '32' (the 8th and last element in the list)
```

The concept of "array-slicing" means creating a new array out of a subsequence from a previous array. For example, a "slice" of the above array could be:

``` python
newArray = [85, 100, 61]  # a slice containing the 4th-6th elements of the original array
```

Most programming languages provide a function that allows you to take a "slice" of an already-existing array. They do this by asking you to specify where in the original array you'd like to start the slice, and where you'd like to end it. This is where my complaint comes in. Most languages ask you to specify two numbers: the index of the first element you want, and the index *after* the last element you want. For example, to get [85, 100, 61] out of the original array, I would do the following:

``` python
newArray = array.slice(3, 6)  # creates a new array with the 4th, 5th, and 6th elements from the original array
```

To me, this is silly and unnecessarily confusing. Why is the second number the index *after* the last element you want? Wouldn't it make more sense for it to just be the index *of* the last element you want, like the first number is the index of the first element you want?

I've heard people say that this is intended to make it easier to take a slice of the last N elements in a list. This would be because of the length() function that arrays in most languages have. A quick example:

``` python
array = [99, 72, 85, 85, 100, 61, 88, 32]
print array.length()  # prints '8', since there are 8 elements in the array
```

You can combine this with the array slicing function to take a slice of the last N elements in an array, like this:

``` python
array = [99, 72, 85, 85, 100, 61, 88, 32]
newArray = array.slice(3, array.length())  # equivalent to newArray = array.slice(3, 8)
```

Since length() gives back the length of the list, it's guaranteed to give back a number one higher than the highest index (since, as we said before, indices are zero-based). This means that plugging length() into the second position of the slice() function (which refers to the index *after* the last index you want) is the equivalent of saying "up to the last item in my array."

So I can see the convenience of this. But I refuse to believe that this is the primary reason for this ass-backwards method of array-slicing. At least in my experience (which I admit is not comprehensive, and this may be where my misunderstanding lies), slicing the end of an array is *not that common*, and certainly not common enough to warrant the obfuscation of a commonly-used function. If array-slicing was done the way I want (where the last number refers to the index of the last element you want), it would be easy enough to do it like this:

``` python
array = [99, 72, 85, 85, 100, 61, 88, 32]
newArray = array.slice(3, array.length() - 1)  # equivalent to newArray = array.slice(3, 7)
```

In fact, it would be even easier to provide a second version of slice() (and I believe some languages actually do this) which only asks for the first index. When the language sees that you've only included one index instead of two, it *assumes* that the last parameter refers to the end of the list.

``` python
array     = [99, 72, 85, 85, 100, 61, 88, 32]
newArray1 = array.slice(3, 5)  # equivalent to newArray = [85, 100, 61]
newArray2 = array.slice(3)     # equivalent to newArray = array.slice(3, 7) = [85, 100, 61, 88, 32]
```

Obviously, this is a pretty minor thing in programming; it just strikes me as weird, especially because every language does it this way. The majority of the time, when I have to do an array-slice, I look the method up online, to make sure I remember how the indexing works. It's unusual for such a commonly-used function (besides one with lots of extra syntax, like [date() in PHP](http://www.php.net/date)) to require a reference to the documentation on every use. I'm not suggesting that it's straight up *wrong* and should be changed immediately; I just don't understand why it works this way in every language.