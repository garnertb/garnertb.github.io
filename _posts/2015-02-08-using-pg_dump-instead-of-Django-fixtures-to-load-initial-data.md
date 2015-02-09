---
layout: post
title: Using pg_dump instead of Django fixtures to load initial data.
description: "How to use pg_dump instead of fixtures to bootstrap large data sets in a Django application."
modified: 2015-02-08
tags: [django, postgresql, fixtures, psql]
---

Over the last several years, I have found the Django fixture system to be pretty good at bootstrapping Django applications
with initial data.  Recently though, while working on a geospatial application with a fair amount of features I
went to ```loaddata``` and got back this: ```MemoryError: Problem installing fixture 'fixtures/initial_data.json'.```  A quick
Google search provided a few helpful StackOverflow responses that said to use "database backup tools" in this scenario but didn't include any
 information as to how.  Here are some notes on how I used ```pg_dump``` to dump and load Django objects when fixtures weren't an option.

I've used ```pg_dump``` quite a bit for backing up databases and exporting tables but in all of those circumstances I wanted
to recreate the table, all of its permissions and load all of the data.  Of course in the Django world, I expect Django to create all of
the tables and handle permissions and constraints.  I had never used ```pg_dump``` to simply dump a table's data and
I didn't know how it would handle things like foreign key constraints that Django's ```dumpdata``` already abstracts for me.

Turns out ```pg_dump``` can handle it all.  After a couple of attempts, here is what I came up with:

{% highlight python %}
pg_dump <db_name> -t <table_name> -O -a --disable-triggers > fixture.sql
{% endhighlight %}

An explanation on the options:

``-t`` Specifies the table name.  If you are using [abstract classes](https://docs.djangoproject.com/en/dev/topics/db/models/#abstract-base-classes), be sure
you fully understand how your models are represented in the database and include the base tables that your objects may be stored on.

**Note**: You can provide multiple tables in a single dump by adding additional ```-t``` parameters.

``-O`` No owner.  This gives ownership of all of the objects to the user executing the ```psql``` import.

``-a`` Dumps only the data.

``--disable-triggers`` This handy feature will temporarily disable triggers on the target tables while your data is being loaded,
once the import is done, the triggers will be re-enabled.  This is necessary if you have data sets that have foreign key
constraints either through abstract classes or through the various relationships that Django supports.

To load the fixture into your application execute:

{% highlight python %}
psql <db_name> < fixture.sql
{% endhighlight %}