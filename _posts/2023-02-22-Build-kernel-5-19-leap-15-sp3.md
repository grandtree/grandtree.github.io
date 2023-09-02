---
layout: post
title:  "Build kernel 5.19 on openSUSE Leap 15 SP3"
tags: Linux kernel, Build, SUSE 
---

# Get source code of kernel
Create a directory dedicated for this task, sorry for the "`15_9`" in my test, it should have been "`5_19`" 

    linux-fy00:~ # mkdir build_kernel_15_9
    linux-fy00:~ # cd build_kernel_15_9

Clone kernle source code from kernel.org

    linux-fy00:~/build_kernel_15_9 # mkdir linux-5.19
    linux-fy00:~/build_kernel_15_9 # cd linux-5.19
    linux-fy00:~/build_kernel_15_9/linux-5.19 # git clone https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
    Cloning into 'linux-stable'...
    remote: Enumerating objects: 53, done.
    remote: Counting objects: 100% (53/53), done.
    remote: Compressing objects: 100% (3/3), done.
    Receiving objects: 100% (10522594/10522594), 4.06 GiB | 1014.00 KiB/s, done.
    remote: Total 10522594 (delta 51), reused 50 (delta 50), pack-reused 10522541
    Resolving deltas: 100% (8403906/8403906), done.
    Updating files: 100% (76950/76950), done.

Choose one release of 5.19 for build

    linux-fy00:~/build_kernel_15_9/linux-5.19 # cd linux-stable/
    linux-fy00:~/build_kernel_15_9/linux-5.19/linux-stable # git tag |grep 5.19
    v2.6.25.19
    v5.15.19
    v5.19-rc1
    v5.5.19
    linux-fy00:~/build_kernel_15_9/linux-5.19/linux-stable # git branch
    * master
    linux-fy00:~/build_kernel_15_9/linux-5.19/linux-stable # git checkout -b my_v5.19 v5.19-rc1
    Switched to a new branch 'my_v5.19'
    linux-fy00:~/build_kernel_15_9/linux-5.19/linux-stable # git log -1
    commit f2906aa863381afb0015a9eb7fefad885d4e5a56 (HEAD -> my_v5.19, tag: v5.19-rc1, origin/master, origin/HEAD, master)
    Author: Linus Torvalds <torvalds@linux-foundation.org>
    Date:   Sun Jun 5 17:18:54 2022 -0700
    
        Linux 5.19-rc1
    linux-fy00:~/build_kernel_15_9/linux-5.19/linux-stable # cd ..
    linux-fy00:~/build_kernel_15_9/linux-5.19 # cd ..
    linux-fy00:~/build_kernel_15_9 # mkdir 5.19-rc1

# Get kenrel configuration file
I was working on an openSUSE Leap 15.3 machine 

    linux-fy00:~/build_kernel_15_9 # uname -a
    Linux linux-fy00 5.3.18-59.10-default #1 SMP Fri Jun 25 12:36:56 UTC 2021 (6856d31) x86_64 x86_64 x86_64 GNU/Linux
    linux-fy00:~/build_kernel_15_9 # cat /etc/os-release
    NAME="openSUSE Leap"
    VERSION="15.3"
    ID="opensuse-leap"
    ID_LIKE="suse opensuse"
    VERSION_ID="15.3"
    PRETTY_NAME="openSUSE Leap 15.3"
    ANSI_COLOR="0;32"
    CPE_NAME="cpe:/o:opensuse:leap:15.3"
    BUG_REPORT_URL="https://bugs.opensuse.org"
    HOME_URL="https://www.opensuse.org/"

The kernel configuration file was get from package kernel-default-devel on
another machine in my test, of course, you can do it on the build machine. 

First at all, I need to add Zypper repo for kernel development and  install this package from it:
```
hostname1:~ # zypper addrepo -f http://download.opensuse.org/repositories/Kernel:/HEAD/standard/ kernel-rep
hostname1:~ # zypper refresh
...
Do you want to reject the key, trust temporarily, or trust always? [r/t/a/?] (r): a
...
```

