---
layout: post
title: "My pre-ARC Objective-C memory management conventions"
date: 2011-10-03 12:06
comments: true
categories: 
---

iOS 5 is coming soon, and introduces ARC. ARC will make Objective-C memory management significantly simpler. But, it will be a long time before iOS 5 is ubiquitous, and I'm not sold on the subset of ARC that will be available for iOS 4. So, in the meantime, I thought people might benefit from seeing the conventions I use to keep Objective-C memory management simple. Most iOS developers probably already follow something close to this, but it doesn't hurt to have it written out.

If you follow these five simple rules, you should pretty much never get a memory leak or a crash from over-releasing.

---

1. Always declare @properties for your object instance variables
===
If you're putting an instance variable on a class and it's an object, declare a property for it

``` objc Declaring properties
@interface MyObject : NSObject

@property (nonatomic, retain) NSDate* someDate;
@property (nonatomic, retain) AVPlayerLayer* someLayer;
@property (nonatomic, retain) NSMutableArray* someMutableArray;
@property (nonatomic, copy) NSString* someString;
@property (nonatomic, copy) NSArray* someImmutableArray;
@property (nonatomic, assign) id<MyObjectDelegate> delegate;

@end
```

By default, you should always declare your object properties as `retain`.

If you're declaring a property for an object that's expected to be immutable, but has a mutable subclass (for example, if you're declaring an `NSArray` property, which has the `NSMutableArray` subclass), declare it as `copy` instead of `retain`.

If you're declaring a property for an object that is *guaranteed* to be retained elsewhere (for example, a delegate, or something that you're also storing in an array and just need a convenient shortcut to), you can declare it `assign`. Be very careful with this though. If you're unsure, just do `retain` or `copy`.

You'll also need to `@synthesize` all these properties in your `@implementation`:

``` objc Synthesizing properties
@implementation MyObject

@synthesize someDate;
@synthesize someLayer;
@synthesize someMutableArray
@synthesize someString;
@synthesize someImmutableArray;
@synthesize delegate;

// ...

@end
```

Finally, you'll need to release all of the `retain` and `copy` properties (but *not* the `assign` properties) in your `dealloc`. This is the *only place in your code* where you should refer to your properties without `self.` (which means you're actually accessing the instance variables rather than the properties).

``` objc Releasing properties
@implementation MyObject

// ...

- (void)dealloc {
  [someDate release]; // Note that we're using someDate, not self.someDate
  [someLayer release];
  [someMutableArray release];
  [someString release];
  [someImmutableArray release];
  // Note that we aren't releasing delegate
}
```

2. Always use `self` when accessing a property
===
If you don't use `self`, you will be accessing the underlying instance variable directly, rather than going through the @synthesized getter/setter.

``` objc
self.someImmutableArray = [NSArray array];             // Correct!
NSLog(@"printing array: %@", self.someImmutableArray); // Correct!
someImmutableArray = [NSArray array];                  // WRONG
NSLog(@"printing array: %@", someImmutableArray);      // WRONG
```

If you skip the @synthesized getter/setter, you skip the memory management rules you defined in your `@property`, which will lead to problems. The only exception to this is mentioned in rule 1: in your `dealloc` method, when releasing the object.

3. Make sure all newly-created objects are autoreleased
===
There is a bit of disagreement on the community on this one, but I have yet to hear a compelling argument for all but the most extreme cases, so we'll go with the side that makes life easier. Whenever you create a new object, **make sure it is autoreleased**.

The convention for this is as follows: all methods that return objects return autoreleased objects, unless the method name starts with `init` or `copy`. For example:

``` objc
self.someNumber = [NSNumber numberWithInteger:5];       // autoreleased
self.someNumber = [[NSNumber alloc] initWithInteger:5]; // not autoreleased
```

If you create an object with a method that starts with `init` or `copy`, make sure you explicitly tell it to autorelease:

``` objc
self.someNumber = [[[NSNumber alloc] initWithInteger:5] autorelease]; // autoreleased
```

This will save you *a lot* of headaches. Any objects that you don't want to keep will be released automatically. Any objects that you do want to keep should be assigned to a property, and since the property is `retain` or `copy` (as mentioned in rule 1), they will be retained and kept automatically.

4. Never call `retain` on anything
===
The method `retain` is not called once in all the lines of Objective-C code in my current code base. If you find yourself needing to explicitly retain an object, create a `retain` or `copy` property for it instead (as mentioned in rule 1) and assign it to that property.

5. Never call `release` on anything (except in `dealloc`)
===
If you're following rule 4, there should never be a time when you explicitly need to release an object, except in your class's `dealloc` method (as mentioned in rule 1).

---

If you follow all these rules, you should never have to think about memory management. If you deviate from any of them, you could end up in weird tough-to-debug situations, so try your hardest not to. If you find yourself in a situation where you think you absolutely need to deviate, write a comment on this post explaining your situation, and I'll see if I can help.