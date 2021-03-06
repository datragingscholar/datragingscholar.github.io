---
title        : "2 Eggs versus 100 Floor"
date         : "2011-06-29"
tags         : [Algorithm, Python]
---

I saw this problem posted on Facebook today, and find it really intriguing and
quite enjoyable to solve.

The problem is described as follow:

> * You are given 2 eggs.
> * You have access to a 100-storey building.
> * Eggs can be very hard or very fragile means it may break if dropped from the first floor or may not even break if dropped from 100 th floor.Both eggs are identical.
> * You need to figure out the highest floor of a 100-storey building an egg can be dropped without breaking.
> * Now the question is how many drops you need to make. You are allowed to break 2 eggs in the process

*The following methods, codes, and algorithms are all from a pure computer
  student's point of view, I know some people can directly solve the problem
  without writing any code, but that's no fun, is it?*

### Understand the question
Let’s take a rough look at the question first. Given that these two eggs are identical, the hardness of these two eggs are equally the same, which means that

```
hardness(egg1) = hardness(egg2)

highest_breaking_floor(egg1) = highest_breaking_floor(egg2)
```

I'm quite tempted to use binary search, drop the first egg at 50th floor, we
will then have two possible consequences

> 1. Egg1 breaks
>
> This means the highest floor the egg doesn’t break is less than 50, or 1-49th.
>
> 2. Egg1 did not break
>
> And this means the highest floor the egg can bear to be dropped down from is greater than 50, 51-100th.

This seems to be a very straight forward solution to find out the answer, either
one of these two situations will require a linear search to find out the real answer. This is a general solution to solve the problem, but the performance of total tries are questionable.

## The General Solution
In order to better understand how this approach works, let's look at the
following pseudocode and flowchart.

```
DROP egg1 AT 50th FLOOR

IF egg1.break == TRUE:
    FOR answer=1, answer<=49,answer++
        DROP egg2 AT answer FLOOR
        IF egg2.break == TRUE:
            PRINT 'The Answer IS:', answer-1
            BREAK

ELSE:
    FOR answer=51, answer<=100,answer++
        DROP egg1 AT answer FLOOR
        IF egg1.break == TRUE:
            PRINT 'The Answer IS:', answer-1
            BREAK
```

![General Solution Flowchart](/assets/images/2-Eggs-versus-100-Floor/flowchart.png)

Well, let's admit it, this is, as the name suggests, a very basic and general
solution. But before we try to figure out better algorithms, let's look at this
general solution a little bit closer.

We all know that under most context, a binary search is almost definitely better
than a linear search, then why don't we keep performing it and instead utilized
linear search after the first try?

Imagine if we broke the first egg at 50th floor, and wants to continue binary
search, then the next floor to try would be 25th floor. Now if the second egg
breaks at 25th floor, or any other subsequent binary search tries, there would
be no egg left to find out the exact answer.

However if the first egg does not break when dropped from 50th floor, we now
still have 2 eggs to spare, which would allow us to perform more binary searches
through 51-100th floor until we have only 1 egg left. So in short, we can only
perform binary search when we have both eggs intact, and linear search when
we have only 1 of them left. Pretty straighforward.

## More binary search!

Instead of performing 49 linear search plus the initial binary search, scoring a
horrific 50 tries at worst circumstance, let's find out what is the optimal
number of tries if the first egg manages to survive every binary search
drops.

| Count | Try At | Binary Search Start Value | Binary Search End Value | Number of Search Values |
|-------|--------|---------------------------|-------------------------|-------------------------|
| 1     | 50     | 51                        | 100                     | 50                      |
| 2     | 75     | 76                        | 100                     | 25                      |
| 3     | 88     | 87                        | 100                     | 13                      |
| 4     | 94     | 93                        | 100                     | 7                       |
| 5     | 97     | 96                        | 100                     | 4                       |
| 6     | 99     | 98                        | 100                     | 2                       |

