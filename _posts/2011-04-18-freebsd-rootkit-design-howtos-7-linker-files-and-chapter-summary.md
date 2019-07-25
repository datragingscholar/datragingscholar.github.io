---
title        : "FreeBSD Rootkit Design Howtos - 7 - Linker Files and Chapter Summary"
date         : "2011-04-18"
tags         : [FreeBSD, Rootkit, C++]
---

The book allocated a short section to describe the `kldstat` command once more
before it ends the first chapter. So I’ll do this as well but with a little more
information.

## Linker Files

As we already know, that the `kldstat` command can give us a whole list of all
loaded modules with their id, address, size, and module name. This is a simple
and powerful tool for sysadmins, developers, and all kinds of other users,
except that this might not be a lovely tool for rootkit developers (or let’s say
low-level rootkit developers), ’cause people can easily ***detect*** your
rootkit with only one command. But you know what, don’t you be worried, there
are a whole bunch of techniques that the book will tell us to avoid your rootkit
being listed in that list.

Let’s take a look at the manual page of `kldstat`.

``` shell
     kldstat [-v] [-i id] [-n filename]
     kldstat [-q] [-m modname]
....

DESCRIPTION
     The kldstat utility displays the status of any files dynamically linked
     into the kernel.

     The following options are available:

     -v        Be more verbose.

     -i id     Display the status of only the file with this ID.

     -n filename
               Display the status of only the file with this filename.

     -q        Only check if module is loaded or compiled into the kernel.

     -m modname
               Display the status of only the module with this modname.
```

Let's see some examples.

``` shell
ZFSTest# kldstat -v
 1   22 0xffffffff80100000 cfc488   kernel (/boot/kernel/kernel)
    Contains modules:
        230 nfs
        235 ufs
         1 cam
        50 ata
        10 pass
         4 pmp
...
...

 2    1 0xffffffff81012000 169d25   zfs.ko (/boot/kernel/zfs.ko)
    Contains modules:
        Id Name
        285 zfsctrl
        286 zfs
        288 zfs_vdev
        287 zfs_zvol
 3    1 0xffffffff8117c000 20f7     opensolaris.ko (/boot/kernel/opensolaris.ko)
    Contains modules:
        Id Name
        284 opensolaris
 4    1 0xffffffff8117f000 4050     linprocfs.ko (/boot/kernel/linprocfs.ko)
    Contains modules:
        Id Name
        290 linprocfs
 5    1 0xffffffff81184000 1dd6d    linux.ko (/boot/kernel/linux.ko)
    Contains modules:
        Id Name
        289 linuxelf
 6    1 0xffffffff811a2000 67e      cd_example.ko (./cd_example.ko)
    Contains modules:
        Id Name
        291 cd_example
```

``` shell
ZFSTest# kldstat -v -i 6
Id Refs Address            Size     Name
 6    1 0xffffffff811a2000 67e      cd_example.ko (./cd_example.ko)
    Contains modules:
        Id Name
        291 cd_example
```

``` shell
ZFSTest# kldstat -m cd_example
Id  Refs Name
291    1 cd_example
```

``` shell
ZFSTest# kldstat -n cd_example.ko
Id Refs Address            Size     Name
 6    1 0xffffffff811a2000 67e      cd_example.ko

```

