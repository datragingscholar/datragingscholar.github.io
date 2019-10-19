---
title        : "Clean Code Tips and Tricks: Introduction"
date         : "2014-04-11"
tags         : [Clean Code, PHP, Programming Guideline]
---

I recently organized a workshop for my team members to talk about Clean Code. It
has been a major issue plaguing our code and has been slowing us down by
generating tons of unreadable and unmaintainable code. So I took the oppoturnity
of the Chinese New Year holidays and delivered these three-days talk online.

I had great feedback from my team members and learned a few things myself by
preaching the Clean Code concept along with the S.O.L.I.D Principle. The
following weeks brought exiciting and notable results, the code quality improved
significantly just by following these simple tips and tricks. Better still, even
some skilled developers with many years of work experience showed signs of
improvement in code quality.

So I figured it might be of help to other developers if I posted these notes
online. I revised the notes I used to deliver the talks and incorporated some of
the new ideas I've learned during the process to share with you, though I'll
leave S.O.L.I.D principles for a later series that focuses on design principles.

This talk will be divided into five installments to try to cover one aspect of
clean code guidelines per post. The workshop lasted for three days and I've got
a lot more to share so it might be unpleasant to wade through the whole thing
one shot.

## What is this all about?

It's all about quality, and to be specific, code quality.

You see we programmers and software users have different definitions when it
comes to judging the quality of a piece of software, an app or a web site.
Customers expect good look and feel, simple and elegant interactive design and
rich functionalities, but from the point of view of those who *make* the
software, they *should* care about its internal quality.

I said we *should* because that is unfortunately and surprisingly not often the
case despite how natural and convincing that sounds. Yes you may hear some of
your colleagues talking about how messy and troublesome a codebase is to work
with, but when they program, they do it as if they were end users -- focusing
solely on delivering functionalities that satisfy customers' requirements.

Overtime they stop caring about internal quality and view the very software that
they are making themselves as a black box, they don't put enough thoughts on the
actual implementation and structure, they only expects their code to *just
work*.

Ironically, some others see the harm this does to the codebase, and they
understand good quality should be defined internally, but they choose not to
follow guidelines and best practices for reasons that are plainly wrong: they
think sacrificing quality is the necessary evil for faster delivery and lower
development costs. It is so horribly wrong I couldn't wait for you to read the
part I explain this in detail.

And of course there are others who only *followed the team styles* or *did it
the usual way* that they unknowingly contributed to poor code quality with bad
variable names, repetitive implementations, poor function structures, and the
like which to them, *is not a big deal, we did it like this only all the time*.

What characteristics I think a good quality codebase should have are as listed
below, people may come up with different list but I believe they essentially
mean the same thing.

