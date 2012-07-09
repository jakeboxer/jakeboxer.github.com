---
layout: post
title: "Converting to the new RSpec 2.11 expectation syntax"
date: 2012-07-09 15:28
comments: true
categories: 
---
[RSpec 2.11](http://blog.davidchelimsky.net/2012/07/07/rspec-211-is-released/) was released a few days ago. The biggest change is [a brand new expectation syntax](http://myronmars.to/n/dev-blog/2012/06/rspecs-new-expectation-syntax) by [Myron Marston](http://myronmars.to/n/dev-blog).

Now, instead of writing this:

``` ruby Old expectation syntax
foo.should eq(bar)
foo.should_not eq(bar)
```

You can write this

``` ruby New expectation syntax
expect(foo).to eq(bar)
expect(foo).not_to eq(bar)
```

[Myron's blog post](http://myronmars.to/n/dev-blog/2012/06/rspecs-new-expectation-syntax) does a fantastic job explaining why this is better. To summarize:

1. RSpec does not own every object in the system, so it cannot add the `should` and `should_not` methods to every object. This can lead to some really confusing errors. Using `expect` works around this.
1. Block expectations already used the `expect` syntax out of necessity. The inconsistent syntax made it harder to read specs that used both. Using `expect` for normal expectations unifies the syntax, making these specs much more readable.

What follows is a step-by-step guide to convert all your specs from the old syntax to the new one as quickly as possible.

## Change all equality operator matchers

The new expectation syntax does not support `foo.should == bar`. You need to do `expect(foo).to eq(bar)`. So, go through all your specs and make this change.

You can use this regular expression: 

``` ruby Regular expressions for replacing == with eq()
# Note that these both start with a leading space. Don't forget them, or they won't work.

# Find
/ == (.*)$/

# Replace  
/ eq($1)/
```

For global find/replace stuff, I prefer to open and editor and click "Replace" over and over. It's slower, but I don't have to track down mistakes afterwards, and I get to improve my regex as I go. Feel free to do it however you want, but make sure to check your diff afterwards.

Once you're done, re-run your specs and make sure everything is still passing. Note that we haven't yet changed `should` to `expect`, so everything should be passing.

## Change other unsupported operator matchers

These should be rare enough that you don't need a regex. If you do, write it and send it to me, and I'll include it in here and give you credit.

Go through all your specs and make these changes:

``` ruby Unsupported operator matcher replacements
# Regex matching
"a string".should =~ /a str[iou]ng/ # Old, bad.
"a string".should match(/a str[iou]ng/) # New, good.

# Matching array contents while ignoring order
[1, 2, 3].should =~ [2, 3, 1] # Old, bad.
[1, 2, 3].should match_array([2, 3, 1]) # New, good.

# Comparison matchers (do this with ALL matchers: <, <=, >, >=)
foo.should < 10 # Old, very bad. You should never have been using this.
foo.should be < 10 # New, good
```

Re-run your specs and make sure everything is still passing.

## Disallow "should" from your specs

Open your `spec/spec_helper.rb` file and add the following lines to your configuration block:

``` ruby spec/spec_helper.rb
RSpec.configure do |config|
# .. other stuff

  # Add all three of these lines:
  config.expect_with :rspec do |c|
    c.syntax = :expect
  end

# .. other stuff
end
```

This will stop RSpec from adding `should` to all objects. Make sure you add all three lines; I didn't realize it was within a block at first, and got errors.

If you run your specs now, pretty much everything should fail.

## Replace should and should_not with expect

This is the most tedious part. Luckily, I came up with a regex that works for both `should` and `should_not`. Here it is:

``` ruby Regular expressions for replacing should/should_not with expect
# Find
/^(\s+)(.*(?=.should))\.should(_)?(not)? (.*)$/

# Replace
/$1expect($2).$4$3to $5/
```

One caveat: this will *not work* for expectations that aren't on their own line. Example (using [Capybara](https://github.com/jnicklas/capybara))

``` ruby Working and non-working examples for regex
# Regex WON'T find/replace properly
within('#header') { page.should have_content('About') }

# Regex WILL find/replace properly
within '#header' do
  page.should have_content('About')
end
```

The first example won't work, because the expectation comes after `within('#header') {`. The second will work, because the expectation is on its own line.

So, like I said above, don't just do one "Replace All"; click the "Replace" button in your editor over and over, so you can spot these broken cases and fix them manually.

I tried for awhile to fix these broken cases, but I'm pretty sure it's not possible via regex. If someone can prove me wrong, I'd love to update this guide and give you credit.

After doing this in all your specs, re-run them. Everything should pass. If not, comment, and I can try to help you figure out what's up.