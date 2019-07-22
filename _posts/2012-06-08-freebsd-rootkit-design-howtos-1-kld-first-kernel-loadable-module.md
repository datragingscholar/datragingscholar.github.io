---
title        : "FreeBSD Rootkit Design Howtos - 1 - KLD First Kernel Loadable Module"
date         : "2012-06-08 21:53:49"
tags         : [FreeBSD, Rootkit]
---

Needless to say that the first thing you need to start FreeBSD kernel rootkit
development is a FreeBSD box. There are plenty of installation guides on the
Internet about FreeBSD installation, so that I’ll just skip this part for the
sake of simplicity.

And FYI, basically you are free to choose any hardware to run FreeBSD as long as
it's supported. You can even install your FreeBSD as a virtual machine. 5GB hard
drive, at least 256MB memory with any modern CPU will do.

## The Environment

The version of FreeBSD I’m going to use throughout this howtos is FreeBSD 9
Stable, it is recommended that you install at least FreeBSD 9 Release although I
believe code examples in this howtos are able to run on most of versions after
FreeBSD 6.2

```
myBSD# uname -a
FreeBSD myBSD 9.0-STABLE FreeBSD 9.0-STABLE #0: Thu Mar  8 14:16:25 CST 2012 bestwc@myBSD:/usr/obj/usr/src/sys/GENERIC  amd64
```

## Package Installation

You'll need the FreeBSD source tree to compile your kernel rootkit, be sure to
install it because it is not included in minimal installation. You can do this
by two ways, the first is to enable src distribution option during installation,
and the recommended way is to perform the following command after the
installation.

```
myBSD# cp /usr/share/examples/cvsup/stable-supfile /root && cd /root
myBSD# ee stable-supfile
=================================
*default host=CHANGE_THIS.FreeBSD.org ###Change this line
*default host=cvsup.FreeBSD.org ###To this
=================================
myBSD# csup -g -L2 stable-supfile
```

This will sync the source tree on your local file system to the latest one from
remote repository, and this will simply download the latest for you if you don’t
already have the source tree. It is recommended to run the cusp command shown
above to frequently update your source tree, for this will make sure your are
developing rootkit with the newest kernel source code, and potentially make it
more compatible with new FreeBSD releases.

To examine if you have the src tree on your local box, type the following
command:

```
myBSD# ls /usr/src
COPYRIGHT       ObsoleteFiles.inc       crypto          lib         share
LOCKS           README      etc         libexec         sys
MAINTAINERS     UPDATING    games       release         tools
Makefile        bin         gnu         rescue          usr.bin
Makefile.inc1   cddl        include     sbin            usr.sbin
Makefile.mips   contrib     kerberos5   secure
```

## The Dynamic Kernel Linker (KLD)

The Dynamic Kernel Linker (KLD) is a facility in FreeBSD that allows users to
interact with the system kernel by dynamically loading or unloading kernel
modules. It may sounds strange to you, but believe it or not that you have
utilized KLD multiple times during daily operations on your FreeBSD box. The KLD
gets called every time you plug in or plug out a device, or by manually typing
kldload or kldunload commands. It is especially useful to device driver
developers as they can dynamically load their drivers as kernel module and test
the functionalities on the fly without rebooting the system. For more
information on the KLD, please refer to the FreeBSD Handbook.

The KLD provides a high way for us to put our code into the running kernel space
without recompiling as long as we have the required privilege. So that, if we
have our kernel rootkit compiled as loadable kernel module, then we’ll
theoretically be able to load that module in any FreeBSD machines on the fly.

Although the KLD interface is not the only way for people to interact with
kernel, but is undoubtedly the easiest and probably the fastest way to do that.
The only question is, will there be any compatibility issues if we stick to KLD
interface?

> In FreeBSD 3.0, substantial changes were made to the kernel module subsystem,
> and the LKM Facility was renamed the Dynamic Kernel Linker (KLD) Facility.
> Subsequently, the term KLD is commonly used to describe LKMs under FreeBSD.

According to the quote above from Designing BSD Rootkits: An Introduction to
Kernel Hacking by Joseph Kong, the KLD interface hasn’t been changed since
FreeBSD 3.0, which means our yet-to-be-done rootkit should be able to run on any
(not always true) modern FreeBSD systems without any (not always true as well)
modification.

Just in case that you haven’t realized, we are going to use KLD interfaces a lot
in this tutorial. Now let’s get to know the KLD interface more.

## The Module Event Handler

