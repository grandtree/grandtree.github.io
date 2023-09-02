---
layout: post
title:  "Build kernel 5.19 on openSUSE Leap 15 SP2"
tags: Linux kernel, Build, SUSE 
---

# Get source code of kernel

Added repo from Suse with latest kernel
```
linux-fy00:~ # zypper addrepo -f http://download.opensuse.org/repositories/Kernel:/HEAD/standard/ kernel-rep
```

Get details of the packages available in this repo:
```
linux-fy00:~ # zypper search -s -r kernel-rep kernel
Loading repository data...
Reading installed packages...

S | Name                            | Type       | Version               | Arch   | Repository
--+---------------------------------+------------+-----------------------+--------+-----------
  | kernel-debug                    | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-debug                    | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-debug                    | srcpackage | 5.19~rc1-1.1.g515f42c | noarch | kernel-rep
...
  | kernel-default-debugsource      | package    | 5.19~rc1-1.1.g515f42c | i586   | kernel-rep
  | kernel-default-devel            | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-default-devel            | package    | 5.19~rc1-1.1.g515f42c | i586   | kernel-rep
  | kernel-default-devel-debuginfo  | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-default-livepatch-devel  | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
  | kernel-default-livepatch-devel  | package    | 5.19~rc1-1.1.g515f42c | i586   | kernel-rep
...
  | kernel-pae-livepatch-devel      | package    | 5.19~rc1-1.1.g515f42c | i686   | kernel-rep
  | kernel-source                   | package    | 5.19~rc1-1.1.g515f42c | noarch | kernel-rep
  | kernel-source                   | srcpackage | 5.19~rc1-1.1.g515f42c | noarch | kernel-rep
  | kernel-source-vanilla           | package    | 5.19~rc1-1.1.g515f42c | noarch | kernel-rep
  | kernel-syms                     | package    | 5.19~rc1-1.1.g515f42c | x86_64 | kernel-rep
...
```

Installed the kernel source of 5.19
```
linux-fy00:/usr/src/packages # zypper install kernel-source=5.19~rc1-1.1.g515f42c
```

# Get kernel configuration file
Installed the kernel dev package of default flavor, no need to worry about the conflict, I only needed the kernel config file.
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # zypper install  kernel-default-devel=5.19~rc1-1.1.g515f42c
...
Problem: nothing provides 'libc.so.6(GLIBC_2.34)(64bit)' needed by the to be installed kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64
 Solution 1: do not install kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64
 Solution 2: break kernel-default-devel-5.19~rc1-1.1.g515f42c.x86_64 by ignoring some of its dependencies

Choose from above solutions by number or cancel [1/2/c/d/?] (c): 2
Resolving dependencies...
Resolving package dependencies...
...
```

Then, I copy the config file of default flavor to our build directory
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls /usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config
/usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # cp /usr/src/linux-5.19.0-rc1-1.g515f42c-obj/x86_64/default/.config ./
```

Now, it is time to config kernel, here, I don't tune anything, or you need to use "menuconfig" instead of "oldconfig":
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # make -C /usr/src/linux O=$PWD oldconfig
```
## Issue related to /bin/bash
Create link from /usr/bin/bash to /bin/bash, because some script needs /usr/bin/bash
```
linux-fy00:/usr/bin # ln -s /bin/bash ./
```
Or we will get error below:
```
  WRAP    arch/x86/include/generated/asm/mcs_spinlock.h
  WRAP    arch/x86/include/generated/asm/irq_regs.h
  WRAP    arch/x86/include/generated/asm/kmap_size.h
  WRAP    arch/x86/include/generated/asm/local64.h
  WRAP    arch/x86/include/generated/asm/mmiowb.h
  WRAP    arch/x86/include/generated/asm/module.lds.h
  WRAP    arch/x86/include/generated/asm/rwonce.h
  WRAP    arch/x86/include/generated/asm/unaligned.h
  UPD     include/config/kernel.release
  UPD     include/generated/uapi/linux/version.h
  UPD     include/generated/utsrelease.h
  UPD     include/generated/uapi/linux/suse_version.h
  CC      scripts/mod/empty.o
