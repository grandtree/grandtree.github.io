<!--
It is based on my note 'Dive into the output of CPP for kernel source code'
-->
---
layout: post
title: "Dive into the output of CPP for kernel source code"
tags: Linux kernel, Build
---

Kernel development often means wandering through a maze of header files,
macros, and generated definitions. Sometimes the hardest part is simply
figuring out what the C preprocessor has done to your source code.

This post shows how to ask the preprocessor to reveal its work.

# How to get the CPP output

The kbuild system can generate preprocessor output directly. You only need
to build the target file with a `.i` extension.

In the example below, we build `test_panic.c` in the `crash_mod`
directory. It is a small kernel module used for tutorial purposes, and the
generated output is saved as `test_panic.i`.

```sh
[quj@jacobqu_centos_Linux crash_mod]$ PLAT_OS=/auto/home2/quj/workspace/KU44/prod/li_6/platform/os make -C /auto/home2/quj/workspace/KU44/prod/li_6/platform/os/linux-4.4 M=$(pwd) V=1 test_panic.i
make: Entering directory `/auto/home2/quj/workspace/KU44/prod/li_6/platform/os/linux-4.4'
make -f ./scripts/Makefile.build obj=/auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod /auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.i
  /auto/home/lsbuild/desktop-578094/usr/bin/gcc -E -Wp,-MD,/auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/.test_panic.i.d  -nostdinc -isystem /auto/toolset_nfs/toolchain/desktop-578094/usr/bin/../lib64/gcc/x86_64-unknown-linux-gnu/4.8.3/include -I/auto/home2/quj/workspace/KU44/prod/li_6/platform/os/include -I./arch/x86/include -Iarch/x86/include/generated/uapi -Iarch/x86/include/generated  -Iinclude -I./arch/x86/include/uapi -Iarch/x86/include/generated/uapi -I./include/uapi -Iinclude/generated/uapi -include ./include/linux/kconfig.h -D__KERNEL__ -Wall -Wundef -Wstrict-prototypes -Wno-trigraphs -fno-strict-aliasing -fno-common -Werror-implicit-function-declaration -Wno-format-security -std=gnu89 -fno-PIE -mno-sse -mno-mmx -mno-sse2 -mno-3dnow -mno-avx -m64 -falign-jumps=1 -falign-loops=1 -mno-80387 -mno-fp-ret-in-387 -mpreferred-stack-boundary=3 -march=core2 -mno-red-zone -mcmodel=kernel -funit-at-a-time -maccumulate-outgoing-args -DCONFIG_AS_CFI=1 -DCONFIG_AS_CFI_SIGNAL_FRAME=1 -DCONFIG_AS_CFI_SECTIONS=1 -DCONFIG_AS_FXSAVEQ=1 -DCONFIG_AS_SSSE3=1 -DCONFIG_AS_CRC32=1 -DCONFIG_AS_AVX=1 -DCONFIG_AS_AVX2=1 -DCONFIG_AS_SHA1_NI=1 -DCONFIG_AS_SHA256_NI=1 -pipe -Wno-sign-compare -fno-asynchronous-unwind-tables -fno-delete-null-pointer-checks -Wno-maybe-uninitialized -Os --param=allow-store-data-races=0 -DCC_HAVE_ASM_GOTO -Wframe-larger-than=2048 -fno-stack-protector -Wno-unused-but-set-variable -fno-omit-frame-pointer -fno-optimize-sibling-calls -fno-var-tracking-assignments -g -pg -mfentry -DCC_USING_FENTRY -Wdeclaration-after-statement -Wno-pointer-sign -fno-strict-overflow -fconserve-stack -Werror=implicit-int -Werror=strict-prototypes  -DMODULE  -D"KBUILD_STR(s)=#s" -D"KBUILD_BASENAME=KBUILD_STR(test_panic)"  -D"KBUILD_MODNAME=KBUILD_STR(test_panic)"   -o /auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.i /auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.c
make: Leaving directory `/auto/home2/quj/workspace/KU44/prod/li_6/platform/os/linux-4.4'
```

# How to read the CPP output

The output from the C preprocessor looks a lot like the original source,
except that preprocessing directives are replaced with blank lines and
comments are turned into spaces. Long runs of blank lines are removed.

Source file names and line numbers are represented by lines like this:

```text
# linenum filename flags
```

These lines are called *linemarkers*. They are inserted where needed and
mean that the following line came from `filename` at line `linenum`.

After the file name, you may see zero or more flags: `1`, `2`, `3`, or `4`.
If there are multiple flags, they are separated by spaces. Here is what
they mean:

