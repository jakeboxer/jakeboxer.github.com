---
layout: post
title: "The importance of attr_accessible in Ruby on Rails"
date: 2010-01-28 20:27
comments: true
categories: 
---

I'm sure this has been written about ad nauseum, but I spent some time yesterday explaining it to someone who didn't understand, and now I feel like writing it up a bit more formally.

What is attr_accessible?
------------------------

In Ruby on Rails, [attr_accessible](http://api.rubyonrails.org/classes/ActiveRecord/Base.html#M002281) allows you to specify which attributes of a model can be altered via mass-assignment (most notably by `update_attributes(attrs)` and `new(attrs)`). Any attribute names you pass as parameters will be alterable via mass-assignment, and all others won't be.

How does mass-assignment work normally?
---------------------------------------

By default, mass-assignment methods accept a hash of attribute values, each keyed by their associated attribute's name. If I ran the following code:

``` ruby
User.new({ :name => 'Harry Potter', :email => 'hp@hogwarts.com' })
```

A new instance of the `User` model would be created, and the `name` and `email` attributes would be set accordingly. It can also be used to alter related models. For example:

``` ruby
User.new({
  :name => 'Albus Dumbledore',
  :is_teacher => true,
  :course_ids => [1, 2, 3] })
```

In addition to creating a user with the appropriate attributes, this will update the specified courses to be owned by this user(assuming a user `has_many` courses in our app).

How can this be abused?
-----------------------

Very easily. What if someone did this:

``` ruby
User.new({ :name => 'Draco Malfoy', :is_teacher => true })
```

This Draco Malfoy fellow may not actually be a teacher, but the system is none the wiser. Of course, the developer would never code this; in a real Rails app, the code is going to look like this:

``` ruby
User.new(params[:user])
```

The elements in `params[:user]` are taken from the POST/GET/PUT data passed along when the action was run. They're thrown blindly into the mass-assignment, and any attributes whose names match the keys will be set.

*"So what's the big deal? Just don't include an 'is_teacher' field in the web form, and the param won't be there."* This is true for innocent users, but the malicious ones (and Draco Malfoy is definitely a malicious one) have an easy way around this. A web form is just a way to make it easy for users to pass data to your app. There are other ways. For example, if I wanted to register for the app via the command line instead of a browser, I could do it like this:

``` bash
curl -d "user[name]=Harry Potter&user[email]=hp@hp.com" \
    http://myapp.com/users/
```

This sends a request to `http://myapp.com/users/` and passes data in the exact format it would've appeared if I'd filled out a web form that asked for a name and email address. However, I could also do this:

``` bash
curl -d \
    "user[name]=Draco Malfoy&user[email]=m@hp.com&user[is_teacher]=1" \
    http://myapp.com/users/
```

Since `is_teacher` is an attribute name in my User model, and mass-assignment methods blindly accept whatever attributes they see, Draco Malfoy has just set himself a teacher.

Even worse, I could use this to grab courses that may not be mine.

``` bash
curl -d \
    "user[name]=Draco Malfoy&user[course_ids]=1&user[course_ids]=2" \
    http://myapp.com/users/
```

Draco Malfoy has now taken courses 1 and 2 away from whoever they originally belonged to (Dumbledore, if my memory serves me) and given them to himself.

How can we prevent this?
------------------------

There are a few obvious but clumsy ways. We could skip mass assignment, setting each individual attribute in our controller, but this will introduce a lot of duplicate and unnecessary code. We could explicitly pull unwanted parameters out:

``` ruby
params.delete(:is_teacher)
params.delete(:course_ids)
```

This also introduces a lot of duplicate code. If we ever add new columns that we want to restrict, or decide we want to unrestrict a column, we're going to have to go through the `create` and `update` actions, and any others that perform mass assignment.

We could factor these out into some sort of `sanitize_params` method on each model. This is a better solution, but you still have to call it in every action that alters the data. It's definitely not as good as the built-in one: `attr_accessible`. We can add this to the top of the `User` model:

``` ruby
attr_accessible :name, :email
```

This white-lists `name` and `email`; these two attributes will be accepted from a mass-assignment method, while *all others* will be ignored. This is by far the safest way to do it; only attributes you've explicitly allowed (which hopefully means you've thought carefully about them) can be set by mass-assignment. This way, if some intern comes along and adds a bunch of dangerous columns or relations (`payment_accepted` or `horcruxes`, for example), no one has to think about updating the `sanitize` methods.

What does this <em>not</em> do?
-------------------------------

I saw one person say "Why would I put anything in `attr_accessible`? Why would I want any of my attributes to be hackable?"

Make no mistake: `attr_accessible` is no substitution for proper access control. If all users have write access to all other users, `attr_accessible` will let one user change another's `name` attribute if it's specified. Regular authentication and access control must be used to prevent users from writing to model instances that they shouldn't be able to write to. Once this is done correctly, `attr_accessible` can be used to prevent a malicious user from altering data of her own that she shouldn't be able to alter.

To be more clear, it could be considered "hacking" if a user were able to change everyone's `name` to "Voldemort". `attr_accessible` can't prevent this; you need to do proper authentication with something like [Authlogic](http://github.com/binarylogic/authlogic). Once you've set your controllers up to prevent a user from even attempting to change another user's data, you've prevented this "hack".

If the user tries to change *his own* name to "Voldemort", that's totally fine. We don't care if he does it via the web app, curl, or anything else; users are allowed to change their own name. Including `:name` in `attr_accessible` isn't making it "hackable", because it's an attribute that users `should` be able to change.

If the user tries to change his `is_teacher` attribute from `false` to `true`, that's also considered "hacking". We don't want to let users do this, so we exclude `:is_teacher` from `attr_accessible` to prevent it.

Are attributes excluded from attr_accessible immutable?
-------------------------------------------------------

**No. They can still be altered, just not via mass-assignment.** If I exclude `is_teacher` from `attr_accessible`, and I go:

``` ruby
hagrid = User.first(:conditions => { :name => 'Rubeus Hagrid' })
hagrid.is_teacher = true
hagrid.save
```

That will work just fine. The difference is, it forces you to set the attribute explicitly, so there's no potential of accidentally setting an attribute unexpectedly passed to mass-assignment. This way, I can allow my non-dangerous attributes to be set via mass-assignment with `attr_accessible`, then explicitly provide or deny control over dangerous attributes in other actions.