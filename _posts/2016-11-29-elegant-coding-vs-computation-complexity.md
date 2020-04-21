---
layout: post
title: "Elegant coding Vs. computational complexity: the truth"
date: 2016-11-29
description: Computational load factor of function, a python example
img: big-o-notation.jpg
---
As developer, is very frequent that you could incur into the most 
famous pursue of the glory: **write elegant and beautiful code**.

But what if elegant code is not performant code at all?

> Let's see two algorithms that actually achieve the same result: 
>you have a list of integers, and for each index you want to find 
>the product of  every integer except the integer at that index 
>(not using division!)

Here the elegant version:

{% highlight python %}
def get_products_of_all_ints_except_at_index__quadratic(l):
    len_l = len(l)
    products = [1] * len_l
    for i in range(len_l):
        for j in range(len_l):
            if j == i:
                continue
            products[i] *= l[j]
    return products
{% endhighlight %}

As you can guess reading the code, or simply the name of the function, the code above 
has a **quadratic** `O(n^2)` complexity. 
In fact, you can understand simply standing in front of the nested-loop.

> Is that elegant? Yes, it is.

Now let's see the ugly version:

{% highlight python %}
def get_products_of_all_ints_except_at_index__linear(l):
    len_l = len(l)
    products = [1] * len_l
    products_after, products_before = [1] * len_l, [1] * len_l
    for i in range(len_l - 1, -1, -1):
        if i == len_l - 1:
            continue
        products_after[i] = l[i + 1] * products_after[i + 1]
    for i in range(0, len_l, 1):
        if i == 0:
            continue
        products_before[i] = l[i - 1] * products_before[i - 1]
    for i in range(len_l):
        products[i] = products_before[i] * products_after[i]
    return products
{% endhighlight %}

As you can guess reading the code, or simply the name of the function, 
the code above has a **linear** `O(3n) == O(n)` complexity. 
In fact, you can understand simply counting the 3 loop that aren't nested each-other.

> Is that ugly? Yes, it is.

_but.._

{% highlight console %}
100 repetitions: "O(n^2)" version: 10.9088919163 secs
100 repetitions: "O(3n) == O(n)" version: 0.08582782745 secs
{% endhighlight %}

**Lesson is: sometimes ugly (but performant!) is better :)**

![The Good, the Bad, the Ugly](/assets/img/good-bad-ugly.jpg)
