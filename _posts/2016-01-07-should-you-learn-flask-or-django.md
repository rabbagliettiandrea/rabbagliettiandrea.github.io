---
layout: post
title: "Should you learn Flask or Django?"
date: 2016-01-07
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes.
img: flask-vs-django.jpg
---
Flask just seemed a lot more straightforward to me. There was no "magic", just a small API to learn. 
Django's community is very strong, particularly on IRC. There is no forum that I've found that is
popular for them, though, which I found a little odd. Flask... well, there isn't much to Flask.
There is a Werkzeug community and a Jinja2 community, and I've found lots of information I needed 
via search for both of those. I've not tried to find them on IRC, as I'm at work and it is blocked.

> /r/django is far more active that /r/flask, though.

The thing is, with Flask, I've not needed the support. The documentation is very good, 
and it's so small there really isn't a lot to go wrong. I'm learning different pieces of 
the framework as I develop more advanced needs, and they stay out of the way until then. 
With Django, I often found myself having to learn why things were structured the way there were, 
before I could learn the system. An example of this would be forms processing - on Django, 
I remember having to learn about the Forms models, and all of their configuration. 
Then I had to learn about Forms validation, and how they were handled in the view
(by branching the logic based on request type). Finally, I had to learn about how to tie a form to a Model, 
or to use a ModelForm.

With Flask... there is no forms processing included to my knowledge. 
I built the form in HTML (and wrote some helper functions in the process),
validated the data on an ad-hoc basis, then wrote the SQL to drop it in the DB.
There are extensions that add form processing to Flask, but I didn't use them.

**In this use case, a competent Django developer could get a CRUD form up and running in a
shorter time than a competent Flask developer** - but by the time I would consider myself 
"competent", I will have developed my own library of helpers functions and architectural 
conventions, and will have learned a lot about Python in the process. It isn't really a fair comparison
- no one sits down with only one module and writes everything else from scratch.

I think I'll ultimately end up using Django for complex application development, 
and Flask (or its components, Werkzeug and Jinja2) for writing things that are 
less well-defined or don't quite fit into the mold poured by Django. 
I don't think it's fair to say that one is better than the other overall,
 but I stand behind my contention that Flask is better suited to a **new Python programmer**.

## But definitevly, I'm in love with Django :)

Anyway, a rule of thumb could be:

If you don't need:
* django-admin
* django ORM
* form / modelform facilities

or you're a newbie web-developer, then *flask* is you choice. But, for everything else..

## come on, we're djangonaut my friend, trust your soul ^^

