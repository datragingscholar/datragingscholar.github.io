---
title        : "FreeBSD Rootkit Design Howtos - Introduction"
date         : "2011-04-01 13:15:00"
tags         : [FreeBSD, Rootkit, C++]
---

As the title suggests, this is a howto tutorial on FreeBSD kernel rootkit
design. It was prepared as notes as I worked on my college final year project
while reading Designing BSD Rootkits: An Introduction to Kernel Hacking, where I
got most of the information I needed. I then posted these notes on my blog
hoping it could serve as a quick introduction to general kernel hacking. Those
notes were filled with unorganized brainstorming and unnecessary technical
details that even I myself couldn't bear reading a page without drinking way too
much coffee. It clearly failed it's purpose.

It took me a year to realize that it's a shame to give up all that hard work. I
rewrote the whole thing to make them more like tutorials other than academic
class notes.

I actually wanted to write this howto together with my college final year
project synchronously, but sadly things didn't work out the way I expected.
It took me almost a year to reach this point of rewriting most of them to make
them more like tutorials other than academic class notes.


Anyway, here I am, writing this intro again, hoping that I can finally finish it
this time.

FreeBSD Kernel Rootkit Design and Defense Techniques is the title of my college
final year project, I chose this topic because my lecturer who I consulted with
thinks this one can help me at applying master degree in malicious software
analysis.

I myself have no interest in pursuing master degree in malicious software
analysis, but it sure sounds cool to me to create my very own rootkit,
especially on my favorite platform, FreeBSD.

I still remember the lecturer asked me why I wanna make it on FreeBSD instead of
Linux, well, there are two reasons.

* I'm more familiar with FreeBSD than Linux
* I don't see any recent and public accessible rootkit for FreeBSD

Furthermore, there is another secret weapon to make my life even easier,
Designing BSD Rootkits: An Introduction to Kernel Hacking by Joseph Kong. I
highly recommend you guys to grab a copy of this book. It's good, and it's the
only book on FreeBSD kernel rootkit design as far as I know, so you kinda have
no choice here.

There's another nice book to start with, The Design and Implementation of the
FreeBSD Operating System By Marshall Kirk McKusick and George V. Neville-Neil.
As the book name suggests, it can give you a detailed understanding of FreeBSD
kernel. Get a copy of this book, read the first few chapters to get a rough
understanding and use it as a reference book later on when you work on specific
modules of the kernel.

## What to Expect

Designing BSD Rootkits: An Introduction to Kernel Hacking by Joseph Kong is
somehow sufficient for lots of people to kickstart their own rootkit, but you
have to be familiar with modern system kernels and have some experiences in
working in C.

However, I find this book a little bit rushy, what the author did was to drag
you through lots of kernel terminologies and then give you lots of code snippets
that, I myself at least, couldn't understand until I read them for multiple
times.

The Design and Implementation of the FreeBSD Operating System By Marshall Kirk
McKusick and George V. Neville-Neil on the other hand, discusses overall FreeBSD
kernel implementation and as well as each major kernel modules in detail, but
the only drawback is that it doesn't talk much about rootkit techniques, it only
concentrates on normal kernel development.


So that's why I wrote this series on FreeBSD Kernel Rootkit Design, it combines
what I have learned, and what I think is useful from all those books and online
resources.
I also improved some of the examples from Joseph Kong's book, filled in all
related information which were omitted by the author.

Consider this as a tutorial series. If you are looking for a quick and
comprehensive from novice to advanced novice guide on writing FreeBSD Kernel
Rookits, then congratulations, this is just right for you.