/bin/sh: /usr/src/linux-5.19.0-rc1-1.g515f42c/scripts/check-local-export: /usr/bin/bash: bad interpreter: No such file or directory
make[1]: *** [/usr/src/linux-5.19.0-rc1-1.g515f42c/scripts/Makefile.build:250: scripts/mod/empty.o] Error 126
make[1]: *** Deleting file 'scripts/mod/empty.o'
make: *** [/usr/src/linux-5.19.0-rc1-1.g515f42c/Makefile:1209: prepare0] Error 2
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls /usr/bin/bash
ls: cannot access '/usr/bin/bash': No such file or directory
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # which bash
/bin/bash
```

## Issue related to certificate 
We also need to generate the file containing private key and certificate for signing kernel module, or we will get error below:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # make
...
  CC      certs/system_keyring.o
  HOSTCC  certs/extract-cert
  CERT    certs/x509_certificate_list
make[1]: *** No rule to make target '.kernel_signing_key.pem', needed by 'certs/signing_key.x509'.  Stop.
make: *** [/usr/src/linux-5.19.0-rc1-1.g515f42c/Makefile:1853: certs] Error 2
```
In fact, kernel build system should generate this key file for me, but here it fails for some reason. 

Refer to Build kernel 15.9 on OpenSUSE Leap 15 SP2 for kernel module signature feature:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # grep CONFIG_MODULE_SIG .config        
CONFIG_MODULE_SIG_FORMAT=y
CONFIG_MODULE_SIG=y
# CONFIG_MODULE_SIG_FORCE is not set
# CONFIG_MODULE_SIG_ALL is not set
# CONFIG_MODULE_SIG_SHA1 is not set
# CONFIG_MODULE_SIG_SHA224 is not set
CONFIG_MODULE_SIG_SHA256=y
# CONFIG_MODULE_SIG_SHA384 is not set
# CONFIG_MODULE_SIG_SHA512 is not set
CONFIG_MODULE_SIG_HASH="sha256"
CONFIG_MODULE_SIG_KEY=".kernel_signing_key.pem"
CONFIG_MODULE_SIG_KEY_TYPE_RSA=y
# CONFIG_MODULE_SIG_KEY_TYPE_ECDSA is not set
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c #
```
Here, we need to generate our own key file firstly, then tell kernel build system to use this one. 

Firstly, let us get the configuration file for generation of key file, here, I create this file according to man page of openssl-req:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # cat ../x509.genkey                    
[ req ]
distinguished_name = req_distinguished_name
default_bits = 4096
prompt = no
[ req_distinguished_name ]
#O = Unspecified company
CN = Build time kernel key
#emailAddress = unspecified.user@unspecified.company
```
under crets/ of kernel source tree, there is also one default configuration file: default_x509.genkey
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls /usr/src/linux/certs
Kconfig      blacklist_hashes.c    default_x509.genkey        system_keyring.c
Makefile     blacklist_nohashes.c  extract-cert.c
blacklist.c  common.c              revocation_certificates.S
blacklist.h  common.h              system_certificates.S
```
generate key file:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # openssl req -new -nodes -utf8 -sha256 -days 36500 -batch -x509    -config ../x509.genkey -outform PEM -out kernel_key.pem    -keyout kernel_key.pem
Generating a RSA private key
......................................................................................................................................................++++
.................................++++
writing new private key to 'kernel_key.pem'
-----
```
finally, point it out to kernel config file:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls $(pwd)/kernel_key.pem
/root/build_kernel/linux-5.19.0-rc1-1.g515f42c/kernel_key.pem
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # vim .config
...
CONFIG_MODULE_SIG_KEY="/root/build_kernel/linux-5.19.0-rc1-1.g515f42c/kernel_key.pem"
...
".config" 11025L, 262180C written
```

# Build kernel
Now, it is time to build kernel
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # make -C /usr/src/linux O=$PWD
```

