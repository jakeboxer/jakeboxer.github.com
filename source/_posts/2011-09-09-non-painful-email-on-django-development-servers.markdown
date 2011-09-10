---
layout: post
title: "Non-painful email on Django development servers"
date: 2009-05-05 17:56
comments: true
categories:
---

I've been actively learning and using Django since August 2008, and I've loved almost every bit of it. There are plenty of places to read all about the virtues of Django, so I'll leave that out for now.

One thing that's always bugged me about web development in general is the sending of emails. I do development on my local computer (with a badly set up Apache / MySQL / PHP / Python / whatever else stack), and I've never felt like dealing with the headache of setting up a mail server. This means, when I add something that's supposed to send an email (like an activation email after registration), I have to get very hacky to test and debug it (making sure the email text is being produced correctly, making sure it's being sent to and from the right people, etc.).

This was one of the few web development pains that I thought Django didn't solve. Whenever I'd test a bit of code that was supposed to send email, I'd get a "Connection refused" error page (meaning my computer has no mail server to send the email with). I would usually add in a bit of printf debugging to make sure the subject and body had the correct text, but beyond that, I'd usually wait to test the email portions until I uploaded to a server that could send email (usually the production server, unfortunately).

Yesterday, I bumped into [a little section in the Django documentation that explains how to get around this](http://docs.djangoproject.com/en/dev/topics/email/#testing-e-mail-sending). As usual, Python has all the solutions. First, set this code in your settings.py file:

``` python
EMAIL_HOST = 'localhost'
EMAIL_PORT = 1025 # replace this with some free port number on your machine
```

Then, assuming you're on a Unix system (I'm on a Mac), run the following on the command line to start a "dumb" Python mailserver:

``` bash
python -m smtpd -n -c DebuggingServer localhost:1025
```

Make sure to replace 1025 with whatever you filled in for EMAIL_PORT.

Now, try running the email-sending code in your Python application. Voila! No error pages (or at least, none related to email), and the full text of the email (headers and all) appears in whatever command line prompt you ran the dumb mailserver on. This allows you to the see senders, recipients, subject, and body of the email being sent out, all without getting hacky or sending to an email account you own.

Taking this a step further, I created a small bash script called "dumbmail" in /usr/local/bin that looks like the following:

``` bash
#!/usr/bin/env bash
if [ -z $1 ]
then port=1025
else port=$1
fi

echo "Starting dumb mail server on localhost:$port"
python -m smtpd -n -c DebuggingServer localhost:$port
```

Now, when I'm testing a Django application and I get to a section that is going to send an email, I just run "dumbmail" (or "dumbmail some_number" if I need to use a different port, for some reason I can't imagine), and I'm ready to go.

Hope this helps people. The documentation was always there - I just never noticed that part until yesterday.