- Readability
  - Good code should be easy to read and understand. They are not only written
    for computers to execute, but also (I'd even claim more importantly) for
    your co-workers to read and collaborate with.
- Maintainability
  - When fixing bugs or modifying existing functionalities, codebase with good
    maintainability not only reduces the workload, but also the chance of
    introducing new bugs.
- Extensibility
  - And when we add new features, we don't end up pulling our hairs off trying
    to figure out where and how to implement in a way that won't break existing
    features or introduce new problems.
- Testability
  - I'd say all code pieces are testable, but there are some that are
    particularly hard to test. Good quality code should be easy to write test
    for.
- Robustness
  - It's equivalent to endurance, it makes your code strong and mature in
    exception handling that would still fight like a tough soldier after years
    of production.

These characteristics are interconnected, you'd often times find that in order
to adopt one of them properly, you have to implement one of the others, it's also
true that when you work on one aspect, others would naturally improve as well.

We did have a brief discussion about S.O.L.I.D principle on day 3 when I first
presented to my team, I think it's great and should be adopted by every
developer. Following good design patterns and architectural principles would make a
profound qualitative change to maintainability, extensibility and testability,
and certainly attributes to the cleanness of your codebase as well, but those
are out of the scope of this post series. I'm planning on writing about patterns
and principles I've personally had experience with in the near future, stay
tuned if you are interested.

Though this series mainly focus on readability, working on it alone can already
have a powerful impact on your codebase, and not only do they become easier to
understand, but also more enjoyable to maintain, to hunt bugs and to write tests
for.

## But Why?

> "When it comes to writing code, an ounce of prevention is worth a pound of cure."

We all had some experiences that somewhat resemble this: whether you yourself
or one of your co-workers who wrote code carelessly and irresponsibly that led
to bugs, performance issues, and weird system behaviors nobody knew what caused
it and how to solve it. After days of debugging, you finally pinpointed that
piece of ~~shit~~, um I mean code, you found it utterly gibberish and unbearably
hard to understand. Then after pulling all your hairs off, you finally
understood the code and the cause of the issue, just to be confronted by the
harsh reality that it is nearly impossible to modify the original code without
editing ***too much*** or breaking ***too many*** existing functionalities. As the
same issue piles up and consumes nearly all your time, and at some stage along
this path, a hard decision would be made to ***rewrite*** the whole thing in hope
of solving those issues altogether, but this time you wouldn't even be surprised
to find out the ***refactored*** project would go through exactly the same set
of problems all over again.

It's a vicious cycle with no way but only one way out -- WRITE, CLEAN, CODE.

### It's Evil, But Certainly Not Necessary

People intentionally and irresponsibly write bad code justify their actions on a
wrong premise that this could save development time or cost, perhaps both. When
confronted with a tight schedule or budget, they instinctively deteriorate their
code quality assuming it is just a trade-off of quality for efficiency as if
software development was a one-time deal.

It is not.

When you take bug fixings, requirement changes and feature additions into
account, sacrificing code quality to save development time and cost is most
likely a dumb thing to do, it's as if you are intentionally saving long and
exhaustive hair-pulling sessions for later.

Saving time and money developing a piece of software but end up spending more
than you saved on extensions and maintenance due to bad code quality does not
sound like a good plan at all, it perhaps only make sense when the scope is only
to develop a MVP or prototype, and you are dead sure nobody needs to touch the
codebase ever again.

The reason I observed resistance to the idea of writing clean code from the
teams I've worked with is perhaps that it asks programmers to spend ***more*** time
writing good code when horrific bugs and new feature requests with undetermined
side effects are already taking too much of their time. Hence I hear this over
and over,

> ***Meh, I'm Just Too Busy***

It is more important to do the ***right stuff*** than doing the ***wrong
thing*** but being productive and busy as a bee. You are just being a really
naughty bee, now don't be a bad bee when asked to improve your work ethics by
saying, "but I'm already too busy polluting the honey with poison!"

Writing readable, robust and maintainable code is not a waste of time, it's what
you should do as a professional developer.

> “We spend most of our time staring into the abyss rather than power typing.”
> ~Douglas Crockford

Let's face the fact here, we are not like those dope hackers in the movies who
type like their lives depend on it in front of the computer. No, we spend most of
our time looking at ugly code, cursing the guy who committed it and sometimes
even later realized it was you who wrote it.

We spend so much time staring at code that in fact the time ratio of reading and
writing code is 10:1. For each hour of typing code, you spend 10 hours reading.

Writing beautiful, elegant, and understandable code is not altruism, it's to
relieve yourself from pulling all-nighters to catch up with schedule, and being
called in the middle of the night to be lashed about a bad bug.

Surprisingly only few people understands schedule delay and slow feature
delivery are not because your team members are typing too slow, on the contrary,
it's because you are writing too fast without caring about code quality and have
to spend tons of time wading through horrific death swamp just to extend a new
feature or fix a minor bug. It ***is*** lucrative to spend more time writing
better code, and less time reading bad ones.

> ***But we don't have the budget for it***

And some project owners feel reluctant towards clean code for the same reasons,
except that they are more concerned about monetary issues. Starting off a brand
new project can benefit from adopting clean code as it saves time and money by
reducing the costs on maintaining and extending the software, it is however less
straightforward in cases of existing projects, since again, we are asking for
more costs writing better code before we can observe notable costs reductions.

Managers and project owners assume good quality things cost more, it must be
logical to lower quality standards in order to reduce costs. It may be true from
the end user's point of view, they want a better looking software with more
functionalities, they often have to pay more.

However it is not necessarily true from the development team's point of view,
and sometimes I even observe the exact opposite -- better the quality of the
codebase, less costs it incurs comparing to competitors that offer same level of
end user experience.

It is true that in short term, we can cut some costs by sacrificing code
quality, but team members and managers often underestimate how quickly bad code
pieces would build up and slow the overall process.

High code quality leads to faster and less expensive delivery of new features
and improvements, which I'd say is more important for customers. I personally
would be more willing to pay for an app with less feature but more frequent
improvements than one with more features but slow in fixing bugs and responding
to new feature requests. And eventually this app with better internal structure
will catch up and have more features to offer thanks to its accumulating costs
reductions.

### It's Your Responsibility

As a professional software developer, your job is not just to implement required
functionalities, you have to deliver them with good practices and code quality.

It is your responsibility to write clean and quality code, not your clients',
not your colleagues', not your team leader's or your boss's, yours. A famous
analogy is that you don't tell your doctor to skip washing their hands because
it saves time. Client or manager telling you to be quick to deliver and catch up
with schedule are ***NOT*** suggesting lowering code quality.

Every job has their own work ethics, guidelines and procedures they follow to do
their jobs right, so should we. As Martin said in one of his talks, software
developers are becoming more and more influential to the physical world, if we
don't come up with some code of conduct or principles on our own, governments
would impose them on us when the day comes where a software bug could
potentially get people killed.

That day's nigh, clock's ticking :-D

## How to Clean Code

We'll be discussing tips, tricks and guidelines to write clean code in the later
installments of this blog series, but the general rule of thumb is just to be
responsible, polite and considerate when you write code, think about the
consequences of your approach, respect and reflect your team's style and
architectural design.

### Be Polite

Bad code is rude, and you don't want to be rude to your co-workers and the
future you.

At the dawn of Computer Science, programming was more of a method to talk to
computers, to let the computers know our intentions and do our deeds. But that
has changed since we invented high level progamming languages and introduced
teamwork to the making of software.

Now we must write for not just computers, but other humanzzz as well. Writing
code that is hard to read and maintain by the future you or your co-workers
brings much more trouble than compiling errors and runtime exceptions.

Programmers are more like writers, you should write as a writer, write stuff you
and your co-workers would enjoy reading and working on.

### Be Considerate

Computers don't care about your code quality, it does whatever you tell it to
do. But you should keep other people in mind when you code, think about what
approach and implementation they'd appreciate, and how you can help to make
their life (and yours) easier.

> ***You code is the best document if done right***

Your code is the best document if it's done right. No matter how ellaborate you
document your code, it won't be of any help if the code quality is horrible.

And everytime you update your codebase, corresponding changes should be
committed to the documents as well. If we don't even write good quality code, I
can't imagine how awful the document would look like.

My team maintains no document whatsoever, partly because it brings more overhead
than it brings good for a small team, but mostly because I believe
self-explanatory code invalidate the purpose of documentation.

However, I'm not saying documentless is the definite way to go, many teams still
maintain documentation or wiki about their codebase. But if you and your
co-workers can understand eachother's code without referring to comments or
documentation, then why not?

### Be Thorough

Unfortunately, modern high level languages are supposedly closer to human
languages, but the flow of expressions and the way we convey instructions are
far from what we are familiar with in our daily lives. A non-programmer may find
herself fully understand the literal meaning of a piece of code but couldn't
just grasp the true intention of it.

We have to admit that the way we are used to write code often isn't the best way
to present it to other people. Train a human resource employee to validate job
application forms is much easier than writing a piece of code to do it, we have
to consider all the possibilities and small steps, handle every subtle and even
hidden aspects of the matter and try to cover everything there is to consider.

We are good at being thorough to computers, now is time to do the same for your
co-workers. We should consider how we can present our thoughts in a way that is
more natural for another programmer to understand, maintain and extend.

### Leave It Better Than You Found It

So what if you are working on a gigantic ball of mud codebase and it seems too
late to adopt any of these tips?

Actually it's never too late to start writing clean code, and if you can
convince yourself and you co-workers to improve the dirty code everytime they
come across it, you'll eventually have a clean codebase.

Now, note that leave the code you come across better than you found it does not
necessarily mean rewriting or refactoring a whole class. Fix the variable names,
improve the structure of the class, divide functions that don't follow Single
Responsibility Principle into smaller ones, and implement meaningful method
names would be good enough to have notable effects in just a few weeks.

That's it! Thanks for reading. We talked about why we have to write clean code,
the benefits of adopting it and some common misconceptions and
misunderstandings. I hope you enjoyed it, looking forward to seeing you in the
next installment of Clean Code Tips and Tricks.
