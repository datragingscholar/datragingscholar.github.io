---
title        : "FreeBSD Rootkit Design Howtos - 6 - Character Device First cdev Application"
date         : "2011-04-13"
tags         : [FreeBSD, Rootkit, C++]
---

This is a refined version using `uiomove()` instead of `copystr()`. It surprised
me when I found out how easy it was to use `uiomove()` function. Anyway, I made
the change, along with some debug messages to make the result more descriptive,
so let’s take a look at the refined code and compile it first.

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
    printf("[cd_example]Open Called!n");

    return(0);
}

int close(struct cdev *dev, int flag, int otyp, struct thread *td)
{
    printf("[cd_example]Close Called!n");
    return(0);
}

int write(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;

    error = copyinstr(uio->uio_iov->iov_base, &buf, 512, &len);
    if (error != 0)
        printf("[cd_example]Write to "cd_example" failed.n");
    else
        printf("[cd_example]Write Called! String is:%sn", buf);

    return(error);
}

int read(struct cdev *dev, struct uio *uio, int ioflag)
{
    int error = 0;

    if (len <= 0) {
        printf("[cd_example]Read Called! ERROR: Length is invalid!n");
        error = -1;
    }

    else {
        /* Return the saved character string to userland. */
    uiomove(&buf, 513, uio);
    printf("[cd_example]Read Called! String is:%sn", buf);
    }

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

Nothing much changed except `uiomove(&buf, 513, uio)`; part.

``` c++
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

## The Application

Same as the relationship between system call and its application, you can
consider this as C/S model, one service, one client. There’s nothing tricky in
this client application code, except that we used a file descriptor to open our
device interface under DEVFS, and `write()`, `read()`, `open()`, `close()`
functions to interact with it.

``` c++
#include <stdio.h>
#include <stdlib.h> //Add this to avoid warning
#include <fcntl.h>
#include <paths.h>
#include <string.h>
#include <sys/types.h>

#define CDEV_DEVICE "cd_example"
static char buf[512+1];

int main(int argc, char*argv[])
{
    if (argc != 2) {
        printf("Usage:n%s <string>n", argv[0]);
        exit(0);
    }

    int kernel_fd; //Kernel File Descriptor
    int len;

    /* Open cd_example. */
    if ((kernel_fd = open("/dev/" CDEV_DEVICE, O_RDWR)) == -1) {
        perror("/dev/" CDEV_DEVICE); exit(1);
    }

    if ((len = strlen(argv[1]) + 1) > 512) {
        printf("ERROR: String too longn"); exit(0);
    }

    /* Write to cd_example. */
    if (write(kernel_fd, argv[1], len) == -1)
        perror("write()");
    else
        printf("Wrote "%s" to device /dev/" CDEV_DEVICE ".n", argv[1]);

    /* Read from cd_example. */
    if (read(kernel_fd, buf, len) == -1)
        perror("read()");
    else
        printf("Read "%s" from device /dev/" CDEV_DEVICE ".n", buf);

    /* Close cd_example. */
    if ((close(kernel_fd)) == -1) {
        perror("close()");
        exit(1);
    }

    exit(0);
}
```

``` shell
ZFSTest# gcc -o testcdev testcdev.c
ZFSTest# ./testcdev Having fun with cdev
Wrote "Having fun with cdev" to device /dev/cd_example.
Read "Having fun with cdev" from device /dev/cd_example.
ZFSTest# dmesg | tail -n 5
--- syscall (5, FreeBSD ELF64, open), rip = 0x800732abc, rsp = 0x7ffffffbc058, rbp = 0x7ffffffe6089 ---
[cd_example]Open Called!
[cd_example]Write Called! String is:Having fun with cdev
[cd_example]Read Called! String is:Having fun with cdev
[cd_example]Close Called!
ZFSTest#
```

Pretty straightforward code, so let's keep this note simple. Let's jump right
into the next note, see you there!
