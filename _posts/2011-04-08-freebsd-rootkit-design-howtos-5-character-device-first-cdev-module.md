---
title        : "FreeBSD Rootkit Design Howtos - 5 - Character Device First cdev Module"
date         : "2011-04-08"
tags         : [FreeBSD, Rootkit, C++]
---

Sup guys, welcome back to today's FreeBSD Kernel Rootkit Design Howtos! I know
it's been a couple of days since I posted the previous note, I was dealing with
my visa issue.

## Review

We have discussed two types of kernel modules in the past 4 sessions: General
Kernel Module, and System Call Kernel Module.

Hope you still remember that it took us 3 sessions to discuss the System Call
Kernel Module and a tiny little client application, and lastly we discussed a
little bit on Kernel/User Space Transitions.

So I guess that'd be enough for the System Call Kernel Module since we already
know how to [declare]({% post_url
2011-04-04-freebsd-rootkit-design-howtos-2-system-call-first-kernel-service-module %})
it, how to [call]({% post_url
2011-04-06-freebsd-rootkit-design-howtos-3-system-call-first-kernel-service-application
%})
it, and how to do it in a [safe way]({% post_url
2011-04-07-freebsd-rootkit-design-howtos-4-kernel-and-user-space-transitions %}).

And today we are gonna talk about a new type of kernel module -- ***The Character
Device Module***, I hope you'll enjoy.

## The Device

Before we discuss what is a Character Device, let's talk about what is a
***Device*** first.

A device is simply components connected with your motherboard, such as your hard
disk, USB Drive, Handphone, or even Keyboard and Mouse.

You may have known that one of the most famous "characteristics" of unix-like
systems, is  that [In UNIX Everything is a
File](http://ph7spot.com/musings/in-unix-everything-is-a-file) (This is a great
article, check this out).

That's true, and by Everything we mean documents, directories, links, network
connections, terminals, inter-process communication, and devices.

All of these are represented as files in the unified local filesystem because
they are treated as stream bytes, so you can *read*, *write*, *lseek*, and
*close* them. So getting user input from the keyboard is simply read from the
keyboard device file, very nice and clean right?

The benefit of this design is of course a simplified and unified API for all
devices, and this was very advanced design back in the day when other operating
systems required user to use different commands to copy files from different
types of devices.

You get get a list of device files by entering `ls /dev` in a terminal. `/dev`
is a specific implementation of filesystem called devfs in which all device
files are stored and managed.

## The Block Device and The Character Device

Now we know that we can interact with a device by accessing its device file, but
what actually controls the behavior of a device?

The answer is in the kernel. Since firstly kernel should be the one dealing with
hardware, and secondly the segregation between kernel modules can prevent a
single hardware error from crashing the whole system.

That means if you are a device driver developer and you are about to work on a
new hardware driver for most Unix-like systems, you'll have to program a Device
Kernel Module, and the module should follow the standard and create a file in
devfs for users to interact with.

So that the concept of Device Kernel Module is similar to the concept of driver,
except that in fact there are many different kinds of device kernel modules
designed for different purposes. Here I'm gonna talk about two main types of
modern unix-like system device kernel modules, the ***Block Device Module*** and
the ***Character Device Module***.

