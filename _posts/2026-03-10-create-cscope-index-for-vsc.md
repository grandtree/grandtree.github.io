---
layout: post
title:  "Create cscope index for VSC"
tags: VSC, Vim, cscope
---

# Problem

I have been using Vim for years and always build code indexes for my projects
with cscope. But this is the AI era, and a coding assistant is a must-have
companion, so I started using VSC (I will not give up Vim, never!) because
talking to it is so much more convenient.

There are multiple ways to use a code index in VSC. For C/C++, there are
cpptools and clangd, but usually they need compile flags as input, and
collecting them is an intimidating task for big projects, especially when
they don't use gmake or cmake, or their makefile is a monster. So I prefer
the "old-school" way: cscope.

The `cscope-code` extension of VSC is good enough, but again, its automatic
index generation causes trouble in big projects like the Linux kernel source
tree. Usually I don't care about most architectures except for x86 and arm64,
and indexing all the code is slow and costs extra space on the internal SSD
of my laptop.

# Solution

Generating the cscope index manually gives the most flexibility. But before
that, let me dig into how `cscope-code` works.

## Under the hood of cscope-code

Below is the log excerpted from the output panel, showing how the index was
built:

```text
2026-02-14 20:15:32.499 [info] I: queue state 0 fulfilled
2026-02-14 20:15:32.505 [info] I: start to build

2026-02-14 20:15:32.505 [info] I: cscope ["-RbUq","-f","cscope.out"] /git_workspace/ddos/os/linux-5.4

2026-02-14 20:16:36.901 [info] I: stdout: 0
2026-02-14 20:16:36.902 [info] I: done to build: success
...
```

Below is how the index was used by `cscope-code`:

```text
2026-02-14 21:57:10.958 [info] I: cscope ["-RdL","-f","cscope.out","-1","bnx2x_func_tx_start_params"] /git_workspace/ddos/os/linux-5.4

2026-02-14 21:57:10.989 [info] I: stdout: 0

drivers/net/ethernet/broadcom/bnx2x/bnx2x_sp.h bnx2x_func_tx_start_params 1255 struct bnx2x_func_tx_start_params {

2026-02-14 21:57:10.989 [info] I: results: [{"file":"drivers/net/ethernet/broadcom/bnx2x/bnx2x_sp.h","func":"bnx2x_func_tx_start_params","line":1254,"text":"struct bnx2x_func_tx_start_params {","raw":"drivers/net/ethernet/broadcom/bnx2x/bnx2x_sp.h bnx2x_func_tx_start_params 1255 struct bnx2x_func_tx_start_params {"}]
```

The index created by `cscope-code` works with the cscope of this Linux
machine too:

```sh
quj@qujslesdev:/git_workspace/ddos/os/linux-5.4$ /usr/bin/cscope -RdL -f cscope.out -1 bnx2x_func_tx_start_params

drivers/net/ethernet/broadcom/bnx2x/bnx2x_sp.h bnx2x_func_tx_start_params 1255 struct bnx2x_func_tx_start_params {

quj@qujslesdev:/git_workspace/ddos/os/linux-5.4$
```

The most important takeaways are:

- the cscope index lives at `<root of workspace>/cscope.out`
- the command behind it is `cscope -RbUq -f cscope.out <root of workspace>`

## My own way

This is how I build the cscope index on my Mac for the Linux kernel source
tree.

Create the file list, with some architectures and paths excluded:

```sh
qy@YiMBA2025:~/workspace/mounted_git_repos/linux-5.4$ find * \
-path "arch/*" ! -path "arch/x86*" -prune -o -path "tmp*" \
-prune -o -path "Documentation*" -prune -o -path "scripts*" \
-prune -o -name "*.[chxsS]" -print > cscope.files
```

The file count is not that big anymore, given how huge the Linux kernel
source tree is:

```sh
qy@YiMBA2025:~/workspace/mounted_git_repos/linux-5.4$ wc -l cscope.files
   38875 cscope.files
```

Finally, build the index:

```sh
qy@YiMBA2025:~/workspace/mounted_git_repos/linux-5.4$ time /opt/local/bin/cscope \
-RbUq -f cscope.out -i cscope.files

real    0m49.216s
user    0m40.916s
sys     0m3.358s

qy@YiMBA2025:~/workspace/mounted_git_repos/linux-5.4$
```

Less than a minute — old-school, but still faster than begging the "smart"
tools for compile flags. Usually VSC needs less than one minute to pick up
this index.

# Reference

[Using Cscope on large projects (example: the Linux kernel)](https://cscope.sourceforge.net/large_projects.html)
