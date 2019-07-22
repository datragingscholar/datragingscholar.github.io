---
title        : "Sum Square Difference"
date         : "2013-05-29"
tags         : [Algorithm, Python]
---

> The difference between the sum of the squares of the first ten natural numbers
> and the square of the sum is 3025 385 = 2640. Find the difference between the
> sum of the squares of the first one hundred natural numbers and the square of
> the sum.

Sixth question from [Project Euler](http://projecteuler.net/problem=6).

Sum Square Difference (SSD) is commonly used in image processing. For the sum of
squares of the first 100 natural numbers,

``` python
sumsqr = (1 ** 2 + 2 ** 2 + ... 100 **2)
```

And for the square of the sum,

``` python
sqrsum = (1 + 2 + ... + 100) ** 2
```

A simple loop will do the job.

``` python
sumsqr, sqrsum = 0

for i in ((x, x ** 2) for x in xrange(1, 101)):
    sqrsum += i[0]
    sumsqr += i[1]

print sqrsum ** 2 - sumsqr
#25164150
```

The answer is **25164150**.