When you load or unload any kernel modules to or from the current running
kernel, the KLD interface will perform some pre-defined routines to prepare the
system. This is called the Module Event Handler, it should be present in
every(!) kernel module to handle the initialization and shutdown processes. This
handler gets called every time when the code enters or exits kernel space.

*(!) Just keep in mind that this isn’t always true, as what the quote says below
from Designing BSD Rootkits: An Introduction to Kernel Hacking*

> Actually, this isn’t entirely true. You can have a KLD that just includes a
> sysctl. You can also dispense with module handlers if you wish and just use
> SYSINIT and SYSUNINIT directly to register functions to be invoked on load and
> unload, respectively. You can’t, however, indicate failure in those.

The prototype of the module event handler is defined in *sys/module.h* and it is
called *modeventhand_t*

``` c++
FILE:/usr/src/sys/sys/module.h

myBSD# cat /usr/src/sys/sys/module.h | grep modeventhand_t
typedef int (*modeventhand_t)(module_t, int /* modeventtype_t */, void *);
```

And *module_t* is a pointer to the module’s struct as defined in the same file.

``` c++
FILE:/usr/src/sys/sys/module.h

typedef struct module *module_t;
```

The *modeventtype_t* on the other hand is an enumerated type of event types
defined in the same file as well.

``` c++
FILE:/usr/src/sys/sys/module.h

typedef enum modeventtype {
        MOD_LOAD,    /* Set when module is loaded. */
        MOD_UNLOAD,    /* Set when module is unloaded. */
        MOD_SHUTDOWN,    /* Set on shutdown. */
        MOD_QUIESCE    /* Set on quiesce. */
} modeventtype_t;
```

With all these information, we can now try to define a simple module event
handler. Here is a simple event handler function called load, which displays
Hello, world! when it’s loaded, and print Good-bye, cruel world! when it’s
unloaded.

``` c++
static int load(struct module *module, int cmd, void *arg)
{
    int error = 0;

    switch (cmd) {
    case MOD_LOAD:
        uprintf("Hello, world!\n");
        break;

    case MOD_UNLOAD:
        uprintf("Good-bye, cruel world!\n");
        break;

    default:
        error = EOPNOTSUPP; //EOPNOTSUPP stands for Error: Operation not supported.
        break;
    }

    return(error);
}
```

It’s perfectly safe if this code doesn’t make much sense to you, as we will get
back later. However, what you do need to understand is the prototype of defining
a *module event handler*.

This code snippet should present in your first loadable kernel module, and
perform pre-defined routines according to the cmd sent by the kernel
accordingly.

Consider this to be a protocol between your module and the kernel, which
basically says *"Oh the user asked you to unload me? Hold on and let me check my
event handler, alright, I’ll print Good-bye, cruel world! and then go away."*

## The DECLARE_MODULE Macro

It's time to get back to our module declaration.

We now know *module_t* is a pointer to the module structure, but what on earth is
a module and how do we define it? The thing is, we must let KLD know the basic
information about our module, and it should register itself with kernel when it
loads. This process can be awfully long and complicated, lucky that we have a
pre-defined macro in sys/module.h that just does this to help us.

``` c++
FILE:/usr/src/sys/sys/module.h
#define DECLARE_MODULE_WITH_MAXVER(name, data, sub, order, maxver)
#define DECLARE_MODULE(name, data, sub, order)
 * The module declared with DECLARE_MODULE_TIED can only be loaded
#define DECLARE_MODULE_TIED(name, data, sub, order)
```

As shown above, there are actually tree macros for us to declare a module,
*DECLARE_MODULE*, *DECLARE_MODULE_TIED*, and *DECLARE_MODULE_WITH_MAXVER*.

* DECLARE_MODULE
  * The simplest one that requires only four parameters to be passed, for normal
    purposes
* DECLARE_MODULE_TIED
  * Use this macro when your kernel module can only be loaded into the kernel
    with exactly the same _FreeBSD_version_
* DECLARE_MODULE_WITH_MAXVER
  * To declare kernel modules that can only run on machines with FreeBSD version
    maxver or lower

We'll stick with the general *DECLARE_MODULE* macro, and now let's discuss it's
four parameters: *name*, *data*, *sub*, and *order*.

### name

> This specifies the generic module name, which is passed as a character string.

### data

> This parameter specifies the official module name and event handler function,
> which is passed as a moduledata structure.

The *moduledata structure* is defined in sys/module.h