The table above illustrates a typical binary search, we always halve the
*Number of Search Values* and add it to the last *Try At* to get the next binary
search offset.

It is clear that the minimal number of tries to get the answer would be 6, we
drop the egg at 99th floor if it manages to survive all previous attempts, the
answer is 98 if it breaks, and 100 if it doesn't.

Now we know the range of tries to find the answer is 6 to 50, it is time to
optimize our approach.

## Optimization

Every optimization starts with a question, are there room for optimization?
Fortunately, answering this question often brings us right to the optimized
solution, if there is one.

Let's assume the optimized maximum number of tries to solve this problew is *X*.
We already know that the worst scenario only happen when first binary
search resulted in a broken egg, in that case, we have to linear search from *1*
to *X-1*, totalling X-1-1+1 linear searches and 1 already performed binary
search.

This means *X* is not only the max number of tries, but also the first binary
search index, or offset if you will. Let's illustrate this with a table.

| Try At (Binary Search Offset) | Linear Start | Linear End | Linear Searches         | Binary Search Performed | Total Tries |
|-------------------------------|--------------|------------|-------------------------|-------------------------|-------------|
| X                             | 1            | X-1        | (X-1-1)+1 = X-1         | 1                       | X-1+1 = X   |
| ???                           | X+1          | ???-1      | (???-1)-(X+1)+1 = *X-2* | 2                       | X-2+2 = X   |

It is obvious for the first try, since at this point we only care about maximum
tries, the scenario would be we drop the first egg at Xth floor, it breaks, we
then perform linear search from 1st floor to X-1th floor, resulting (X-1)-1+1 =
X-1 linear searches, adding the first binary seach try we performed to it, we
get X-1+1 = X tries.

Perfect, it meets our assumption that the maximum tries needed to figure out the
answer is X. But at which floor should us perform the second binary search try
if the egg survived?

Well, we can't halve the number of remaining search values like we did
previously, that simply breaks our assumption of getting the answer within *X*
tries.

An easier way to understand it is, let's assume the second binary search offset
is ???, the linear start would be X+1 since all previous values are ruled out
given the first egg survived, the linear end would be ???-1, then total number
of linear searches needed to perform if the egg broke at the secound try would
be *(???-1)-(X+1)+1*, and that value *must* equal to *X-2* since only then would
total tries equal to *X*. Thus,

> *(???-1)-(X+1)+1 = X-2*
>
> *??? = 2X-1*

Subsequential offsets can all be calculated similarly.

| Try At (Binary Search Offset) | Linear Start  | Linear End | Linear Searches          | Binary Search Performed | Total Tries |
|-------------------------------|---------------|------------|--------------------------|-------------------------|-------------|
| X                             | 1             | X-1        | (X-1-1)+1 = X-1          | 1                       | X-1+1 = X   |
| 2X-1                          | X+1           | (2X-1)-1   | (2X-1-1)-(X+1)+1 = *X-2* | 2                       | X-2+2 = X   |
| 3X-3                          | (2X-1)+1 = 2X | (3X-3)-1   | (3X-3-1)-2X = X-3        | 3                       | X-3+3 = X   |
| ...                           | ...           | ...        | ...                      | ...                     | ...         |
|                               |               |            |                          |                         |             |

It is now obvious that the second binary search offset is *X+(X-1)* and the
third *X+(X-1)+(X-2)*, which allows us to deduce the last try at should be
*X+(X-1)+(X-2)+...+[X-(X-2)]+1*.

| Try At (Binary Search Offset) | Linear Start  | Linear End | Linear Searches          | Binary Search Performed | Total Tries |
|-------------------------------|---------------|------------|--------------------------|-------------------------|-------------|
| X                             | 1             | X-1        | (X-1-1)+1 = X-1          | 1                       | X-1+1 = X   |
| X+(X-1)                       | X+1           | (2X-1)-1   | (2X-1-1)-(X+1)+1 = *X-2* | 2                       | X-2+2 = X   |
| X+(X-1)+(X-2)                 | (2X-1)+1 = 2X | (3X-3)-1   | (3X-3-1)-2X = X-3        | 3                       | X-3+3 = X   |
| ...                           | ...           | ...        | ...                      | ...                     | ...         |
| X+(X-1)+(X-2)+...+[X-(X-2)]+1 | N/A           | N/A        | 0                        | X                       | X           |

