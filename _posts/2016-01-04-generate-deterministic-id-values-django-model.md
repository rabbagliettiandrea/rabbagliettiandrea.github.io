---
layout: post
title: "Generate deterministic ID values in a Django Model"
date: 2016-01-04
description: Deterministic ID values: strange behavior or what
img: race-condition.jpg
---
OK, let's talk about a thing that **seems** trivial & obvious, but _unfortunately_ it is not.

Assume that we want that when we insert objects, those objects has to have an unique, 
progressive and dense integer as ID.

A thing like that:

{% highlight python %}
from django.db import models

class Customer(models.Model):
    name = models.CharField(max_length=255)

[...]

c1 = Customer.objects.create(name='c1')
c2 = Customer.objects.create(name='c2')
c3 = Customer.objects.create(name='c3')

[...]

c1.id  #: 1
c2.id  #: 2
c3.id  #: 3
{% endhighlight %}

Well, someone (_me too, though_) would say that is there also a field that does this behaviour, 
in Django framework: `models.AutoField`, that is the type of the implicit `id` attribute, 
default primary key of any Model.

## Wrong.

Turns out that `AutoField` doesn't do anything Django-side but instead asks 
(at the saving time) at the DBMS for a generated ID or something as reported also here.

The consequence is that what the DBMS may return **isn't guaranteed at all densely progressive**.

That is a DBMS related thing that has to do with concurrence and performance avoiding gapless sequence.

Clearly when an object is saved a race-condition is generated between the Django _agents_ 
(threads, different processes, different processes on different machines and so on), 
that possibly could want to save another object at the same time. Of course, Django 
(_and so the DBMS_) has to manage that sort of thing and **possibly** in a performant manner.

Hence, instead of locking anything the DBMS prefers to skip some number that have 
the risks (timely-related risks) of a race-conditions.

{% highlight python %}
c1 = Customer.objects.create(name='c1')  # t0 time
c2 = Customer.objects.create(name='c2')  # t1 time
c3 = Customer.objects.create(name='c3')  # t1` time

[...]

c1.id  #: 1
c2.id  #: 3
c3.id  #: 4
{% endhighlight %}

Clearly also in that scenario it **maybe** could want to skip 2 as number and take 3 and 4.

Don't misunderstand me, generally speaking this is a great choice, the world needs performances.

## But what if do you need a deterministic progressive IDs?

So I've solved that issue using an auxiliary model _Counter_ which holds, locks and increments an attribute.

{% highlight python %}
from django.db import models, transaction


class Counter(models.Model):
    model_name = models.CharField(max_length=255, unique=True, db_index=True)        
    n = models.PositiveIntegerField(default=1)
    
    
class Customer(models.Model):
    name = models.CharField(max_length=255)
    id_dense = models.PositiveIntegerField()
    
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        if self.id is None:
            with transaction.atomic():
                counter, _ = Counter.objects.select_for_update().get_or_create(model_name='Customer')
                self.id_dense = counter.n
                counter.n += 1
                counter.save()
            super().save(*args, **kwargs)
{% endhighlight %}

note that `select_for_update()` is what locks the row until the transaction doesn't terminate.

I think that the above is a good and simple way to achieve that sort of thing.

Hope this helps.
