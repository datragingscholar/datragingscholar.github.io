---
title        : "Largest Prime Factor"
date         : "2013-02-02"
tags         : [Algorithm, Python]
---

> Third question from Project Euler, finding the largest factor of a prime
> number.

Great, I've actually been researching in prime numebrs recently. Got this crazy
idea, or a hypotheses few monthes ago, but never had the time to prove it. I spent
days trying to figure out better way to generate prime numbers, since my crazy
theory involves large primes, I mean, really large ones... Anyway, turns out
it's indeed just a crazy thought, not even worth mentioning.

Get back to the topic. [Problem 3](http://projecteuler.net/problem=3) asks to
find the lagest prime factors of the number 600851475143. Not a large number to
be frank. So I'll stick simplest factorization algorithm for this particular
question.

I guess I'll write a review on more complex factorization algorithms, those that
are used to factor large prime numbers in number theory and cryptograhy. Will
get back and put the link here if I ever decide to write it.

``` python
def prime_factorization(n):
    factors = []
    x = 2
    while x <= n:
        if n % x == 0:
            n /= x
            factors.append(x)
        else:
            x += 1
    return factors
```

This is like a brute-force algorithm, put x in the factors list if n is
divisible by x, then divide n by x before next iteration. If n cannot be divided
by x, then increase the value of x by 1.

Then I can just do this to get the result

``` python
print "The answer is: " + str(max(prime_factorization(600851475143)))
```

And the answer is **6857**.

Few things to keep in mind

1. Except for prime numbers, all whole numbers can be broken down into prime factors

2. There's no need to check whether x is a prime number, it is guaranteed to be prime

To explain this, take a look at the example below.

``` python
>>> prime_factorization(100)
[2, 2, 5, 5]
```

Of course, 100 can be factored into non-prime numbers like 4, 10, 20, 25, 50.
But as I have explained in (1), all these non-prime numbers can be divided into
prime number factors.

> 4 = 2 * 2
>
> 10 = 2 * 5
>
> 20 = 2 * 2 * 5

And so on. The fact that the program starts with smallest prime numbers (2, 3,
5, 7...) prevents non-prime factors.

Lastly, note that it's x <= n
