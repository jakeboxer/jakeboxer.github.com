---
layout: post
title: "Little Joys of Rails (Part 1) - Renaming Associations"
date: 2009-11-04 19:46
comments: true
categories: 
---

I've spent the last month or so teaching myself the basics of [Ruby on Rails](http://www.rubyonrails.org/). By far my favorite thing about it is that, at every turn, I find a little feature or helper that works really well and does exactly what I want without feeling messy or "magical".

Unfortunately, I have no one to share these little joys with, as pretty much none of my programmer friends know Ruby or Ruby on Rails. So, I figured I'd start sharing them on here. Hopefully, Rails newbies (like myself) will learn something, and Rails veterans will be able to chuckle knowingly as they fondly look back on the glory days of learning Rails.

Renaming Associations
---------------------

If you have one model that `belongs_to` another, the association (in both directions) is named after the models. For example, if I have Users and Courses, and I want a course to `belong_to` its teacher (who is a User), I would start with the following:

``` ruby
class User < ActiveRecord::Base
  has_many :courses
end

class Course < ActiveRecord::Base
  belongs_to :user
end
```

This is nice and simple, but if I want to access a course's teacher, I have to do it like this:

``` ruby
my_course = Course.first
puts my_course.user # Print out the course's teacher
```

That's a little confusing. What if my course also `has_many` students (also Users)?

``` ruby
my_course = Course.first
puts my_course.users.first # Print out the course's first student
```

So, `my_course.user` is the teacher, and `my_course.users` are the students. There's nothing to indicate that, I just have to know it. Wouldn't it be nicer if the associations could be named for what they represent?

``` ruby
class Course < ActiveRecord::Base
  belongs_to :teacher, :class_name => 'User'
end
```

To make this work properly, the foreign key needs to be `teacher_id` instead of `user_id` (that can be overridden too, but I won't go into that right now). Then, you get to do this:

``` ruby
my_course = Course.first
puts my_course.teacher # Print out the course's teacher
```

Much better!

Now, I recognize that this is far from a revolutionary feature. I'm pretty sure it's possible in Django and most other MVC frameworks. I was just very happy to bump into it.