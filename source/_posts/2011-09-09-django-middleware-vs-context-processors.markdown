---
layout: post
title: "Django middleware vs. context processors"
date: 2009-05-18 18:00
comments: true
categories:
---

This may be old news to many people, but it's something I just recently learned (after doing it wrong for about nine months), so I figured someone else may be able to benefit from my mistakes.

For a long time, when I needed to access the currently logged-in user in one of my Django views, they would follow this pattern:

``` python
from django.template import RequestContext

# some other view code

def my_view (request, obj_id):
  context = RequestContext(request)
  obj     = get_object_or_404(SomeModel, pk=int(obj_id))

  # do some stuff, including:
  some_function(context["user"])

  return render_to_response("some_url_name", {"some_var": some_val},
    context_instance=context)
```

Now, seasoned Django vets (why are you reading this post, by the way?) are probably laughing, but this seemed perfectly fine to me.  Luckily, my ignorance was revealed to me while asking on IRC about a problem which, while didn't seem so at the time, was completely tied to my misuse of RequestContext.

To put it simply, context processors are made to be used in templates. The only time they should ever be instantiated is at the very end of a view, like so:

``` python
return render_to_response("some_url_name", {"some_var": some_val},
  context_instance=RequestContext(request))
```

I was using them all over views, and even in a few of my decorators. What's wrong with this? The main thing is, certain mutation functions are sometimes performed when a RequestContext is instantiated (such as user.get_and_delete_messages(), which gets all of a user's messages from the database and then deletes them), and performing them multiple times before loading the template can cause unexpected results.  In my example, I was instantiating a RequestContext in a bunch of decorators, which meant that by the time my template was loaded, all of the user's messages were deleted (and stored in an earlier instance of RequestContext), making it look to me as if my auth messages were being thrown out.

What's the solution? Middleware. Middleware allows the programmer to attach variables to the request object before it reaches the view. With this (and the provided django.contrib.auth.middleware.AuthenticationMiddleware), I can still achieve my original goal (accessing the logged-in user from a view), but now I can do it without creating a RequestContext and potentially running mutation functions multiple times:

``` python
from django.template import RequestContext

# some other view code

def my_view (request, obj_id):
  obj = get_object_or_404(SomeModel, pk=int(obj_id))

  # do some stuff, including:
  some_function(request.user)

  return render_to_response("some_url_name", {"some_var": some_val},
    context_instance=RequestContext(request))
```

I've removed all the references to RequestContext from my decorators, and made sure to only instantiate it at the very end of views. Now, messages work perfectly. I've even written my own middleware (which you can learn how to do [here](http://docs.djangoproject.com/en/dev/topics/http/middleware/)) to load instances of a few of my own models into the request.

To reiterate, I'm aware that this is common knowledge for many people, but it took me 9 months of moderate Django use and embarrassment on IRC to discover it for myself. Hopefully, this will help someone else in a similar position.