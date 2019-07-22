---
title        : "FreeBSD Rootkit Design Howtos - 2 - System Call First Kernel Service Module"
date         : "2012-06-09"
tags         : [FreeBSD, Rootkit, C++]
---

So, this is the second tutorial on FreeBSD Kernel Rootkit Design. I hope you
have worked through the previous one before you continue, which is obviously a
prerequisite.


What we’ve discussed are that KLD is the way we interact with kernel, and how to
declare a module by having module name, module data (consists of official name
and event handler function), sub, and order. And we agree on the
not-completely-true assumption that every kernel module should have an event
handler function which deals with event type such as MOD_LOAD, MOD_UNLOAD, and
so on. If any of these terms sounds strange to you, I encourage you to go back
and review [FreeBSD Kernel Rootkit Design Howtos - 1 - KLD First Kernel Loadable
Module]({% post_url 2012-06-08-freebsd-rootkit-design-howtos-1-kld-first-kernel-loadable-module %})

## The System Call Module

Today we're gonna talk about the *system call module*, which is a little bit
different compared to the general module we've discussed previously.

A general module which we’ve talked about in the last session performs
programmed actions only when certain actions take place, such as when it loads,
unloads, shutdown, and etc.

A *system call module* on the other hand, is basically as same as the general KLD
module, except that it installs itself as a *kernel service request*, and then
*listen to certain signals* to perform programmed actions accordingly.

Such functions can be considered as kinda of a bridge between the kernel space
and the user space, which enables the ability for its users to send signals to
the kernel and make it react accordingly.

What makes this system call module different is that, instead of printing
messages every time we load or unload the module, we're gonna make it print
messages every time we send command to it.

Here in this session, we’re gonna talk about the system call module, its
structure, its declaration routine, and finally write our first system call
module along with a tiny client application to send command to it.

## The System Call Function

A *system call function* is a function defined in the system call module, which
contains a list of actions to be taken every time it receives a system call.

It's similar to the *module event handler* except that we have control over what
command to receive and what actions to perform.

The prototype of system call function is defined in sys/sysent.h and is shown
below.

``` c++
FILE:/usr/src/sys/sys/sysent.h

typedef int     sy_call_t(struct thread *, void *);
```

The struct `struct thread *` points to the current running thread, which you don’t
have to care about it at this stage. The `void * points` to the structure of
**system call’s arguments**.

Compare to the general KLD module, the system call module can receive multiple
arguments instead of limited and pre-defined ones. So it is your responsibility
to define the arguments that the system call module needs to deal with.

Since the **system call's arguments** are wrapped in a struct, so we can define it
like this:

``` c++
struct sc_example_args {
    char *str;
};
```

Having the **system call's arguments** `struct` successfully defined, we can now
declare our **system call function** like this:

``` c++
static int sc_example(struct thread *td, void *syscall_args) {
    struct sc_example_args *uap;
    uap = (struct sc_example_args *)syscall_args;
    printf("%s\n", uap->str);
    return(0);
}
```

The first line is obvious the declaration of the system call function, note that
we can receive all arguments via `void *syscall_args` inside the function.