- `1` indicates the start of a new file.
- `2` indicates a return to a file after including another one.
- `3` indicates that the following text comes from a system header, so some
  warnings should be suppressed.
- `4` indicates that the following text should be treated as if it were
  wrapped in an implicit `extern "C"` block.

# What can we learn from CPP output?

## Trace the header include chain

When I was reading `linux-4.4/arch/x86/include/asm/bitops.h`, I wondered
where the definition of `U64_C` came from:

```c
#define BIT_64(n)           (U64_C(1) << (n))
```

Its actual definition lives in
`linux-4.4/include/asm-generic/int-ll64.h`:

```c
#define U64_C(x)    x ## ULL
```

So I added the relevant header include to `test_panic.c`:

```c
/*
 * Copyright (c) 2005-2010 Wind River Systems, Inc.
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
 * See the GNU General Public License for more details.
 */
#include <linux/module.h>
#include <linux/delay.h>
#include <linux/moduleparam.h>
#include <linux/types.h>
#include <linux/timer.h>
#include <linux/miscdevice.h>
#include <linux/watchdog.h>
#include <linux/fs.h>
#include <linux/notifier.h>
#include <linux/reboot.h>
#include <linux/init.h>
#include <linux/proc_fs.h>
#include <linux/sched.h>
#include <linux/slab.h>
#include <linux/version.h>
#include <asm/uaccess.h>
#include <linux/bitops.h>
...
```

The build command shown earlier generates output like this. Pay attention to
the comments I added with the `<<quj:>>` prefix:

```text
# 1 "/auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.c"
# 1 "/auto/home2/quj/workspace/KU44/prod/li_6/platform/os/linux-4.4//"
# 1 "<built-in>"
# 1 "<command-line>" <<quj: the header file included in command line is given the line number 1 of original source file.
# 1 "././include/linux/kconfig.h" 1 <<quj: this is included from the command line, content below are from this kconfig.h
<<quj: 1 of 3 lines from kconfig.h, it is "#ifndef __LINUX_KCONFIG_H"
<<quj: 1 of 3 lines from kconfig.h, it is "#define __LINUX_KCONFIG_H", note that all the macros are expanded in the final .i file, so the 2 lines are not shown here
<<quj: 1 of 3 lines from kconfig.h, it is one blank line
# 1 "include/generated/autoconf.h" 1 << quj: enter a new header file, this file only contains many macros, so no content is shown here.
# 5 "././include/linux/kconfig.h" 2 << quj: back to kconfig.h
# 1 "<command-line>" 2 << quj: back to the command line
# 1 "/auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.c"
# 13 "/auto/home2/quj/my_repo/sample_code/Linux_kernel/misc_modules/crash_mod/test_panic.c" << quj: module.h is included on line 13 of test_panic.c
# 1 "include/linux/module.h" 1 << quj: enter a new header file
# 9 "include/linux/module.h" << quj: list.h is included on line 9 of module.h
# 1 "include/linux/list.h" 1



# 1 "include/linux/types.h" 1




# 1 "include/uapi/linux/types.h" 1



# 1 "./arch/x86/include/uapi/asm/types.h" 1



# 1 "./include/uapi/asm-generic/types.h" 1





# 1 "include/asm-generic/int-ll64.h" 1
```

That lets us reconstruct the include chain:

- `include/linux/module.h`
- `include/linux/list.h`
- `include/linux/types.h`
- `include/uapi/linux/types.h`
- `arch/x86/include/uapi/asm/types.h`
- `include/uapi/asm-generic/types.h`
- `include/asm-generic/int-ll64.h`

## Check the final definition

In kernel 4.4, `spinlock_t` is defined like this:

```c
typedef struct spinlock {
    union {
        struct raw_spinlock rlock;

#ifdef CONFIG_DEBUG_LOCK_ALLOC
# define LOCK_PADSIZE (offsetof(struct raw_spinlock, dep_map))
        struct {
            u8 __padding[LOCK_PADSIZE];
            struct lockdep_map dep_map;
        };
#endif
    };
} spinlock_t;
```

But is `CONFIG_DEBUG_LOCK_ALLOC` actually enabled in your kernel build?

Search for `spinlock_t` in the generated `.i` file and you will find this:

```c
# 81 "include/linux/spinlock_types.h"
typedef struct spinlock {
 union {
  struct raw_spinlock rlock;
# 92 "include/linux/spinlock_types.h"
 };
} spinlock_t;
```

The conditional block guarded by `CONFIG_DEBUG_LOCK_ALLOC` is gone, which
means that macro was not defined for this build.