``` c++
FILE:/usr/src/sys/sys/module.h
typedef struct moduledata {
        const char      *name;          /* module name */
        modeventhand_t  evhand;         /* event handler */
        void            *priv;          /* extra data */
} moduledata_t;
```

### sub

> This specifies the system startup interface namely sysinit_sui_id, which
> identifies the module type.

We can find a complete list of *sysinit_sub_id* from sys/kernel.h

``` c++
FILE:/usr/src/sys/sys/kernel.h
enum sysinit_sub_id {
        SI_SUB_DUMMY            = 0x0000000,    /* not executed; for linker*/
        SI_SUB_DONE             = 0x0000001,    /* processed*/
        SI_SUB_TUNABLES         = 0x0700000,    /* establish tunable values */
        SI_SUB_COPYRIGHT        = 0x0800001,    /* first use of console*/
        SI_SUB_SETTINGS         = 0x0880000,    /* check and recheck settings */
        SI_SUB_MTX_POOL_STATIC  = 0x0900000,    /* static mutex pool */
        SI_SUB_LOCKMGR          = 0x0980000,    /* lockmgr locks */
        SI_SUB_VM               = 0x1000000,    /* virtual memory system init*/
        SI_SUB_KMEM             = 0x1800000,    /* kernel memory*/
        SI_SUB_KVM_RSRC         = 0x1A00000,    /* kvm operational limits*/
        SI_SUB_WITNESS          = 0x1A80000,    /* witness initialization */
        SI_SUB_MTX_POOL_DYNAMIC = 0x1AC0000,    /* dynamic mutex pool */
        SI_SUB_LOCK             = 0x1B00000,    /* various locks */
        SI_SUB_EVENTHANDLER     = 0x1C00000,    /* eventhandler init */
        SI_SUB_VNET_PRELINK     = 0x1E00000,    /* vnet init before modules */
        SI_SUB_KLD              = 0x2000000,    /* KLD and module setup */
        SI_SUB_CPU              = 0x2100000,    /* CPU resource(s)*/
        SI_SUB_RACCT            = 0x2110000,    /* resource accounting */
        SI_SUB_RANDOM           = 0x2120000,    /* random number generator */
        SI_SUB_KDTRACE          = 0x2140000,    /* Kernel dtrace hooks */
        SI_SUB_MAC              = 0x2180000,    /* TrustedBSD MAC subsystem */
        SI_SUB_MAC_POLICY       = 0x21C0000,    /* TrustedBSD MAC policies */
        SI_SUB_MAC_LATE         = 0x21D0000,    /* TrustedBSD MAC subsystem */
        SI_SUB_VNET             = 0x21E0000,    /* vnet 0 */
        SI_SUB_INTRINSIC        = 0x2200000,    /* proc 0*/
        SI_SUB_VM_CONF          = 0x2300000,    /* config VM, set limits*/
        SI_SUB_DDB_SERVICES     = 0x2380000,    /* capture, scripting, etc. */
        SI_SUB_RUN_QUEUE        = 0x2400000,    /* set up run queue*/
        SI_SUB_KTRACE           = 0x2480000,    /* ktrace */
        SI_SUB_OPENSOLARIS      = 0x2490000,    /* OpenSolaris compatibility */
        SI_SUB_CYCLIC           = 0x24A0000,    /* Cyclic timers */
        SI_SUB_AUDIT            = 0x24C0000,    /* audit */
        SI_SUB_CREATE_INIT      = 0x2500000,    /* create init process*/
        SI_SUB_SCHED_IDLE       = 0x2600000,    /* required idle procs */
        SI_SUB_MBUF             = 0x2700000,    /* mbuf subsystem */
        SI_SUB_INTR             = 0x2800000,    /* interrupt threads */
        SI_SUB_SOFTINTR         = 0x2800001,    /* start soft interrupt thread */
        SI_SUB_ACL              = 0x2900000,    /* start for filesystem ACLs */
        SI_SUB_DEVFS            = 0x2F00000,    /* devfs ready for devices */
        SI_SUB_INIT_IF          = 0x3000000,    /* prep for net interfaces */
        SI_SUB_NETGRAPH         = 0x3010000,    /* Let Netgraph initialize */
        SI_SUB_DTRACE           = 0x3020000,    /* DTrace subsystem */
        SI_SUB_DTRACE_PROVIDER  = 0x3048000,    /* DTrace providers */
        SI_SUB_DTRACE_ANON      = 0x308C000,    /* DTrace anon enabling */
        SI_SUB_DRIVERS          = 0x3100000,    /* Let Drivers initialize */
        SI_SUB_CONFIGURE        = 0x3800000,    /* Configure devices */
        SI_SUB_VFS              = 0x4000000,    /* virtual filesystem*/
        SI_SUB_CLOCKS           = 0x4800000,    /* real time and stat clocks*/
        SI_SUB_CLIST            = 0x5800000,    /* clists*/
        SI_SUB_SYSV_SHM         = 0x6400000,    /* System V shared memory*/
        SI_SUB_SYSV_SEM         = 0x6800000,    /* System V semaphores*/
        SI_SUB_SYSV_MSG         = 0x6C00000,    /* System V message queues*/
        SI_SUB_P1003_1B         = 0x6E00000,    /* P1003.1B realtime */
        SI_SUB_PSEUDO           = 0x7000000,    /* pseudo devices*/
        SI_SUB_EXEC             = 0x7400000,    /* execve() handlers */
        SI_SUB_PROTO_BEGIN      = 0x8000000,    /* XXX: set splimp (kludge)*/
        SI_SUB_PROTO_IF         = 0x8400000,    /* interfaces*/
        SI_SUB_PROTO_DOMAININIT = 0x8600000,    /* domain registration system */
        SI_SUB_PROTO_DOMAIN     = 0x8800000,    /* domains (address families?)*/
        SI_SUB_PROTO_IFATTACHDOMAIN     = 0x8800001,    /* domain dependent data init*/
        SI_SUB_PROTO_END        = 0x8ffffff,    /* XXX: set splx (kludge)*/
        SI_SUB_KPROF            = 0x9000000,    /* kernel profiling*/
        SI_SUB_KICK_SCHEDULER   = 0xa000000,    /* start the timeout events*/
        SI_SUB_INT_CONFIG_HOOKS = 0xa800000,    /* Interrupts enabled config */
        SI_SUB_ROOT_CONF        = 0xb000000,    /* Find root devices */
        SI_SUB_DUMP_CONF        = 0xb200000,    /* Find dump devices */
        SI_SUB_RAID             = 0xb380000,    /* Configure GEOM classes */
        SI_SUB_SWAP             = 0xc000000,    /* swap */
        SI_SUB_INTRINSIC_POST   = 0xd000000,    /* proc 0 cleanup*/
        SI_SUB_SYSCALLS         = 0xd800000,    /* register system calls */
        SI_SUB_VNET_DONE        = 0xdc00000,    /* vnet registration complete */
        SI_SUB_KTHREAD_INIT     = 0xe000000,    /* init process*/
        SI_SUB_KTHREAD_PAGE     = 0xe400000,    /* pageout daemon*/
        SI_SUB_KTHREAD_VM       = 0xe800000,    /* vm daemon*/
        SI_SUB_KTHREAD_BUF      = 0xea00000,    /* buffer daemon*/
        SI_SUB_KTHREAD_UPDATE   = 0xec00000,    /* update daemon*/
        SI_SUB_KTHREAD_IDLE     = 0xee00000,    /* idle procs*/
        SI_SUB_SMP              = 0xf000000,    /* start the APs*/
        SI_SUB_RACCTD           = 0xf100000,    /* start raccd*/
        SI_SUB_RUN_SCHEDULER    = 0xfffffff     /* scheduler*/
};
```