To guarantee that we get the package of needed version, let us list all the packages with version information
```
hostname1:~ # zypper search -s -r kernel-rep kernel
Refreshing service 'Basesystem_Module_x86_64'.
Refreshing service 'Python_2_Module_x86_64'.
Refreshing service 'SUSE_Linux_Enterprise_Server_x86_64'.
Refreshing service 'Server_Applications_Module_x86_64'.
Loading repository data...
Reading installed packages...

S | Name                            | Type       | Version               | Arch   | Repository
--+---------------------------------+------------+-----------------------+--------+-----------
  | kernel-debug                    | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-debug                    | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-debug                    | srcpackage | 5.19~rc1-1.1.g515f42c | noarch | kernel-rep
  | kernel-debug-debuginfo          | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-debug-debuginfo          | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-debug-debugsource        | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-debug-debugsource        | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-debug-devel              | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-debug-devel              | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-debug-devel-debuginfo    | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  ...
  | kernel-default-devel            | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-default-devel            | package    | 5.19~rc1-1.1.g515f42c | i586   | kernel-rep
  ...
  | kernel-vanilla-devel-debuginfo  | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-vanilla-livepatch-devel  | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-vanilla-livepatch-devel  | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
...
```

Install kernel-default-devel, the conflict doesn't matter, because we only need
the kernel configuration file of this default flavour:
```
hostname1:~ # zypper install kernel-default-devel=5.19~rc1-1.1.g515f42c
...
Problem: nothing provides 'libc.so.6(GLIBC_2.34)(64bit)' needed by the to be installed kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64
 Solution 1: do not install kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64
 Solution 2: break kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64 by ignoring some of its dependencies

Choose from above solutions by number or cancel [1/2/c/d/?] (c): 2
Resolving dependencies...
Resolving package dependencies...
...
hostname1:~ # ls /usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config
/usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config
```

Tranfer configuration file to this build machine

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # rsync root@anothermachine:/usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config ./
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls
    .config

Almost all material are ready, save the current state in snapshot

    linux-fy00:~/build_kernel_15_9 # snapper create --description "just after add of kernel-rep"
    linux-fy00:~/build_kernel_15_9 # pwd
    /root/build_kernel_15_9
    linux-fy00:~/build_kernel_15_9 # ls
    5.19-rc1  linux-5.19
    linux-fy00:~/build_kernel_15_9 # mv 5.19-rc1/ build_5.19-rc1/
    linux-fy00:~/build_kernel_15_9 # cd build_5.19-rc1/

Configure kernel, here I just used the configuration file get without any change, feel free to tune it, for example with "make menuconfig": 

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # make -C /root/build_kernel_15_9/linux-5.19/linux-stable O=$PWD oldconfig
    make: Entering directory '/root/build_kernel_15_9/linux-5.19/linux-stable'
    make[1]: Entering directory '/root/build_kernel_15_9/build_5.19-rc1'
      GEN     Makefile
      HOSTCC  scripts/basic/fixdep
      HOSTCC  scripts/kconfig/conf.o
      HOSTCC  scripts/kconfig/confdata.o
      HOSTCC  scripts/kconfig/expr.o
      LEX     scripts/kconfig/lexer.lex.c
      YACC    scripts/kconfig/parser.tab.[ch]
      HOSTCC  scripts/kconfig/lexer.lex.o
      HOSTCC  scripts/kconfig/menu.o
      HOSTCC  scripts/kconfig/parser.tab.o
      HOSTCC  scripts/kconfig/preprocess.o
      HOSTCC  scripts/kconfig/symbol.o
      HOSTCC  scripts/kconfig/util.o
      HOSTLD  scripts/kconfig/conf
    #
    # configuration written to .config
    #
    make[1]: Leaving directory '/root/build_kernel_15_9/build_5.19-rc1'
    make: Leaving directory '/root/build_kernel_15_9/linux-5.19/linux-stable'

# Miscs
I found from failures that items below were needed 

/bin/bash is must-have

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls -l /usr/bin/bash
    -rwxr-xr-x 1 root root 1012544 May  5  2021 /usr/bin/bash
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls -l /bin/bash
    lrwxrwxrwx 1 root root 13 May  5  2021 /bin/bash -> /usr/bin/bash

