---
layout: post
title:  "How to build kernel on SUSE Linux"
tags: Linux kernel, Build, SUSE 
---

# Important Concepts
There are multiple flavors of SUSE kernel, each one corresponds to one set of
kernel configuration items, i.e. one kernel configuration file needed for build.
You can get more details of flavors in [kernel page of openSUSE] [flavors],
usually default is chosen. 


# Best practice
It is better to use one private and dedicated directory for files generated from
build process, so there is no need to work with root privilege or fill git repo
directory with many binary files. 

It is better to create one snapshot, if you were using BTRFS for system disk, at
least before installation of new kernel, so if anything wrong, you have one
snapshot of good shape to boot up system. 

# Prepare the build environment
Of course, packages in pattern devel\_basis and devel\_kernel are are needed.
They inlcude must-have tools like gcc, some of them like kernel-source are not
must-have in some cases, for example, if you needed to build kernel source code
from kernel.org, there is no need for kernel-source. Depending on your use
cases, probably you will find that some packages are needed in the midway.  

# Steps
Below are generic steps according to [kernel-source/doc/README.SUSE] [readme],
but I promise that they are not enough, the journy may still be bumpy, you need
to refer to [examples](#examples) 

- Get kernel source code, there are multiple ways:
    - Git clone from [git repo of kernel.org] [linux-git] 
    - Get kernel source RPM from SUSE 
- Create a build directory for use in configuring and building the kernel.
- Configure the kernel.  
- Build the kernel and all its modules ("make").
- Make sure that /etc/modprobe.d/unsupported-modules contains   
   allow\_unsupported\_modules 1   
otherwise modprobe will refuse to load any modules.
- Install the kernel and the modules ("make modules\_install", followed by "make
  install"). This will automatically create  an initrd for the new kernel as well.
- Add the kernel to the boot manager. So far, I find that it can be done
  automatically by "make install" 

# Kernel Configuration file 
Note that there is NO kernel configuration file from kernel.org, so I need to
get my own.

Usually, the best start point is the configuration used by the running system.
The running kernel exposes a gzip compressed version of its configuration file
as /proc/config.gz. The kernel sources can be configured based on
/proc/config.gz with "make silentoldconfig".

I can also get it from kernel dev package of one flavor. The kernel-$FLAVOR
packages are based on config/$ARCH/$FLAVOR. (kernel-default is based on
config/$ARCH/default, for example). The kernel-$FLAVOR packages install their
configuration files as /boot/config-$VER\_STR-$FLAVOR (for example,
/boot/config-2.6.5-99-default). The config is also packaged in the
kernel-$FLAVOR-devel package as /usr/src/linux-obj/$ARCH/$FLAVOR/.config.

```
linux-fy00:~ # rpm -ql kernel-default-5.3.18-lp152.106.1.x86_64
...
/boot/config-5.3.18-lp152.106-default
...
linux-fy00:~ # rpm -ql  kernel-default-devel
...
/usr/src/linux-5.3.18-lp152.106-obj/x86_64/default/.config
...
```


The most flexibile way is to get it from repo of openSUSE, the advantage is that
in this way, I can get it of any kernel version because the dev work of openSUSE
Tumbleweed always tracks progress of kernel


# Troubleshooting
Go back to old kernel if boot up failed provided that you have a backup or
snapshot.

For issue related to /bin/bash, refer to [Build kernel 5.19 on openSUSE Leap 15
SP2][build-kernel-5.19-leap-15-sp2]  

For issue related to certificate, refer to [Build kernel 5.19 on openSUSE Leap 15
SP2][build-kernel-5.19-leap-15-sp2]  

For issue related to /usr/lib/modules and /lib/modules, refer to [Build kernel
5.19 on openSUSE Leap 15 SP2][build-kernel-5.19-leap-15-sp2]  

For issue related to pahole, refer to [Build kernel
5.19 on openSUSE Leap 15 SP2][build-kernel-5.19-leap-15-sp2]  




# Examples
Examples below cover various combinations of cases mentioned above:

- Build from git repo of kernel.org:   
    - [Build kernel 5.19 on openSUSE Leap 15 SP3][build-kernel-5.19-leap-15-sp3]
-  Build from kernel source RPM package   
    - [Build kernel 4.4 on SLES 12 SP3][build-kernel-4.4-sles-12-sp3]: in this
    case, the kernel version is the same one of build machine
	- [Build kernel 5.19 on openSUSE Leap 15
	SP2][build-kernel-5.19-leap-15-sp2], which by default uses kernel 5.3: in this
	case, the kernel version was the newest one (at that time, 5.19)




# References
[kernel-source/doc/README.SUSE] [readme] is definitely the most authorative doc

[How SUSE builds its Enterprise Linux distribution – PART 4] [versions] has
mapping table for kernel versions and SLES versions (until SLES 15 SP4) 

[Git repo of kernel.org] [linux-git]

[readme]:https://github.com/openSUSE/kernel-source/blob/master/doc/README.SUSE 

[versions]: https://www.suse.com/c/how-suse-builds-its-enterprise-linux-distribution-part-4/

[linux-git]: https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/

[build-kernel-4.4-sles-12-sp3]: http://localhost:4000/2023/02/22/Build-kernel-4-4-sles-12-sp3.html

[build-kernel-5.19-leap-15-sp3]: http://localhost:4000/2023/02/22/Build-kernel-5-19-leap-15-sp3.html

[build-kernel-5.19-leap-15-sp2]: http://localhost:4000/2023/02/22/Build-kernel-5-19-leap-15-sp2.html

[flavors]: https://en.opensuse.org/Kernel