## Issue related to pahole
Building kernel needs various tools, some of them are listed in the dependency of 2 packages installed above, some of them are built firstly as part of building kernel, but others may only be found after build failure. In this practice, I encountered this issue:
```
  AS      arch/x86/lib/iomap_copy_64.o
  AR      arch/x86/lib/built-in.a
  GEN     .version
  CHK     include/generated/compile.h
  LD      vmlinux.o
  MODPOST vmlinux.symvers
  MODINFO modules.builtin.modinfo
  GEN     modules.builtin
  CC      .vmlinux.export.o
BTF: .tmp_vmlinux.btf: pahole (pahole) is not available
Failed to generate BTF for vmlinux
Try to disable CONFIG_DEBUG_INFO_BTF
make[1]: *** [/usr/src/linux-5.19.0-rc1-1.g515f42c/Makefile:1174: vmlinux] Error 1
make[1]: Leaving directory '/root/build_kernel/linux-5.19.0-rc1-1.g515f42c'
make: *** [Makefile:219: __sub-make] Error 2
make: Leaving directory '/usr/src/linux-5.19.0-rc1-1.g515f42c'
```

On OpenSUSE, it is simple:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # zypper search -f pahole
...
S  | Name                 | Summary                                                 | Type
---+----------------------+---------------------------------------------------------+--------
   | dwarves              | DWARF utilities                                         | package
i+ | kernel-default-devel | Development files necessary for building kernel modules | package
i  | kernel-devel         | Development files needed for building kernel modules    | package
...
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # zypper install dwarves
```

# Install kernel modules
Now, let us install external modules:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # make -C /usr/src/linux O=$PWD modules_install
```

## Issue related to /usr/lib/modules and /lib/modules
External kernel modules should be installed to /lib/modules, but I found that in my practice kernel build system installed them to /usr/lib/modules, but used /lib/modules at the end, so it failed because there was nothing. The workaround is to create link from /usr/lib/modules to /lib/modules:
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # pushd /usr/lib
linux-fy00:/usr/lib # ln -s /lib/modules/ modules
```

# Install kernel
Finally, install kernel image, initrd will also be built and installed automatically
```
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # make -C /usr/src/linux O=$PWD install
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c #
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls /boot/*5.19*
/boot/System.map-5.19.0-rc1-1.g515f42c-default
/boot/System.map-5.19.0-rc1-1.g515f42c-default.old
/boot/config-5.19.0-rc1-1.g515f42c-default
/boot/config-5.19.0-rc1-1.g515f42c-default.old
/boot/initrd-5.19.0-rc1-1.g515f42c-default
/boot/vmlinux-5.19.0-rc1-1.g515f42c-default.gz
/boot/vmlinux-5.19.0-rc1-1.g515f42c-default.gz.old
/boot/vmlinuz-5.19.0-rc1-1.g515f42c-default
/boot/vmlinuz-5.19.0-rc1-1.g515f42c-default.old
linux-fy00:~/build_kernel/linux-5.19.0-rc1-1.g515f42c # ls /boot/*5.3*
/boot/System.map-5.3.18-lp152.106-default   /boot/symvers-5.3.18-lp152.106-default.gz
/boot/config-5.3.18-lp152.106-default       /boot/sysctl.conf-5.3.18-lp152.106-default
/boot/initrd-5.3.18-lp152.106-default       /boot/vmlinux-5.3.18-lp152.106-default.gz
/boot/symtypes-5.3.18-lp152.106-default.gz  /boot/vmlinuz-5.3.18-lp152.106-default
```

# Test the new kernel
On OpenSUSE, the new kernel and initrd can be automatically picked up by Grub 2, so we only need to reboot and check the result:
```
linux-fy00:~ # uname -a
Linux linux-fy00 5.19.0-rc1-1.g515f42c-default #2 SMP PREEMPT_DYNAMIC Thu Jun 9 20:13:35 CST 2022 x86_64 x86_64 x86_64 GNU/Linux
linux-fy00:~ # cat /etc/os-release
NAME="openSUSE Leap"
VERSION="15.2"
ID="opensuse-leap"
ID_LIKE="suse opensuse"
VERSION_ID="15.2"
PRETTY_NAME="openSUSE Leap 15.2"
ANSI_COLOR="0;32"
CPE_NAME="cpe:/o:opensuse:leap:15.2"
BUG_REPORT_URL="https://bugs.opensuse.org"
HOME_URL="https://www.opensuse.org/"
linux-fy00:~ #
```