*sysinit_sub_id* is a list of pre-defined enumerated constant data to declare
kernel modules for different purposes. We’ll use *SI_SUB_DRIVERS* in our first few
examples, and some other useful subs are to be introduced in later chapters.

### order

> This specifies the KLD’s order of initialization within the subsystem, namely
> the *sysinit_elem_order*.

A complete list of *sysinit_elem_order* can be found in sys/kernel.h

``` c++
FILE:/usr/src/sys/sys/kernel.h
enum sysinit_elem_order {
        SI_ORDER_FIRST          = 0x0000000,    /* first*/
        SI_ORDER_SECOND         = 0x0000001,    /* second*/
        SI_ORDER_THIRD          = 0x0000002,    /* third*/
        SI_ORDER_FOURTH         = 0x0000003,    /* fourth*/
        SI_ORDER_MIDDLE         = 0x1000000,    /* somewhere in the middle */
        SI_ORDER_ANY            = 0xfffffff     /* last*/
};
```

For the sake of simplicity and to concentrate on our purpose, we’ll stick to
*SI_ORDER_MIDDLE* which declares our kernel module somewhere in the middle of the
KLD subsystem. We will discuss the others if we need to.

## Summary

We've talked lots of concepts and kernel terminology, probably too much for a
beginner. So just before we go for example code, let’s try to quickly review
what we have discussed first.

