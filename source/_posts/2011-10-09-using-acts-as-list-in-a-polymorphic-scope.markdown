---
layout: post
title: "Using acts_as_list in a polymorphic scope"
date: 2011-10-09 15:52
comments: true
categories: 
---

If you're using [acts_as_list](https://github.com/swanandp/acts_as_list) in a polymorphic scope, you need to define the scope a little bit differently to make everything work right.

Here's the non-polymorphic example from acts_as_list's README:

``` ruby acts_as_list in a normal (non-polymorphic) scope
class TodoItem < ActiveRecord::Base
  belongs_to :todo_list
  acts_as_list scope: :todo_list # Note the single-item scope
end
```

We pass `:todo_list` as the `scope` (which `acts_as_list` converts to the `todo_list_id` column).

Now, let's say we're making a polymorphic `Picture` model (like in [the Polymorphic Associations section of the Rails Guides](http://guides.rubyonrails.org/association_basics.html#polymorphic-associations)), and we want pictures to be sortable and reorderable. Here's **the right way to do it**:

``` ruby acts_as_list in a polymorphic scope (the RIGHT way)
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
  acts_as_list scope: [:imageable_id, :imageable_type]
end
```

<!-- more -->

Note how, in the `Picture` model, the `scope` we pass to `acts_as_list` is an array of the two columns that make up the polymorphic association. If we were to follow our template from `TodoItem` and do it this way:

``` ruby acts_as_list in a polymorphic scope (the WRONG way)
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
  acts_as_list scope: :imageable
end
```

Everything would run correctly, but `imageable`s with the same ID and different types would have some problems. Specifically, determining which pictures are first and last:

``` ruby Bugs caused by using acts_as_list in a polymorphic scope the WRONG way
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
  acts_as_list scope: :imageable
end
 
class Employee < ActiveRecord::Base
  has_many :pictures, as: :imageable
end
 
class Product < ActiveRecord::Base
  has_many :pictures, as: :imageable
end

employee1 = Employee.create      # employee1 is an Employee with id=1
employee2 = Employee.create      # employee2 is an Employee with id=2
product1 = Product.create        # product1 is a Product with id=1

employee1.pictures.create        # Create a new picture on employee 1
employee2.pictures.create        # Create a new picture on employee 2
product1.pictures.create         # Create a new picture on product 1

puts employee1.pictures[0].last? # false, unexpected
puts employee2.pictures[0].last? # true
puts product1.pictures[0].last?  # true
```

Since each `imageable` has only one `Picture`, `last?` (an `acts_as_list` method that checks if the item is the last one in scope) should return `true` for all three. But, the first line says `false`. Since we're only telling `acts_as_list` to scope on `:imageable` (which it converts to `imageable_id`), it has no way of telling between the `Employee` with an `id` of 1 and the `Product` with an `id` of 1. So, it thinks that Employee 1's picture belongs to the same record as Product 1's picture, and thus, thinks that only Product 1's picture is `last?`.

If you tell `acts_as_list` the full story, it won't get confused:

``` ruby Correct results from using acts_as_list in a polymorphic scope the RIGHT way
class Picture < ActiveRecord::Base
  belongs_to :imageable, polymorphic: true
  acts_as_list scope: [:imageable_id, :imageable_type]
end
 
class Employee < ActiveRecord::Base
  has_many :pictures, as: :imageable
end
 
class Product < ActiveRecord::Base
  has_many :pictures, as: :imageable
end

employee1 = Employee.create      # employee1 is an Employee with id=1
employee2 = Employee.create      # employee2 is an Employee with id=2
product1 = Product.create        # product1 is a Product with id=1

employee1.pictures.create        # Create a new picture on employee 1
employee2.pictures.create        # Create a new picture on employee 2
product1.pictures.create         # Create a new picture on product 1

puts employee1.pictures[0].last? # true
puts employee2.pictures[0].last? # true
puts product1.pictures[0].last?  # true
```

Now that `acts_as_list` knows to look at both the `imageable_id` and the `imageable_type`, it can tell the difference between Employee 1 and Product 1, and returns `true` for all three `last?`s, as expected.