pahole is needed for analysis of binary in build

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # which pahole
    which: no pahole in (/sbin:/usr/sbin:/usr/local/sbin:/root/bin:/usr/local/bin:/usr/bin:/bin)
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # zypper install dwarves
    Loading repository data...
    Reading installed packages...
    Resolving package dependencies...
    
    The following 2 NEW packages are going to be installed:
      dwarves libdwarves1
    ... 
    Checking for file conflicts: ...........................................................[done]
    (1/2) Installing: libdwarves1-1.19-bp153.1.13.x86_64 ...................................[done]
    (2/2) Installing: dwarves-1.19-bp153.1.13.x86_64 .......................................[done]

# Certificate issue 
Certificate should be created automatically, but it failed in my test:

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # make -C /root/build_kernel_15_9/linux-5.19/linux-stable O=$PWD
      AR      kernel/built-in.a
      CC [M]  kernel/torture.o
      CC      certs/system_keyring.o
      HOSTCC  certs/extract-cert
      CERT    certs/x509_certificate_list
    make[2]: *** No rule to make target '.kernel_signing_key.pem', needed by 'certs/signing_key.x509'.  Stop.
    make[1]: *** [/root/build_kernel_15_9/linux-5.19/linux-stable/Makefile:1839: certs] Error 2
    make[1]: Leaving directory '/root/build_kernel_15_9/build_5.19-rc1'
    make: *** [Makefile:219: __sub-make] Error 2
    make: Leaving directory '/root/build_kernel_15_9/linux-5.19/linux-stable'

Let me create certificate with key configuration file from kernel tree

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # cp ~/build_kernel_15_9/linux-5.19/linux-stable/certs/default_x509.genkey ../
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls ../
    build_5.19-rc1  default_x509.genkey  linux-5.19
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # openssl req -new -nodes -utf8 -sha256 -days 36500 -batch -x509 -config ../default_x509.genkey -outform PEM -out kernel_key.pem -keyout kernel_key.pem
    Generating a RSA private key
    ....................................................++++
    .......................................................................................................................................................................++++
    writing new private key to 'kernel_key.pem'
    -----


Change .config file to use the certificate generated above:

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # cp .config .config_orig
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # vim .config
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # diff .config_orig .config
    10408c10408
    < CONFIG_MODULE_SIG_KEY=".kernel_signing_key.pem"
    ---
    > CONFIG_MODULE_SIG_KEY="/root/build_kernel_15_9/build_5.19-rc1/kernel_key.pem"


# Build kernel
Now everyting run smoothly:

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # make -C /root/build_kernel_15_9/linux-5.19/linux-stable O=$PWD     

    make: Entering directory '/root/build_kernel_15_9/linux-5.19/linux-stable'
    make[1]: Entering directory '/root/build_kernel_15_9/build_5.19-rc1'
	...    
      AR      arch/x86/lib/built-in.a
      GEN     .version
      CHK     include/generated/compile.h
      LD      vmlinux.o
      MODPOST vmlinux.symvers
      MODINFO modules.builtin.modinfo
      GEN     modules.builtin
      CC      .vmlinux.export.o
      LD      .tmp_vmlinux.btf
      BTF     .btf.vmlinux.bin.o
      LD      .tmp_vmlinux.kallsyms1
      KSYMS   .tmp_vmlinux.kallsyms1.S
      AS      .tmp_vmlinux.kallsyms1.S
      LD      .tmp_vmlinux.kallsyms2
      KSYMS   .tmp_vmlinux.kallsyms2.S
      AS      .tmp_vmlinux.kallsyms2.S
      LD      vmlinux
      BTFIDS  vmlinux
      SYSMAP  System.map
      SORTTAB vmlinux
      CC      arch/x86/boot/a20.o
      AS      arch/x86/boot/bioscall.o
      CC      arch/x86/boot/cmdline.o
      AS      arch/x86/boot/copy.o
	...    
      BTF [M] sound/x86/snd-hdmi-lpe-audio.ko
      CC [M]  sound/xen/snd_xen_front.mod.o
      LD [M]  sound/xen/snd_xen_front.ko
      BTF [M] sound/xen/snd_xen_front.ko
      CC [M]  virt/lib/irqbypass.mod.o
      LD [M]  virt/lib/irqbypass.ko
      BTF [M] virt/lib/irqbypass.ko
    make[1]: Leaving directory '/root/build_kernel_15_9/build_5.19-rc1'
    make: Leaving directory '/root/build_kernel_15_9/linux-5.19/linux-stable'
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 #