Let's now take a look at what happens inside the function, we firstly
initialized (It's not the precise term, but it helps) a local variable `*uap`
using our defined sc_example_args structure.

And then convert the incoming arguments from `*syscall_args` to match our
standard, the sc_examples_args structure, and let the `*uap` pointer points to
it. The simple version of this, is we receive arguments from `*syscall_args` and
save it in `*uap` with the `sc_example_args` format.

Now we can do whatever we want with the arguments received, such as print out
the string like this:  `printf("%s\n", uap->str);`

Looks we have successfully declared a system call function, but we actually just
made a huge mistake.

What I mean by mistake is that the code can still be compiled and executed, but
we did it in a very bad manner. You see that modern operating systems segregate
it's memory areas into user space and kernel space, code running in each section
don't directly access each other's resources. The way we assign a user space
structure pointer to a kernel space local variable (`uap = (struct
sc_example_args *)syscall_args;`) is unsafe and not recommended.

Here’s a quote from Designing BSD Rootkits: An Introduction to Kernel Hacking
that explains a little bit about kernel space and user space.

> FreeBSD segregates its virtual memory into two parts: user space and kernel
> space. User space is where all user-mode applications run, while kernel space
> is where the kernel and kernel extensions (i.e., LKMs) run. Code running in
> user space cannot access kernel space directly (but code running in kernel
> space can access user space). To access kernel space from user space, an
> application issues a system call.

Let's move on for now and get back to kernel/user space transition in details
later.

## The sysent Structure

Still remember the general module declaration macro in the previous session?
Well the system call modules need to register themselves by calling a macro as
well, but we have to define a **sysent structure** first and then pass it to the
declaration macro.

The sysent structure is similar to the moduledata that we’ve discussed in the
last session, it contains the basic information about the system call. So that
once we register a system call module with sysent structure, the operating
system will know where and how to quickly fire it.

The FreeBSD system actually maintains **a table of sysent structures** of all system
call modules that are currently loaded in the running kernel, thus every system
call module has to provide its sysent structure during initialization to
register itself with the **sysent table**.

So be sure that you understand how sysent structure differs from sysent table
before we take a look at its definition in sys/sysent.h

``` c++
FILE:/usr/src/sys/sys/sysent.h

struct sysent {                 /* system call table */
        int     sy_narg;        /* number of arguments */
        sy_call_t *sy_call;     /* implementing function */
        au_event_t sy_auevent;  /* audit event associated with syscall */
        systrace_args_func_t sy_systrace_args_func;
                                /* optional argument conversion function. */
        u_int32_t sy_entry;     /* DTrace entry ID for systrace. */
        u_int32_t sy_return;    /* DTrace return ID for systrace. */
        u_int32_t sy_flags;     /* General flags for system calls. */
        u_int32_t sy_thrcnt;
};
…
extern struct sysent sysent[];
```

I guess the comments in above code explain exactly what you need to know. Note
that normally we just need to specify `sy_narg` and `*sy_call` for this to work.

Now we can extend our previous example code as following:

``` c++
struct sc_example_args {
    char *str;
};

static int sc_example(struct thread *td, void *syscall_args) {
    struct sc_example_args *uap;
    uap = (struct sc_example_args *)syscall_args;
    printf("%s\n", uap->str);
    return(0);
}

static struct sysent sc_example_sysent = {
    1,              /* number of arguments */
    sc_example      /* implementing function */
};
```

## The Offset Value

Same as the system call function and the sysent structure, the **offset value** is
another parameter you need to set and pass to the system call module declaration
macro. Basically, the offset value is the system call module’s number, which
will be used by the system to refer to its sysent structure in the sysent table.

It should be an unique integer, and should be explicitly declared in a system
call’s declaration macro. It is considered as a good practice to not to assign
fixed numbers to dynamic system call modules. Instead, we can ask the system to
dynamically assign an unused offset number for our system call module by doing
this:

``` c++
static int offset = NO_SYSCALL;
```

`NO_SYSCALL` is a constant, meaning the next available slots offset in sysent
table.

Just in case if you are interested, the value for NO_SYSCALL is -1 as shown below:

``` c++
FILE:/usr/src/sys/sys/sysent.h

#define NO_SYSCALL (-1)
```

Some of the pre-defined system call offsets are listed in the
/sys/kern/syscalls.master file, here's some allocations:

| Offset Range | Comment                                                                   |
|--------------|---------------------------------------------------------------------------|
| 0-150        | Reserved/unimplemented system calls. For use in future Berkeley releases. |
| 151-180      | Reserved for vendor-specific system calls                                 |
| 181-199      | Used by/reserved for BSD                                                  |
| 210-219      | Reserved for loadable syscalls                                            |
| 220-249      | Were introduced with NetBSD/4.4Lite-2                                     |
| 250-299      | Initially used in OpenBSD                                                 |
| 300-531      | Syscall numbers for FreeBSD                                               |

## The SYSCALL_MODULE Macro

I said at the beginning of this tutorial that we are gonna need to call a macro
to declare a system call module, but we needed to know few other things first.
We talked about system call function, we talked about sysent structure and we
talked about the offset value.

We need these declared first and only then we can call the **SYSCALL_MODULE
macro**. It is defined in sys/sysent.h as following


``` c++
FILE:/usr/src/sys/sys/sysent.h

#define SYSCALL_MODULE(name, offset, new_sysent, evh, arg)
```

Different from the **DECLARE_MODULE** macro which requires four parameters:
**name**, **data**, **sub**, and **order**, the **SYSCALL_MODULE** requires five
parameter to be passed, which are:

### name

> This specifies the generic module name, which is passed as a character string.

### offset

> This specifies the system call’s offset value, which is passed as an integer
> pointer.

### new_sysent

> This specifies the completed sysent structure, which is passed as a struct
> sysent pointer.

### evh

> This specifies the event handler function.

### arg

> This specifies the arguments to be passed to the event handler function. For
> our purposes, we’ll always set this parameter to NULL.

Great, we can now further extend our previous example code as following

``` c++
struct sc_example_args {
    char *str;
};

static int sc_example(struct thread *td, void *syscall_args) {
    struct sc_example_args *uap;
    uap = (struct sc_example_args *)syscall_args;
    printf("%s\n", uap->str);
    return(0);
}

static struct sysent sc_example_sysent = {
    1,              /* number of arguments */
    sc_example      /* implementing function */
};

static int offset = NO_SYSCALL;

SYSCALL_MODULE(sc_example, &offset, &sc_example_sysent, evh, NULL);
```

Don't worry about the event handler function `evh`, it will be exactly same as
general module's event handler function. You'll see that in the complete example
soon, for now let's sum things up first.

## Summary

* System Call Module is another type of kernel loadable module

* It installs itself in the kernel space and perform programmed activities
  according to signals received from user space

* In order to declare a system call module, five parameters are required, name,
  offset, new_sysent, evh, and arg

  * name is the general module name

  * offset is the system call module’s offset number

      * It determines where to store the system call module's sysent structure
        in sysent[] table

  * new_sysent is a pointer to the system call module’s sysent structure

      * sysent structure is similar to moduledata

      * It contains basic information about the system call module including

          * Number of arguments it expects

          * And implementing function

      * The implementation function, also known as the system call function

      * It contains a list of actions to be taken every time it receives a particular signal

## Example Code

We can now write the our first system call module, take a look at the following
code, there are some comments there to help you understand. There is absolutely
nothing new except the event handler function, which in fact, isn't new to us as
well.

``` c++
/*
 * FILE: /root/rootkit/2.1/sc_example.c
 * Example 2.1
 * The First System Call Module
 * FreeBSD Rootkit Design Howtos @ hailang.im
*/
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/sysproto.h>

/* The system call's arguments. */
struct sc_example_args {
    char *str;
};

/* The system call function. */
static int sc_example(struct thread *td, void *syscall_args)
{
    struct sc_example_args *uap;
    uap = (struct sc_example_args *)syscall_args;

    printf("%s\n",uap->str);

    return (0);
}

/* The sysent for the new system call */
static struct sysent sc_example_sysent = {
    1,          /* number of arguments */
    sc_example  /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int load(struct module *module, int cmd, void *arg)
{
    int error = 0;

    switch(cmd) {
    case MOD_LOAD:
        uprintf("System call loaded at offset %d.\n", offset);
        break;
    case MOD_UNLOAD:
        uprintf("System call unloaded from offset %d.\n", offset);
        break;
    default:
        error = EOPNOTSUPP;
        break;
    }

    return(error);
}

/* Declare the System Call Module */
SYSCALL_MODULE(sc_example, &offset, &sc_example_sysent, load, NULL);
```

As usual, we need to have a makefile in the same directory as the source code
file.

``` makefile
KMOD=   sc_example
SRCS=   sc_example.c

.include <bsd.kmod.mk>
```

Now build your first system call module by using the make command in the same
directory.

```
myBSD# make
Warning: Object directory not changed from original /root/rootkit/2.1
@ -> /usr/src/sys
machine -> /usr/src/sys/amd64/include
x86 -> /usr/src/sys/x86/include
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mno-sse -mcmodel=kernel -mno-red-zone -mno-mmx -msoft-float  -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions  -Wmissing-include-dirs -fdiagnostics-show-option   -c sc_example.c
ld  -d -warn-common -r -d -o sc_example.ko sc_example.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk sc_example.ko  export_syms | xargs -J% objcopy % sc_example.ko
objcopy --strip-debug sc_example.ko

myBSD# ls
@       export_syms machine     makefile    sc_example.c    sc_example.ko   sc_example.o    x86
```

We have successfully compiled our first system call module and we got the
**sc_example.ko* file!

## Loading and Calling

Here’s the final step we need to take to make use of our first system call
module, the loading, and the calling. Let’s firstly try to load the module into
the running kernel, and then figure out how to issue the system call.

```
myBSD# kldload ./sc_example.ko
System call loaded at offset 210.
```

Thanks to our event handler, it prints out the system call number which is the
offset value of the system call module's sysent structure in sysent[] table.
You'll soon realize how important it is for us to issue a system call.

Now we have two ways to send command to our system call module, we can either
write a user space application, or type a simple command. I will talk about the
command first since the user space application will be covered in next session.

```
myBSD# kldload ./sc_example.ko
System call loaded at offset 210.
myBSD# perl -e '$str = "Hello kernel!\n I am here to dance with you!";' -e 'syscall(210, $str);'
myBSD# dmesg | tail -n 2
Hello kernel!
 I am here to dance with you!
```

Note that we explicitly specified the system call number in that perl command to
send our command to our system call module.

That's all for today, we made an upgraded version of fun kernel printing tookit
which can print whatever string you want it to. We'll talk about how to call a
system call module without knowing it's offset value in the next tutorial. See
you there.