| Character Device                                                                                                               | Block Device                                                                                                                    |
|--------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| [Transfers data directly to and from a user process](http://www.freebsd.org/doc/en/books/arch-handbook/driverbasics-char.html) | [Disk devices for which the kernel provides caching](http://www.freebsd.org/doc/en/books/arch-handbook/driverbasics-block.html) |
| Performs no buffering                                                                                                          | Performs buffering and access randomly through a cache                                                                          |
| Read/Write 0 or more bytes in a stream                                                                                         | Read/Write fixed size blocks                                                                                                    |

Block devices are used to mount disk partitions in some other unix-like systems,
and were used by FreeBSD as caching devices. The good news is, as you can see
from [FreeBSD Handbook: Block
Devices](http://www.freebsd.org/doc/en/books/arch-handbook/driverbasics-block.html),
that the block device is deprecated by FreeBSD as a step to modernize the system design.

The caching part was moved upward and is now known as the vnode, so we totally
don't have to worry about block device here, but it's still good idea to
understand the differences between these two types of devices.

To conclude the above boring introduction, background, and design scenario, the
Character Device Module, as the most common type of device driver, is simply
just another type of kernel module that installs itself in the kernel and
provide interface (devfs) represented as file to interact with users.

To successfully declare a character device module, there are three unique things
we need to deal with first, we need to have a ***cdevsw structure***,
***character device functions***, and ***device registration routine***. And of
course, we’ll have to have an event handler and module declaration macro just
like any other kernel modules.

## The cdevsw Structure

Like what we have in a system call module, the cdevsw structure is a structure
in which we store related information about the character device module, and
this structure will be stored in a ***cdevsw structure table*** known as the
character device switch table.

It works similarly to what we do with system call modules, we have a ***sysent
structure*** to store some related information, and then we put that structure
into the ***sysent[]*** table for further reference. So there should be no
problem to understand this concept.

The cdevsw structure is defined in sys/conf.h as the following. I added some
comments for quick reference.

``` c++
FILE:/usr/src/sys/sys/conf.h
/*
 * Character device switch table
 */
struct cdevsw {
        int                     d_version;
        u_int                   d_flags;    /* D_TAPE, D_DISK, D_TTY, D_MEM */
        const char              *d_name;    /* Device name in /dev */
        d_open_t                *d_open;    /* Func. pointer to dev open function */
        d_fdopen_t              *d_fdopen;
        d_close_t               *d_close;   /* Func. pointer to dev close function */
        d_read_t                *d_read;    /* Func. pointer to dev read function */
        d_write_t               *d_write;   /* Func. pointer to dev write function */
        d_ioctl_t               *d_ioctl;   /* Func. pointer to dev ioctl (an operation other than a read or a write) function */
        d_poll_t                *d_poll;    /* Polls a device to see if there is data to be read or space available for writing */
        d_mmap_t                *d_mmap;
        d_strategy_t            *d_strategy;
        dumper_t                *d_dump;
        d_kqfilter_t            *d_kqfilter;
        d_purge_t               *d_purge;
        d_mmap_single_t         *d_mmap_single;

        int32_t                 d_spare0[3];
        void                    *d_spare1[3];

        /* These fields should not be messed with by drivers */
        LIST_HEAD(, cdev)       d_devs;
        int                     d_spare2;
        union {
                struct cdevsw           *gianttrick;
                SLIST_ENTRY(cdevsw)     postfree_list;
        } __d_giant;
};
```

This may look very complicated, but actually there are just two parameters
required to declare a character device, the `*d_name` and the `d_version`.

The `*d_name` is obvious the name of the character device module, and will be
used to name the filename in /dev.

And the `d_version` specifies which FreeBSD version this character device module
supports, it is defined in sys/conf.h as well.

``` c++
FILE:/usr/src/sys/sys/conf.h
/*
 * Version numbers.
 */
#define D_VERSION_00    0x20011966
#define D_VERSION_01    0x17032005      /* Add d_uid,gid,mode & kind */
#define D_VERSION_02    0x28042009      /* Add d_mmap_single */
#define D_VERSION_03    0x17122009      /* d_mmap takes memattr,vm_ooffset_t */
#define D_VERSION       D_VERSION_03
```

The `D_VERSION` is the same as the latest `d_version`, so we're gonna use this
unless you want to have your character device module specifically run on a
particular legacy version of FreeBSD.

So if you decide to be a superb lazy ass, the simplest definition of cdevsw
structure can be,

``` c++
static struct cdevsw cd_example_cdevsw = {
    .d_version = D_VERSION,
    .d_name ="cd_example"
};
```

Unfortunately this character device module pretty much does nothing, despite
that we are not required to define all of the functions such as `*d_open` or
`*d_close`, we still need to have some of them to let our module at least do
something.

## The Character Device Functions

There are a lot of functions we can choose to implement in order to feature our
module, but as I said, it does not require us to implement all of these listed
functions in the definition of cdevsw structure.

Oh, FYI, for those functions that are not specified in the declaration of your
module's cdevsw structure, the operation will be considered as *not supported*.

``` c++
static struct cdevsw cd_example_cdevsw = {
    .d_version = D_VERSION,
    .d_name ="cd_example",
    .d_open = open,
    .d_close = close,
    .d_read = read,
    .d_write = write
};
```

In the example above, we explicitly said that we are gonna define a character
device module named `cd_example_cdevsw`, and its gonna have four character device
functions with their function pointers specified.

Take note that once we define a cdevsw structure with particular functions, the
compiler will expect that function to be implemented, and will result in a
compiling error if we fail to do this.

The prototypes of these functions are defined in the same file, sys/conf.h.
Access the file for a complete list of all prototypes, I'm just gonna list down
the four that we are using,

``` c++
FILE:/usr/src/sys/sys/conf.h
/*
 * Character Device Function Prototypes.
 */
typedef int d_open_t(struct cdev *dev, int oflags, int devtype, struct thread *td);
typedef int d_close_t(struct cdev *dev, int fflag, int devtype, struct thread *td);
typedef int d_read_t(struct cdev *dev, struct uio *uio, int ioflag);
typedef int d_write_t(struct cdev *dev, struct uio *uio, int ioflag);
```

To quickly give you an overall understanding of how this works, we are gonna
firstly create a read-only character device module. And by read-only, it means
that we can only read stream bytes from the module which runs in the kernel back
to the user space.

``` c++
// Function prototype
d_read_t read; //d_read_t is typedef'd in sys/conf.h

// Define a read-only device
static struct cdevsw ro_cdevsw = {
    .d_version = D_VERSION,
    .d_read = read,
    .d_name ="cd_example_ro"
};

static char buf[512+1]; //A string in kernel space
static size_t len;

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;
    if (len <= 0)
        error = -1;
    else
        /* Return the saved character string to userland. */
        copystr(&buf, uio->uio_iov->iov_base, 513, &len);

    return(error);
}
```

In order to implement the read function, we firstly defined it with the `d_read_t`
prototype. Then we defined a cdevsw structure called `ro_cdevsw` with only one
function pointer passed, the read function.

Now since we are gonna read a string from the kernel back to the user space, we
at least need a string in the kernel, right? So we defined a string called
`buf[512+1]`.

Finally we reached the implementation of the read function, everything is usual
except that we used `copystr()` here.

I hope you still remember that there are four copy functions, `copyin()`,
`copyinstr()`, `copyout()`, and `copystr()`. All these four functions copy a
continues data from kernel space to user space or vise versa except `copystr()`.
`copystr()` function copies data from one kernel space address to another kernel space
address. It is defined in sys/systm.h with following function prototype.

``` c++
FILE: /usr/src/sys/sys/systm.h
int     copystr(const void * __restrict kfaddr, void * __restrict kdaddr,
            size_t len, size_t * __restrict lencopied)
            __nonnull(1) __nonnull(2);
```

What we did with `copystr(&buf, uio->uio_iov->iov_base, 513, &len);` was to copy
data from `buf` (which is a string in kernel space) to `uio->uio_iov->iov_base`
which as a matter of fact, is also a kernel address, with maximum length 513,
and save copied length to `len`.

It's strange that we used `copystr()` function, 'cause it copies data from
kernel space address to another kernel space address, but we want to copy the
`buf[]` string from kernel back to user space. This is obviously nothing like a
read operation that we expected.

To answer this question, we have to take a look into the destination kernel
space address  `uio->uio_iov->iov_base` that came out of nowhere.

In fact, the `*uio` structure does not came out of nowhere, it is actually
defined in the `read` function prototype, and it was passed to our read function
like this: `int read(struct cdev *dev, struct uio *uio, int ioflag){}`. That
means every read action performed to a character device module from the user
space needs to pass the `*uio` structure.

So now the question is, what is `*uio`, and why `*uio`. Here's the definition of
it in sys/uio.h.

``` c++
FILE:/usr/src/sys/sys/uio.h
struct uio {
        struct  iovec *uio_iov;         /* scatter/gather list */
        int     uio_iovcnt;             /* length of scatter/gather list */
        off_t   uio_offset;             /* offset in target object */
        ssize_t uio_resid;              /* remaining bytes to process */
        enum    uio_seg uio_segflg;     /* address space */
        enum    uio_rw uio_rw;          /* operation */
        struct  thread *uio_td;         /* owner */
};
```

I'll leave the why question to UIO(9) manpage, it says,

>As a result of any read(2), write(2), readv(2), or writev(2) system call that
>is being passed to a character-device driver, the appropriate driver d_read or
>d_write entry will be called with a pointer to a struct uio being passed.
>
>The transfer request is encoded in this structure. The driver itself should use
>uiomove() or  uiomove_nofault() to get at the data in this structure.

That means we've got to have `uio` passed to `read()` or `write()` function and
then use `uiomove()` or `uiomove_nofault()` to,

> The functions uiomove() and uiomove_nofault() are used to transfer data
> between buffers and I/O vectors that might possibly cross the user/kernel
> space boundary.

Now let's take a look at `iovec *uio_iov` array. The I/O vector is defined in
sys/_iovec.h

``` c++
FILE:/usr/src/sys/sys/_iovec.h
struct iovec {
        void    *iov_base;      /* Base address. */
        size_t   iov_len;       /* Length. */
};
```

Take note that the `iov_base` is the base address of the I/O vector, so by
`copystr(&buf, uio->uio_iov->iov_base, 513, &len);` what we are really doing here
is that we copied maximum 513 length of stream bytes from the starting address
of the buffer string `&buf` to the base address of I/O vector
`uio->uio_iov->iov_base`.

We'll get to the `uiomove()` function later, as for now, let's discuss about the
very last new thing we have to know to declare a character device module.

## The Device Registration Routine

The registration of a device module is accomplished by calling the declaration
module called `DEV_MODULE`. Before that, we need to have an event handler which
defines the actions to be performed when the device module is loaded or
unloaded.

This is simple, we call `make_dev()` function to register our character device
module with kernel and create the device file in devfs when the module is
loaded, and we call `destroy_dev()` function to unregister the module and remove
the device file from /dev when the module is unloaded.

### `make_dev()` function

``` c++
FILE:/usr/src/sys/sys/conf.h
struct cdev *make_dev(
    struct cdevsw *_devsw, //pointer to cdevsw structure of the device module which was defined previously
    int _unit, //Normally set to 0
    uid_t _uid, //The owner ID of the device file
    gid_t _gid, //The owner group ID of the device file
    int _perms, //The permissions of the device file, e.g. 0600
    const char *_fmt, //The name of the device
...) __printflike(6, 7);
```

This function is straight forward, we pass the `cdevsw` structure to the
`make_dev()` function to finally get it registered into the character device
switch table. What's new is the return value of the function, which is a `cdev`
structure. Well we've already discussed about the cdevsw structure, so what's a
cdev structure?

### cdev structure

Let's take a look at the definition of the cdev structure which is defined in
sys/conf.h as shown below,

``` c++
struct cdev {
        void            *__si_reserved;
        u_int           si_flags;
#define SI_ETERNAL      0x0001  /* never destroyed */
#define SI_ALIAS        0x0002  /* carrier of alias name */
#define SI_NAMED        0x0004  /* make_dev{_alias} has been called */
#define SI_CHEAPCLONE   0x0008  /* can be removed_dev'ed when vnode reclaims */
#define SI_CHILD        0x0010  /* child of another struct cdev **/
#define SI_DEVOPEN      0x0020  /* opened by device */
#define SI_CONSOPEN     0x0040  /* opened by console */
#define SI_DUMPDEV      0x0080  /* is kernel dumpdev */
#define SI_CANDELETE    0x0100  /* can do BIO_DELETE */
#define SI_CLONELIST    0x0200  /* on a clone list */
        struct timespec si_atime;
        struct timespec si_ctime;
        struct timespec si_mtime;
        uid_t           si_uid;
        gid_t           si_gid;
        mode_t          si_mode;
        struct ucred    *si_cred;       /* cached clone-time credential */
        int             si_drv0;
        int             si_refcount;
        LIST_ENTRY(cdev)        si_list;
        LIST_ENTRY(cdev)        si_clone;
        LIST_HEAD(, cdev)       si_children;
        LIST_ENTRY(cdev)        si_siblings;
        struct cdev *si_parent;
        char            *si_name;
        void            *si_drv1, *si_drv2;
        struct cdevsw   *si_devsw;
        int             si_iosize_max;  /* maximum I/O size (for physio &al) */
        u_long          si_usecount;
        u_long          si_threadcount;
        union {
                struct snapdata *__sid_snapdata;
        } __si_u;
        char            __si_namebuf[SPECNAMELEN + 1];
};
```

This look scary, but we don't have to worry about all these parameters. Since it
is a returen value passed to us, we'll have all necessary information set for
us.

### destroy_dev() function

Everything is just automagical, oh, except one thing -- we have to explicitly
define a `cdev` structure to receive the return value of the `make_dev()` function,
since we'll need to use it for our `destroy_dev()` function. The `destroy_dev()`
function is defined in sys/conf.h as shown below,

``` c++
FILE:/usr/src/sys/sys/conf.h
void    destroy_dev(struct cdev *_dev);
```

This is superb simple, the `destroy_dev()` function will unregister the module and
remove the device file in /dev, so it needs the cdev structure we just got from
`make_dev()` function to get the module's devfs information in order to destroy
the device successfully.

Thus, we can update our example as the following,

``` c++
// Function prototype
d_read_t read; //d_read_t is typedef'd in sys/conf.h

// Define a read-only device
static struct cdevsw ro_cdevsw = {
    .d_version = D_VERSION,
    .d_read = read,
    .d_name ="cd_example_ro"
};

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
    uprintf("Read function called!n");
}

/* Event Handler Code Here... */
...
static struct cdev *sdev;
sdev = make_dev(&ro_cdevsw, 0, UID_ROOT, GID_WHEEL, 0600, "cd_example_ro");
destroy_dev(sdev);
...

/* Declaration Macro Here... */
DEV_MODULE(...)
```

In the above example, we set the owner of the device to root, owner group to
wheel, and permissions to 0600. A complete list of USER_IDs and GROUP_IDs are
defined in sys/conf.h

``` c++
FILE:/usr/src/sys/sys/conf.h
#define         UID_ROOT        0
#define         UID_BIN         3
#define         UID_UUCP        66
#define         UID_NOBODY      65534

#define         GID_WHEEL       0
#define         GID_KMEM        2
#define         GID_TTY         4
#define         GID_OPERATOR    5
#define         GID_BIN         7
#define         GID_GAMES       13
#define         GID_DIALER      68
#define         GID_NOBODY      65534
```

### DEV_MODULE macro

Here comes the last part of the device registration routine, the declaration
macro. The definition of `DEV_MODULE` macro is in sys/conf.h as shown below,

``` c++
FILE:/usr/src/sys/sys/conf.h
#define DEV_MODULE(name, evh, arg)                                      \
static moduledata_t name##_mod = {                                      \
    #name,                                                              \
    evh,                                                                \
    arg                                                                 \
};                                                                      \
DECLARE_MODULE(name, name##_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE)
```

Notice that when we use `DEV_MODULE`, it will automatically call the general
DECLARE_MODULE with default module type and SI_SUB_DRIVERS, and the module
position will be somewhere in the middle. So all we need to do is to specify the
general name, the event handler function pointer, that's all.

### Connecting the dots

Now let's connect all the pieces of code together to see how a character device
register routine looks like.

``` c++
// Function prototype
d_read_t read; //d_read_t is typedef'd in sys/conf.h

// Define a read-only device
static struct cdevsw ro_cdevsw = {
    .d_version = D_VERSION,
    .d_read = read,
    .d_name ="cd_example_ro"
};

static char buf[512+1]; //A string in kernel space
static size_t len;

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;
    if (len <= 0)
        error = -1;
    else
    /* Return the saved character string to userland. */
        copystr(&buf, uio->uio_iov->iov_base, 513, &len);

    return(error);
}

/* Reference to the device in DEVFS */
static struct cdev *sdev;

static int load(struct module *module, int cmd, void *arg)
{
    int error = 0;

    switch (cmd)
    {
        case MOD_LOAD:
        sdev = make_dev(&ro_cdevsw, 0, UID_ROOT, GID_WHEEL, 0600, "cd_example_ro");
        uprintf("Character device loaded.n"); break;

        case MOD_UNLOAD: destroy_dev(sdev);
        uprintf("Character device unloaded.n"); break;

        default: error = EOPNOTSUPP;
        break;
    }

    return(error);
}

DEV_MODULE(cd_example_ro, load, NULL);









#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/conf.h>
#include <sys/uio.h>

// Function prototypes
d_open_t open;
d_close_t close;
d_read_t read;
d_write_t write;

static struct cdevsw cd_example_cdevsw = {
    .d_version = D_VERSION,
    .d_open = open,
    .d_close = close,
    .d_read = read,
    .d_write = write,
    .d_name ="cd_example"
};

static char buf[512+1]; //A string in kernel space
static size_t len;

int open(struct cdev *dev, int flag, int otyp, struct thread *td)
{
/* Initialize character buffer. */
memset(&buf, '�', 513);
len = 0;

return(0);
}

int close(struct cdev *dev, int flag, int otyp, struct thread *td)
{
return(0);
}

int write(struct cdev *dev, struct uio *uio, int ioflag)
{
int error = 0;

/*
* Take in a character string, saving it in buf.
* Note: The proper way to transfer data between buffers and I/O
* vectors that cross the user/kernel space boundary is with
* uiomove(), but this way is shorter. For more on device driver I/O
* routines, see the uio(9) manual page.
*/
error = copyinstr(uio->uio_iov->iov_base, &buf, 512, &len);
if (error != 0)
uprintf("Write to "cd_example" failed.n");

return(error);
}

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
int error = 0;

if (lenuio_iov->iov_base, 513, &len);

return(error);
}

/* Reference to the device in DEVFS */
static struct cdev *sdev;

static int load(struct module *module, int cmd, void *arg)
{
int error = 0;

switch (cmd)
{
case MOD_LOAD:
sdev = make_dev(&cd_example_cdevsw, 0, UID_ROOT, GID_WHEEL, 0600, "cd_example");
uprintf("Character device loaded.n");
break;

case MOD_UNLOAD:
destroy_dev(sdev);
uprintf("Character device unloaded.n");
break;

default:
error = EOPNOTSUPP;
break;
}

return(error);
}

DEV_MODULE(cd_example, load, NULL);
```

## The Implementation

Have discussed all these, we are now ready to look at the example code from the
book.

``` c++
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/conf.h>
#include <sys/uio.h>

// Function prototypes
d_open_t open;
d_close_t close;
d_read_t read;
d_write_t write;

static struct cdevsw cd_example_cdevsw = {
    .d_version = D_VERSION,
    .d_open = open,
    .d_close = close,
    .d_read = read,
    .d_write = write,
    .d_name ="cd_example"
};

static char buf[512+1]; //A string in kernel space
static size_t len;

int open(struct cdev *dev, int flag, int otyp, struct thread *td)
{
    /* Initialize character buffer. */
    memset(&buf, '�', 513);
    len = 0;

    return(0);
}

int close(struct cdev *dev, int flag, int otyp, struct thread *td)
{
    return(0);
}

int write(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;


    /*
     * Take in a character string, saving it in buf.
     * Note: The proper way to transfer data between buffers and I/O
     * vectors that cross the user/kernel space boundary is with
     * uiomove(), but this way is shorter. For more on device driver I/O
     * routines, see the uio(9) manual page.
    */
    error = copyinstr(uio->uio_iov->iov_base, &buf, 512, &len);
    if (error != 0)
        uprintf("Write to "cd_example" failed.n");

    return(error);
}

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;

    if (len <= 0)
        error = -1;
    else
        /* Return the saved character string to userland. */
        copystr(&buf, uio->uio_iov->iov_base, 513, &len);

    return(error);
}

/* Reference to the device in DEVFS */
static struct cdev *sdev;

static int load(struct module *module, int cmd, void *arg)
{
    int error = 0;

    switch (cmd)
    {
        case MOD_LOAD:
            sdev = make_dev(&cd_example_cdevsw, 0, UID_ROOT, GID_WHEEL, 0600, "cd_example");
            uprintf("Character device loaded.n");
            break;

        case MOD_UNLOAD:
            destroy_dev(sdev);
            uprintf("Character device unloaded.n");
            break;

        default:
            error = EOPNOTSUPP;
            break;
    }

    return(error);
}

DEV_MODULE(cd_example, load, NULL);
```

Now compile and try to load it into kernel.

```
ZFSTest# make
Warning: Object directory not changed from original /zroot/development/3.Character_device
cc -O2 -pipe -fno-strict-aliasing -Werror -D_KERNEL -DKLD_MODULE -nostdinc   -I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 --param large-function-growth=1000 -fno-common  -fno-omit-frame-pointer  -mcmodel=kernel -mno-red-zone  -mfpmath=387 -mno-sse -mno-sse2 -mno-sse3 -mno-mmx -mno-3dnow  -msoft-float -fno-asynchronous-unwind-tables -ffreestanding -fstack-protector -std=iso9899:1999 -fstack-protector -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-sign -fformat-extensions -c cdev.c
ld  -d -warn-common -r -d -o cd_example.ko cdev.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk cd_example.ko  export_syms | xargs -J% objcopy % cd_example.ko
objcopy --strip-debug cd_example.ko
ZFSTest# kldload ./cd_example.ko
Character device loaded.
```

Now we can see that the module is successfully loaded to the kernel. And if you
check the /dev folder now, you can find your very first own DEVFS file.

```
ZFSTest# ls -lF /dev/cd_example
crw-------  1 root  wheel    0,  96 Apr 13 10:23 /dev/cd_example
```

All right then, this should be all for today. We'll talk about how to call our
character device from user space with an application in the next note, see you
then.
