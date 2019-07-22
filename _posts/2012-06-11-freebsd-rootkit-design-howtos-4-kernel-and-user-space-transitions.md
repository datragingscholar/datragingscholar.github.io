---
title        : "FreeBSD Rootkit Design Howtos - 4 - Kernel And User Space Transitions"
date         : "2012-06-11"
tags         : [FreeBSD, Rootkit, C++]
---

What's up geeks, it's good to see you again! This is the 4th tutorial on FreeBSD
Kernel Rootkit Design Howtos. Check back everyday for new tutorials and
examples.

## Review

Well, in the previous session, we discussed about an upgraded version of client
application to issue system calls. We used `int modfind(const char *modname);` to
get the id of a kernel module by specifying module name, and then we
successfully retrieved the module's offset value by calling  `int modstat(int
modid, struct module_stat *stat);` function. We learned that the offset value is
actually stored as `int intval;` in `typedef union modspecific` which is a sub-union
of `struct module_stat`.

I hope you still remember that we created our very first system call module that
expects a string argument to be sent from user land application, and it will
then print out the string received. You don't have to recall the whole process
since we'll redo that in this session, what you do need to recall is that we
made a *mistake*.

> modern operating systems segregate it’s memory areas into user space and
> kernel space, code running in each section don’t directly access each other’s
> resources. The way we assign a user space structure pointer to a kernel space
> local variable  (`uap = (struct sc_example_args *)syscall_args;`) is unsafe and
> not recommended.

Now it's time to deal with this problem and fix that example code.

## Kernel and User Space Transition

Before we start fixing our example code, I'd like to talk a little bit more
about ***kernel space*** and ***user space***.

We all know that modern operating systems segregate their virtual memory into
user space and kernel space, and commonly user-mode applications run in user
space, and system kernel runs in kernel space. Now, the question is why, why
such separation exists, what is its purpose.

Actually we can consider user space as isolated sand-boxes, it restricts user
land applications so that they don't mess up with each other's resources, and
most importantly, the kernel space.

As we know, the kernel is the core of any operating system that is in charge of
*hardware resource allocation, scheduling, I/O, process management*, and all sorts
of low level stuff. We want to maintain stability and security of the system, so
we really don't want poorly-designed user applications crashing the whole system
or malicious user application to modify the behavior of the kernel.

Although it it restricted, but it's possible for user space and kernel space to
access each other's virtual memory since such communications are necessary. We
just have to make sure that we do these in a way that the impact on system
stability and security are minimal.

* User space CANNOT access kernel space, if you have to, issue a system call
* Kernel space CAN access user space, because it literally owns everything of
  the system. However, it is recommended to copy memory area from user space to
  kernel space before access it, otherwise may result a fatal panic.

![Chart 4-1](/assets/images/freebsd-rootkit-design-howtos/chart4_1.png)

Look at the figure above, it simply says that user-mode applications can only
access kernel resources via system calls, and the system call module will return
a value or perform pre-defined actions. This is much safer than letting
user-mode applications to freely modify kernel resources or calling kernel
functions, either kernel resource changing or function calling are performed by
pre-programmed and well-designed code running in kernel, which are considered
safer and stabler.

The figure also says that kernel modules running in kernel space normally copy
user space resources back to it's own virtual memory area first, and then
perform pre-defined actions to either return a value to user space or modify the
resource content.

Copying user space resources back to kernel space before referring or modifying
is safer and recommended because, if we refer to a user space resource directly
from kernel space and it's swapped out or not faulted in yet, we'll trigger a
fatal panic.

Now let's figure out how to copy resources around virtual memory areas.

## The COPY(9)

Open your terminal and run the following command to see the manpage for all
***copy functions***

```
myBSD# man copy
```

***Copy functions*** are ***copy***, ***copyin***, ***copyout***, and
***copyinstr***. What they generally do is

> The copy functions are designed to copy contiguous data from one address to
> another. All but copystr() copy data from user-space to kernel-space or vice-versa.

I find these four function names a little bit confusing, so let me put a figure
here to help you remember their usages.

![Chart 4-2](/assets/images/freebsd-rootkit-design-howtos/chart4_2.png)

The following is a list of all 4 copy functions definition and usage

### copyin()

``` c++
int copyin(const void *uaddr, void *kaddr, size_t len);
```

It copies *len* bytes of data from the user-space address *uaddr* to the
kernel-space address *kaddr*.

### copyout()

``` c++
int copyout(const void *kaddr, void *uaddr, size_t len);
```

It copies *len* bytes of data from the kernel-space address *kaddr* to the user-space address *uaddr*.

### copyinstr()

``` c++
int copyinstr(const void *uaddr, void *kaddr, size_t len, size_t *done);
```

It copies a NUL-terminated string, at most *len* bytes long, from user-space
address *uaddr* to kernel-space address *kaddr*. The number of bytes actually
copied, including the terminating NUL, is returned in `*done` (if done is
non-NULL).

### copystr()

``` c++
int copystr(const void *kfaddr, void *kdaddr, size_t len, size_t *done);
```

Copies a NUL-terminated string, at most *len* bytes long, from kernel-space
address *kaddr* to kernel-space address *kaddr*. The number of bytes actually
copied, including the terminating NUL, is returned in `*done` (if done is
non-NULL).

## The Copyin Functions

From the perspective of a system call module, we need to copy memory from user
space to kernel space at most of the times. So that there are two copy functions
we can utilize to achieve this task, `copyin()` and `copyinstr()`.

It is necessary to talk about the difference between `copyin()` and `copyinstr()`
before we choose which one to use for our upgraded example system call module.

|                    | copyin()                                           | copyinstr()                                                                            |
|--------------------|----------------------------------------------------|----------------------------------------------------------------------------------------|
| Direction          | From user space to kernel space                    | From user space to kernel space                                                        |
| Resource Type      | Any user space memory address                      | NUL-terminated string (ends with \0)                                                   |
| Copy Length        | Exactly len bytes                                  | At most len bytes long                                                                 |
| Extra Argument     | NONE                                               | size_t *done stores the number of bytes actually copied, including the terminating NUL |
| Return Value       | 0 = success EFAULT = bad address                   | 0 = success EFAULT = bad address                                                       |
| Extra Return Value | ENAMETOOLONG = the string is longer than len bytes | ENAMETOOLONG = the string is longer than len bytes                                     |

As you can see from the table above, that these two functions perform basically
same actions, but fits under different scenarios. As we need to copy a string
from user space to kernel space in our example system call module, we are gonna
use `copyinstr()` this time.

## Example Code

Finally here comes our example code, just remember to include the two extra lib
files.

``` c++
/*
* FILE: /root/rootkit/4.1/safe_sc_example.c
* Example 4.1
* User and Kernel Space Transitions
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

    char kernstr[1024+1]; //This is the place holds a copy of the string in kernel space
    int err = 0; //Return stat
    size_t size = 0; //Size actually copied

    err = copyinstr(uap->str, &kernstr, 1024, &size);
    if (err == EFAULT)
        return(err);

    printf("Safer version output: %s\n",kernstr);
    return (0);
}

/* The sysent for the new system call */
static struct sysent sc_example_sysent = {
    1,  /* number of arguments */
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

SYSCALL_MODULE(sc_example, &offset, &sc_example_sysent, load, NULL);
```

Save the above code as safe_sc_example.c, and the following as makefile in the
same directory

``` makefile
KMOD=   safe_sc_example
SRCS=   safe_sc_example.c

.include <bsd.kmod.mk>
```

Now we can build our safer system call example by issuing make command

```
myBSD# make
Warning: Object directory not changed from original /root/rootkit/4.1
@ -> /usr/src/sys
machine -> /usr/src/sys/amd64/include
x86 -> /usr/src/sys/x86/include
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mno-sse -mcmodel=kernel -mno-red-zone -mno-mmx -msoft-float  -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions  -Wmissing-include-dirs -fdiagnostics-show-option   -c safe_sc_example.c
ld  -d -warn-common -r -d -o safe_sc_example.ko safe_sc_example.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk safe_sc_example.ko  export_syms | xargs -J% objcopy % safe_sc_example.ko
objcopy --strip-debug safe_sc_example.ko
```

Check if you have successfully built safe_sc_example.ko file, and load it into
the running kernel by

```
myBSD# kldload ./safe_sc_example.ko
System call loaded at offset 210.
```

Great, now you can use the client application we built in the previous session
to make the system call. Just in case you feel as lazy as I do, here's the
client application code again,

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

Save the source code in client.c and build it by

```
myBSD# gcc -o client client.c
client.c: In function 'main':
client.c:12: warning: incompatible implicit declaration of built-in function 'exit'
```

Now we can call our safer system call module example by

```
myBSD# ./client May\ the\ source\ be\ with\ you
myBSD# dmesg | tail -n 1
Safer version output: May the source be with you
```

This is great! We fine-tuned our system call module with the powerful copy
functions, this should be enough for now. User and Kernel space transition is a
very big topic, it involves many kernel design aspects and concepts, I cannot
cover all of them in this short tutorial, but I'll sure get back to this once we
need to.

Hope you enjoy this tutorial, leave a comment below to let me know your
suggestions, and May the source be with you!