Notice the `Contains modules` output, which can be considered as the kernel
module's submodules. So here is the perfect oppoturnity to introduce `linker
files`.

* Let’s use `zfs.ko` as an example. The module file, zfs.ko, is actually the
  linker file for ZFS module and guides the module to be registered and linked
  with the kernel.

* So that the parameters for kldload and kldunload are actually linker file
  names, NOT the module itself.

* Submodules like zfsctrl is considered as the module, not a linker file anymore
  (Once it gets linked with kernel)

* As such, every module is associated with one linker file, and one linker file
  can contain one or more than one module(s).

It is quite common to divide functionalities into different modules and use one
linker file to register them with the kernel, we will discuss this later on.

## Chapter Summary

Let’s take a look back to what we have learned in the first chapter, and list
down all important points that are crucial to continue our discussions. Here’s a
list of all the notes we have for the first chapter.

[FreeBSD Rootkit Design Howtos - 1 - KLD First Kernel Loadable Module]({%post_url
2011-04-02-freebsd-rootkit-design-howtos-1-kld-first-kernel-loadable-module
%}).

* We started with the generic KLD with a simple ‘hello world!’ printing module,

* and learned the concept of event handler using `modeventhand_t(sys/module.h)`
  with `modeventtype(sys/module.h)` provided,

* and finally the the method to declare our module with
  `DECLARE_MODULE(sys/module.h)` with `moduledata(sys/module.h)`,
  `order(sys/kernel.h)`, and `sub(sys/kernel.h)` as parameters.

[FreeBSD Rootkit Design Howtos - 2 - System Call First Kernel Service Module]({%post_url
2011-04-04-freebsd-rootkit-design-howtos-2-system-call-first-kernel-service-module
%}).

* And then we started with system calls.

* We get `sy_call_t(sys/sysent.h)` as system call function prototype,

* and the `SYSCALL_MODULE` macro to register the module with

* name, the general name for the module

* `offset(sys/sys/sysent.h)`, the offset of the system call’s sysent structure.

* `new_sysent(sys/sys/sysent.h)`, the sysent pointer to the complete sysent
  structure for the system call module.

* evh, the event handler

* and arg, the arguments

* a `sysent(sys/sysent.h)` structure contains at least the number of arguments,
  and the function pointer to the implementation function.

[FreeBSD Rootkit Design Howtos - 3 - System Call First Kernel Service Application]({%post_url
2011-04-06-freebsd-rootkit-design-howtos-3-system-call-first-kernel-service-application
%}).

* Then we started with the application side of system call programming, which
  can be used to call the kernel service,

* Firstly we need to find the module id with `modfind(sys/module.h)` function,

* We can then get the `module_stat(sys/module.h)` back once we have the module
  id,

* The `module_stat(sys/module.h)` structure contains a
  `modspecific_t(sys/module.h)`, namely moduledata,

* The intval is then retrieved from `modspecific_t(sys/module.h)` structure as
  the module offset value,

* Then we can issue a syscall function with module offset value specified to
  call the kernel service.

[FreeBSD Rootkit Design Howtos - 4 - Kernel and User Space Transitions]({%post_url
2011-04-07-freebsd-rootkit-design-howtos-4-kernel-and-user-space-transitions
%}).

->We learned 4 new functions which are defined in `sys/systm.h`, `copyin()`,
`copyout()`, `copyinstr()`, and `copystr()`.

->They can all be used to copy data from a continuous memory address to another,

->And all but `copystr()` copy data from user-space to kernel-space or
vice-versa.

->These functions (except `copystr()`) are designed to copy data between
user-space and kernel-space safely.

->Directly use reference to user-space at kernel-space or vice-versa is
dangerous.

[FreeBSD Rootkit Design Howtos - 5 - Character Device First cdev Module]({%post_url
2011-04-08-freebsd-rootkit-design-howtos-5-character-device-first-cdev-module
%}).

* A new kernel module is introduced here, the character device module, it looks
  similar but not a system call module.

* A `cdevsw(sys/conf.h)` known as character device switch table is needed when
  program a character device.

* The cdevsw contains all basic info such as name and version, and also
  supported functions like open, read, and write.

* The present of d_version, and d_name is a minimal requirement for a cdevsw
  structure.

* A bunch of useful function prototypes are defined in `sys/conf.h` such `as
  d_open_t()`, `d_close_t()`, `d_read_t()`, and so on.

* A `cdev(sys/conf.h)` structure is required to register the device module with
  kernel

* `make_dev(sys/conf.h)` and `destroy_dev(sys/conf.h)` are used to register or
  unregister the module with cdev as reference.

* The `DEV_MODULE(sys/conf.h)` Macro is then used to register the character
  device module with kernel.

[FreeBSD Rootkit Design Howtos - 5 - Character Device First cdev Module]({%post_url
2011-04-13-freebsd-rootkit-design-howtos-6-character-device-first-cdev-application
%}).

* This is rather a simple note,

* We started with a refined version of previous character device module program
  by using `copyinstr()` instead of `copystr()` to make a safer transaction
  between kernel-space and user-space.

* And then a client application to interact with character device is introduced.

* `open()`, `read()`, `write()`, and `close()` were used to manipulate the
  character device module.

All right, this is pretty much everything we’ve covered in Chapter 1, so if any
of these terms look strange to you, please do not hesitate to go back and review
the note.
