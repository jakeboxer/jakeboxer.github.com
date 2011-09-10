---
layout: post
title: "How to reset/revert a single file with Git"
date: 2010-07-08 20:41
comments: true
categories: 
---

I'm making this post more for my own reference than to help anyone else, but feel free to comment if you have questions or anything.

If you've made a commit with git (let's call it "commit A"), and you want to make another commit (let's call it "commit B") that, among other things, reverts the changes made to a single file (let's call it file1.rb) in commit A, use the following command:

`git reset commit-a -- path/to/file1.rb`

That'll create two sets of changes: a copy of the changes you made in commit A, and the inverse of the changes you made in commit A. The inverse is staged, while the copy is unstaged. Nine times out of ten, you'll want to commit the staged changes (which, as they're the inverse of the changes in commit A, will result in a revert of file1) and discard the unstaged ones.