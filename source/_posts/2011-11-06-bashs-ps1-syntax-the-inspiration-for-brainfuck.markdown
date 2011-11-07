---
layout: post
title: "Bash's PS1 Syntax: The Inspiration for Brainfuck?"
date: 2011-11-06 19:24
comments: true
categories: 
---

I just spent way too much time struggling to get my Bash `PS1` variable working right. Originally, it looked like this (`parse_git_branch` is a Bash function I have defined elsewhere):

``` bash Old PS1
PS1="\w\e[35m\$(parse_git_branch)\e[m > "
```

Now, it looks like this:

``` bash New PS1
PS1="\w\[\e[35m\]\$(parse_git_branch)\[\e[m\] > "
```

The difference? I wasn't escaping my color codes correctly, which was causing Terminal to wrap long commands onto the beginning of the same line.

Let me break down Bash's color code syntax, because it makes me so angry. A minimal Bash color code looks like this:

``` text Minimal Bash color code
\e[1;30m
```

`\e[` means "here comes a color code" I guess.

`1` means "make it bold". If you don't want bold, leave it out.

`;` means "I'm finished telling you that this is bold". If you don't want bold, or if you want default-colored text (explained below), leave it out.

`30` means "dark gray". Here's an exhaustive list of valid numbers (no number means default color):

- `30` = dark gray
- `31` = red
- `32` = green
- `33` = yellow
- `34` = blue
- `35` = purple
- `36` = turquoise
- `37` = light gray

0-29 don't do anything (at least in Terminal.app on OS X 10.7.2). No idea why they start at 30.

`m` means "my color code is over".

If you match that format, all subsequent characters will be that color (until you define another color code).

**But that's not all!** This sequence is escaped incorrectly. If you have it in your PS1, Terminal will look correct on startup, but if you type a long command that wraps past the first line, Terminal will wrap it back to the beginning of the line you're on and start overwriting characters.

I bet you think the fix is to match the open square bracket with a closed one, right? If so, give yourself a half pat on the back, then punch yourself in the face. Here's how to escape it correctly.

``` text How to escape a Bash color code
\e[1;30m     # Wrong
\[\e[1:30m\] # Right
```

You need to close match the open bracket (with an escaped close bracket), but also add *another* open (escaped) bracket to the front, also with no closer.

Anyone wanna educate me on the reason for all this madness? Right now, it just feels like some dude thought regular expression and strftime syntax were too verbose.