---
title        : "FreeBSD Rootkit Design Howtos - 3 - System Call First Kernel Service Application"
date         : "2011-04-06"
tags         : [FreeBSD, Rootkit, C++]
---

Welcome back to FreeBSD Kernel Rootkit Design Howtos, we’ll walk through all
necessary techniques you need to program your own BSD kernel rookit. Please be
sure you’ve read the previous guides before you proceed with this one.

## Review

Same as usual, let’s review what we’ve discussed in the [last session]({%
post_url
2011-04-04-freebsd-rootkit-design-howtos-2-system-call-first-kernel-service-module
%}). Basically a new kind of kernel module was introduced — The system call
module, it registers itself as a kernel service and wait for user to call it via
system calls.

**SYSCALL_MODULE** is the registration macro for system call modules to install
themselves into the running kernel, and it requires five parameters:

* **name** as a generic name for the system call module

* **offset** value which determines in which location in the sysent[] table to
  save our system call module’s sysent structure

* **new_sysent** pointer to a sysent_t structure that contains basic info about
  the system call module such as number of arguments and pointer to it’s
  implementation function

* **implementation function** defines activities you want your system call
  module to perform every time when it receives system call

In this session, we'll make a custom client application that calls our kernel
service module directly without knowing the module's offset value.

## The modfind Function

The **modfind** function is our key to solve the inflexibility issue, it is very
useful but a little bit confusing.

As the name implies, the modfind function helps us to find a specific module in
a running kernel by giving it the module name. It is sweet ’cause keeping track
of module names is much easier than a bunch of module offset values.

Now the confusing part is, opposite to what you may have guessed, the return
value of this function is **NOT** the system call module’s offset value, but it’s
**id**.

Let’s take a look at a sample piece of modfind function code.

``` c++
#include <sys/param.h>
#include <sys/module.h>

int modfind(const char *modname);
```

## The modstat Function

As we can only obtain the **id** of the system call module by calling
modfind function, so here's what really give us answers - the **modstat
function**.

``` c++
#include <sys/param.h>
#include <sys/module.h>

int modstat(int modid, struct module_stat *stat);
```

The `int modid` is where we should pass the result of modfind function, but we
won't get the offset value from the function's return value.

Instead, we have to construct a `module_stat` structure, and the modstat function
will save results to it. That is why we are passing a pointer to a module_stat
structure as the second parameter.

The module_stat structure is defined in sys/module.h as shown below

``` c++
FILE: /usr/src/sys/sys/module.h

struct module_stat {
        int             version;        /* set to sizeof(struct module_stat) */
        char            name[MAXMODNAME];
        int             refs;
        int             id;
        modspecific_t   data;
};

typedef union modspecific {
        int     intval;
        u_int   uintval;
        long    longval;
        u_long  ulongval;
} modspecific_t;
```

I know this looks messy, but luckily we don't need to deal with all of them.

The first part of the code is the definition of `module_stat` structure, and it
contains a `modspecific_t` union which is defined in the second part, our offset
value is stored in this union as `intval`.

Confusing right, you don't see any variable name here with offset value in it.
The good news is, that's all we have to learn today, we'll get to the example
code after the summary.

## Summary

We now know that having a native client to call system call modules is much
better than the perl command we used in last session. It is better because,

* We don't have to remember the offset value of the system call module to call
  it. This is especially useful when we have to maintain multiple system call
  modules

* The offset value of system call module changes every time we reload it. We'll
  still have our flexibility since we rely on module name which is not likely to
  be changed over time

The way how the client works is that we use modfind to find out the id of a
given module name, but sadly it’s not the module’s offset value, which we can’t
use to issue the system call.

So we have to utilize another function which is modstat that will give us back a
modspecific union, in which we can get the real offset value we want from the
intval integer.

Here's a figure to make your life miserable, you can thank me later. : ] Oh god,
why I'm so bad at drawing...

![Chart 3](/assets/images/freebsd-rootkit-design-howtos/chart3_1.png)

## Example Code

Now it’s time for us to see some examples. Let’s firstly start with the example
from Designing BSD Rootkits: An Introduction to Kernel Hacking with some
additional comments to help you understand the code.

