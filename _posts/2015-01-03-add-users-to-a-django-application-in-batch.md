---
layout: post
title: Add users to a Django application in batch.
description: "A small script to quickly add new users to a Django application."
modified: 2015-01-03
tags: [django, admin, script]
---

As a Django administrator, I often have to create user accounts for a demo or training session.  Below
is a small script that can be run from the Django shell to create users in batch.

{% highlight python %}
import sys
from django.contrib.auth import authenticate
from django.contrib.auth import get_user_model

User = get_user_model()

#  Update the users in this list.
#  Each tuple represents the username, password, and email of a user.
users = [
    ('user_1', 'phgzHpXcnJ', 'user_1@example.com'),
    ('user_2', 'ktMmqKcpJw', 'user_2@example.com'),
]

for username, password, email in users:
    try:
        print 'Creating user {0}.'.format(username)
        user = User.objects.create_user(username=username, email=email)
        user.set_password(password)
        user.save()

        assert authenticate(username=username, password=password)
        print 'User {0} successfully created.'.format(username)

    except:
        print 'There was a problem creating the user: {0}.  Error: {1}.' \
            .format(username, sys.exc_info()[1])
{% endhighlight %}