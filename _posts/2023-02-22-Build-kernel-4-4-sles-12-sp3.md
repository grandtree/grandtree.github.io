---
layout: post
title:  "Build kernel 4.4 on SLES 12 SP3"
tags: Linux kernel, Build, SUSE 
---

# Get source code of kenrel
Because SLES 12 SP3 uses kernel 4.4, I can just get the kernel-source package, and there is no need to add a special Zypper repo:
```
linux-3hww:~ # zypper repos
Repository priorities are without effect. All enabled repositories share the same priority.

# | Alias             | Name              | Enabled | GPG Check | Refresh
--+-------------------+-------------------+---------+-----------+--------
1 | SLES12-SP3-12.3-0 | SLES12-SP3-12.3-0 | Yes     | (r ) Yes  | No
2 | product           | product           | Yes     | (r ) Yes  | No
3 | product_source    | product_source    | Yes     | (r ) Yes  | No
4 | sdk               | sdk               | Yes     | (r ) Yes  | No
5 | sdk_source        | sdk_source        | Yes     | (r ) Yes  | No
linux-3hww:~ # zypper search kernel-source
Loading repository data...
Reading installed packages...

S | Name          | Summary                  | Type
--+---------------+--------------------------+-----------
  | kernel-source | The Linux Kernel Sources | package
  | kernel-source | The Linux Kernel Sources | srcpackage
linux-3hww:~ # zypper install kernel-source
Loading repository data...
Reading installed packages...
Resolving package dependencies...
...
```

# Get kernel configuration file
Here, on this machine, this package has been installed:
```
linux-3hww:~ # zypper search kernel-default
Loading repository data...
Reading installed packages...

S  | Name                 | Summary                                               | Type
---+----------------------+-------------------------------------------------------+-----------
i+ | kernel-default       | The Standard Kernel                                   | package
   | kernel-default       | The Standard Kernel                                   | srcpackage
   | kernel-default-base  | The Standard Kernel - base modules                    | package
i  | kernel-default-devel | Development files necessary for building kernel mod-> | package
linux-3hww:~ # ls /usr/src/
linux  linux-4.4.73-5  linux-4.4.73-5-obj  linux-obj  packages
```

# Other preaparation work
Create one directory for output of build while the source code are located under /usr/src:
```
linux-3hww:~ # mkdir build_kernel
linux-3hww:~ # cd build_kernel
linux-3hww:~/build_kernel # ls
linux-3hww:~/build_kernel # mkdir sles_linux-4.4.73-5
```

Create one snapshot of system for backup
```
linux-3hww:~/build_kernel # snapper create --description "just before building SLES linux kernel 4.4.73-5"
```

# Tune kernel configuration
```
linux-3hww:~/build_kernel # cp /usr/src/linux-4.4.73-5-obj/x86_64/default/.config ./
linux-3hww:~/build_kernel # cd sles_linux-4.4.73-5/
linux-3hww:~/build_kernel/sles_linux-4.4.73-5 # mv ../.config ./
linux-3hww:~/build_kernel/sles_linux-4.4.73-5 # make -C /usr/src/linux O=$PWD oldconfig
make: Entering directory '/usr/src/linux-4.4.73-5'
make[1]: Entering directory '/root/build_kernel/sles_linux-4.4.73-5'
  HOSTCC  scripts/basic/fixdep
  GEN     ./Makefile
  HOSTCC  scripts/kconfig/conf.o
  SHIPPED scripts/kconfig/zconf.tab.c
  SHIPPED scripts/kconfig/zconf.lex.c
  SHIPPED scripts/kconfig/zconf.hash.c
  HOSTCC  scripts/kconfig/zconf.tab.o
  HOSTLD  scripts/kconfig/conf
scripts/kconfig/conf  --oldconfig Kconfig
#
# configuration written to .config
#
make[1]: Leaving directory '/root/build_kernel/sles_linux-4.4.73-5'
make: Leaving directory '/usr/src/linux-4.4.73-5'
```

# Build kernel
Now it is time to build kernel. 
```
linux-3hww:~/build_kernel/sles_linux-4.4.73-5 # make -C /usr/src/linux O=$PWD
```
