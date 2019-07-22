---
title        : "Largest Palindrome Product"
date         : "2013-02-17"
tags         : [Algorithm, Python]
---

> A palindromic number reads the same both ways. The largest palindrome made
> from the product of two 2-digit numbers is 9009 = 91*99.
>
> Find the largest palindrome made from the product of two 3-digit numbers.

4th question from [Project Euler](http://projecteuler.net/problem=4). Finiding
the largest palindromic number that is a product of two 3-digit numbers.

The game plan is to get all the product of 3-digit numbers, find which ones are
palindromic, and then print the largest one.

It is clear that the range of 3-digit number is 100 to 999.

*itertools.product* is used to generate pairs of 3-digit numbers.

``` python
from itertools import product
product(xrange(100, 999), repeat=2)
```

It is a short and clean substitute for nested loops.

``` python
product(A, B) #equivalent to
((x, y) for x in A for y in B)

product(A, repeat=2) #equivalent to
product(A, A)
```

For details, refer to [Python Docs](http://docs.python.org/2/library/itertools.html#itertools.product)

Finding if a given number is palindromic can be done by converting it into a
string and compare it with its reverse copy.

``` python
x = 110011
print str(x) == str(x)[::-1]
#True
```

Now we only need to put the results in a list and print the largest one using
`max()`.

``` python
from itertools import product
results = [ a*b for a,b in product(xrange(100,999), repeat=2) \
            if str(a*b) == str(a*b)[::-1] ]
print max(results)
#906609
```

So the answer is **906609**.