# Install kernel modules
You can just ignore steps from this one if only need to build kernel, or do it on another machine where new kenrel will run.

/usr/lib/modules is must-have

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls /usr/lib/modules
    ls: cannot access '/usr/lib/modules': No such file or directory
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # pushd /usr/lib/
    /usr/lib ~/build_kernel_15_9/build_5.19-rc1
    linux-fy00:/usr/lib # ln -s /lib/modules ./
    linux-fy00:/usr/lib # popd
    ~/build_kernel_15_9/build_5.19-rc1
    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # ls /usr/lib/modules
    4.12.14-lp151.28.91-default  5.3.18-59.10-preempt      5.3.18-lp152.106-preempt
    5.3.18-59.10-default         5.3.18-lp152.106-default

Install kernel modules:

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # make -C /root/build_kernel_15_9/linux-5.19/linux-stable O=$PWD modules_install
    make: Entering directory '/root/build_kernel_15_9/linux-5.19/linux-stable'
    ... 
      INSTALL /lib/modules/5.19.0-rc1-1.g515f42c-default/kernel/virt/lib/irqbypass.ko
      DEPMOD  /lib/modules/5.19.0-rc1-1.g515f42c-default
    make[1]: Leaving directory '/root/build_kernel_15_9/build_5.19-rc1'
    make: Leaving directory '/root/build_kernel_15_9/linux-5.19/linux-stable'

# Install kernel
Install also changed the grub2 configuration.

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # make -C /root/build_kernel_15_9/linux-5.19/linux-stable O=$PWD install
    make: Entering directory '/root/build_kernel_15_9/linux-5.19/linux-stable'
    make[1]: Entering directory '/root/build_kernel_15_9/build_5.19-rc1'
      INSTALL /boot
    dracut: Executing: /usr/bin/dracut --hostonly --force /boot/initrd-5.19.0-rc1-1.g515f42c-default 5.19.0-rc1-1.g515f42c-default
    ...    
    dracut: *** Store current command line parameters ***
    dracut: Stored kernel commandline:
    dracut:  resume=UUID=6c73ea26-0cd3-47f3-beda-d4339e08d0d6
    dracut:  root=UUID=75128fc1-84da-4d3d-9272-fff8b5632bce rootfstype=btrfs
    rootflags=rw,relatime,space_cache,subvolid=268,subvol=/@/.snapshots/1/snapshot,subvol=@/.snapshots/1/snapshot
    dracut: *** Creating image file '/boot/initrd-5.19.0-rc1-1.g515f42c-default'
    ***
    dracut: *** Creating initramfs image file
    '/boot/initrd-5.19.0-rc1-1.g515f42c-default' done ***
    make[1]: Leaving directory '/root/build_kernel_15_9/build_5.19-rc1'
    make: Leaving directory '/root/build_kernel_15_9/linux-5.19/linux-stable'

# Test the new kernel
Now, this machine was running kernel 5.19:

    linux-fy00:~/build_kernel_15_9/build_5.19-rc1 # reboot
    root@quj-dev-centos-7-cec-dur:~# ssh -o PubkeyAuthentication=no -o ServerAliveCountMax=20 -o ServerAliveInterval=15 root@anotehrmachine
    Password:
    Last login: Sat Jun 11 23:25:44 2022 from abc 
    Have a lot of fun...
    linux-fy00:~ # cat /etc/os-release
    NAME="openSUSE Leap"
    VERSION="15.3"
    ID="opensuse-leap"
    ID_LIKE="suse opensuse"
    VERSION_ID="15.3"
    PRETTY_NAME="openSUSE Leap 15.3"
    ANSI_COLOR="0;32"
    CPE_NAME="cpe:/o:opensuse:leap:15.3"
    BUG_REPORT_URL="https://bugs.opensuse.org"
    HOME_URL="https://www.opensuse.org/"
    linux-fy00:~ # uname -a
    Linux linux-fy00 5.19.0-rc1-1.g515f42c-default #1 SMP PREEMPT_DYNAMIC Sun Jun 12 11:54:58 CST 2022 x86_64 x86_64 x86_64 GNU/Linux
    linux-fy00:~ #