The last binary search we perform should not require any further linear search,
thus resulting in *X* total binary searches, and consequentially *X* total
tries.

In order to fulfil the assumption that the last offset should require no further
linear search, which in other words, means we should cover all floors with the
last binary search offset, we can conclude that

> *X+(X-1)+(X-2)+...+[X-(X-2)]+1 >= 100*.
>
> Equally
>
> *(X+1)X/2 >= 100*
>
> *X = 14*

## Sample Code

``` python
#!/usr/bin/python
####Dummy Optimized General Solution for 2 Eggs VS 100th Floor###

import random

total_floor = 100
total_egg = 2
total_try = 14
count_try = 0

max_floor = random.randint(1, 100)

def main():

    print '---===2 Eggs VS 100th Floor===---'

    print '+Type 'exit' to quit the program!'
    start = raw_input('+Start? [Press Enter]')
    if start == 'exit':
        exit()

    last_try_at = 0

    while total_egg == 2:
        #Which means we can still use binary search until we break one egg.
        try_at = last_try_at + total_try - count_try

        print '->[Debug]<!!!-Stage 1-!!!> --> Performing Binary Serach...'
        print '->[Debug]count_try = ', count_try
        print '->[Debug]try_at = ', try_at
        print '->[Debug]last_try_at = ', last_try_at

        if drop(try_at):
            if try_at == 99:
                #This is the extreme case where the binary search was performed at 99th floor, and
                #it breaks, which means 98 is the answer in this context.
                print '->[Debug]The asnwer is 98!'
                print '->[Debug]Total Number of Tries: ', count_try
                exit()
            linear_start = last_try_at + 1
            linear_end = try_at-1
            print '->[Debug]Egg1 Broke!'
            print '->[Debug]Correct answer is in ', linear_start, 'to', linear_end
            print '->[Debug]<!!!-Stage 2-!!!> --> Performing Linear Search...'
            for i in range (linear_start, linear_end+1):
                if drop(i):
                    print '->[Debug]Egg broke at', i
                    print '->[Debug]The answer is', i-1
                    print '->[Debug]Total Number of Tries: ', count_try
                    exit()
                else:
                    print '->[Debug]Egg did not break at', i

            print '->[Debug]Egg survived all linear searches.'
            print '->[Debug]The answer is', linear_end
            print '->[Debug]Total Number of Tries: ', count_try

            break
        else:
            if try_at == 99:
                #This is the extreme case where the binary search was performed at 99th floor, and
                #it didn't break, which means 100 is the answer in this context.
                print '->[Debug]The asnwer is 100!'
                print '->[Debug]Total Number of Tries: ', count_try
                exit()
            last_try_at = try_at
            print '->[Debug]Egg1 did not break at ', try_at
            print '->[Debug]Continue binary search...'
            continue

        break

def drop(floor):
    global total_egg
    global count_try

    if total_egg <= 0:
        print '->No egg available for testing, program failed!'
        exit()
    elif floor > max_floor: #Egg is gonna break
        total_egg -= 1
        print '->Drop egg at ', floor, 'floor: [Broken!]', total_egg, 'egg left!'
        count_try += 1
        return 1;
    else:
        print '->Drop egg at ', floor, 'floor: [Did not break!]', total_egg, 'egg left!'
        count_try += 1
        return 0;
main()
```

Run this piece of code, you'll get the number of total tries at the end of each
run. To simulate the one of the worst scenario, simply change *max_floor* to 13.

> EDIT: I find myself extremely lack of mathematical thinking at the time of
> writing this post. However, it is fun to observe your though process many
> years ago, plus I did manage to get an answer :-D