* *KLD* is the way how we interact our code with FreeBSD kernel

* Every KLD module (At least in our case) needs a *module event handler* called
  *modeventhand_t* and it is defined in sys/module.h

    * The prototype of *modeventhand_t* requires *module_t* and *modeventtype_t*

    * *module_t* is a pointer to our yet-to-be-declared module. We use
      *DECLARE_MODULE* to declare general kernel modules

    * The *DECLARE_MODULE* is defined in sys/module.h and it requires four
      parameters: *name*, *data*, *sub*, and *order*

      * name is the general name for the module

      * data is passed as moduledata consists of the official name of the module
        and the event handler of the module. It is defined in sys/module.h

      * sub is the identifier of the type of the module. It is listed in
        sysinit_sub_id enum defined in sys/kernel.h

      * order is the initialization position of the module. It is listed in
        sysinitelemorder enum defined in sys/kernel.h

    * Finally we need *modeventtype_t* to complete *modeventhand_t* prototype

    * *modeventtype_t* is an enum types of events defined in sys/module.h to let
      the kernel module perform corresponding actions when certain event happens

Here are two figures for you to get an overall understanding of how this works.

![Chart 1](/assets/images/freebsd-rootkit-design-howtos/chart1_1.png)

![Chart 2](/assets/images/freebsd-rootkit-design-howtos/chart1_2.png)

## Example Code

So, after all these pains and hair-pulling, we are finally gonna have our first
kernel loadable module compiled.

``` c++
/*
 * FILE: /root/rootkit/1.1/first_module.c
 * Example 1.1
 * The First Kernel Loadable Module
 * FreeBSD Rootkit Design Howtos @ www.hailang.me
*/
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

/* Define a loader function */
static int module_loader(struct module *module, int cmd, void *arg)
{
    int error=0;

    switch(cmd)
    {
        case MOD_LOAD:
            uprintf("Lets Rock the Kernel!\n");
            break;

        case MOD_UNLOAD:
            uprintf("Time to Leave the Party!\n");
            break;

        default:
            error=EOPNOTSUPP; //Operation not supported
            break;
    }
    return (error);
}

/* Define a module data structure */
static moduledata_t module_data = {
    "first_mlodule",    // Module name
    module_loader,      // Event Handler
    NULL                // Extra Data
};

/* Declare the module */
DECLARE_MODULE(first_mlodule, module_data, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);
```

Now there’s just one more thing left before we can compile the code, the
makefile. If you are not familiar with the concept of makeifle, please google it
and get yourself familiar with its basic syntax, just a little bit by now.

Here’s the content of makefile

``` makefile
KMOD=   first_module
SRCS=   first_module.c

.include <bsd.kmod.mk>
```

Save it as makefile in the same folder with first_module.c

As you can see, we use bsd.kmod.mk macro here to make our life easier, because
otherwise we'll have to link all kernel source files manually.

All we have to do here is to fill in the module name and source file name. With
the makefile and the source file with us, we can now go ahead and compile our
first kernel module by entering *make* in the same folder.

```
myBSD# make
Warning: Object directory not changed from original /root/rootkit/1.1
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mno-sse -mcmodel=kernel -mno-red-zone -mno-mmx -msoft-float  -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions  -Wmissing-include-dirs -fdiagnostics-show-option   -c first_module.c
ld  -d -warn-common -r -d -o first_module.ko first_module.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk first_module.ko  export_syms | xargs -J% objcopy % first_module.ko
objcopy --strip-debug first_module.ko
```

Looks everything is nice and clean, we now have our first module successfully
compiled. Now go ahead and treat yourself a muffin.

Let’s take a look at what we have now.

```
myBSD# ls
@       export_syms first_module.c  first_module.ko first_module.o  machine     makefile    x86
```

We have several files and linkers added here, but hello.ko is what we really
care about. That is the result of all our hard work — the loadable module.

It’s like an executable file, same as the file you get in regular application
programing, and yes, it is distributable.

Just before you bite that muffin, let’s have our fun first by loading and
unloading it.

```
myBSD# kldload ./first_module.ko
Lets Rock the Kernel!
myBSD# kldunload first_module.ko
Time to Leave the Party!
myBSD#
```

Sweet huh? All of your hard works have been paid by seeing these silly but pretty words printing on your screen.