``` c++
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>

int main(int argc, char *argv[]) //Main function of our client application
{
    if (argc != 2) { //Check the parameter passed to the application,
                     //print out usage help information
                     //if we got wrong number of parameters.
        printf("Usage:\n%s <string>\n", argv[0]);
        exit(0);
    }

    struct module_stat stat; //This is the module_stat structure
    int syscall_num; //We'll get the offset value and store it here.


    /* Determine sc_example's offset value. */
    stat.version = sizeof(stat); //set version to sizeof(struct module_stat)
    modstat(modfind("sc_example"), &stat);
    syscall_num = stat.data.intval;

    /* Call sc_example. */
    return(syscall(syscall_num, argv[1])); //Return the statues of the system call
}
```

Unlike kernel modules which we need a makefile to automate some extreme nasty
stuff, we don’t need one for this simple client side application. We can
directly compile the code like this,

```
myBSD# gcc -o client client.c
client.c: In function 'main':
client.c:12: warning: incompatible implicit declaration of built-in function 'exit'
myBSD# ls
client      client.c
```

Looks good so far, we’ve got the client executable file. Let’s load the previous
sc_example module and try to call it with our client application.

```
myBSD# kldload ./sc_example.ko
System call loaded at offset 210.
myBSD# ./client
Usage:
./client <string>
myBSD# ./client Hey\ Kernel!
Bad system call (core dumped)
```

We loaded the sc_example module at offset value of 210, we got a nice help
message when wrong number of parameter is specified, which is nice, but we end
up with a Bad system call error when we give it the right amount of parameters.

This is bad, because we’ve got nearly no clue of what had happened, and what
caused it. But just before we panic, let’s calm down and think about this error
message which is our only lead to the problem, which says Bad system call.

It is obvious that we were calling a bad kernel module, so bad that our little
client crashed. If we called a bad kernel module in our syscall function, then
that means the return value of modfind or modstat function is wrong, or maybe
both of them are wrong, who knows.

So I did a little bit debugging, and I soon realized that the return value of
modfind is -1, which is obviously bad.

``` c++
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>

int main(int argc, char *argv[]) //Main function of our client application
{
...
    printf("The return value of modfind is: %d\n", modfind("sc_example"));
...
}
```

The result says

```
myBSD# gcc -o client client.c
myBSD# ./client
The return value of modfind is: -1
```

Alright, ***-1***, it probably means fail to find this module, or given module was not
found. Given that the module name is the only parameter for the modfind
function, it is obvious that the module name is wrong.

This is interesting, we all know that the module name is right, so there’s
something wrong from the very beginning. The very source of a system call module
is of course it’s declaration macro, so let’s look at there and see if we can
get any luck.

``` c++
FILE:/usr/src/sys/sys/sysent.h
#define SYSCALL_MODULE(name, offset, new_sysent, evh, arg)      \
static struct syscall_module_data name##_syscall_mod = {        \
        evh, arg, offset, new_sysent, { 0, NULL, AUE_NULL }     \
};                                                              \
                                                                \
static moduledata_t name##_mod = {                              \
        "sys/" #name,                                           \
        syscall_module_handler,                                 \
        &name##_syscall_mod                                     \
};
```

Here we go, a ***module name prefix***! That means every kernel module
registered with SYSCALL_MODULE will be given a ***sys/*** prefix in front of
their names.

Now we know the root of the problem, let's modify our code and try again.

``` c++
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>

int main(int argc, char *argv[]) //Main function of our client application
{
    if (argc != 2) { //Check the parameter passed to the application,
                     //print out usage help information
                     //if we got wrong number of parameters.
        printf("Usage:\n%s <string>\n", argv[0]);
        exit(0);
    }

    struct module_stat stat; //We'll get the module statues and store it here.
    int syscall_num; //We'll get the offset value and store it here.


    /* Determine sc_example's offset value. */
    stat.version = sizeof(stat); //set version to sizeof(struct module_stat)
    modstat(modfind("sys/sc_example"), &stat); //With prefix this time
    syscall_num = stat.data.intval;

    /* Call sc_example. */
    return(syscall(syscall_num, argv[1])); //Return the statues of the system call
}
```

Everything is the same except `modstat(modfind("sys/sc_example"), &stat);`. Now
compile and run.

```
myBSD# gcc -o client client.c
client.c: In function 'main':
client.c:12: warning: incompatible implicit declaration of built-in function 'exit'
myBSD# ./client Hey\ Kernel!\ What\'s\ Up\?
myBSD# dmesg | tail -n1
Hey Kernel! What's Up?
```

Yada! Our little client is doing it's job and we've achieved our objectives by
calling a system call module without knowing it's offset value.

That will be all for today, thank you for reading this tutorial, like it and
share it to support my work, and I'll see you in the next tutorial!
