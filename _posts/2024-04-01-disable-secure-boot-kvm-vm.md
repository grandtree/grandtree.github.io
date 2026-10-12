---
layout: post
title:  "Disable secure boot of VM in KVM"
tags: KVM, UEFI
---

# Problem

By default, a VM created on a KVM host uses EFI with Secure Boot enabled. That
is a pain for OS development. I have hit at least two cases:

- a driver I built myself is not signed
- the installation media of an OS is not signed

I could not find a way to disable Secure Boot in Virtual Machine Manager, so I
had to edit the VM configuration file directly, following information I found
on the web.

# Solution

With the firmware auto-selection feature, libvirt picks a firmware by default,
without any explicit specification. Here, I want the opposite: pin one specific
firmware that comes without Secure Boot.

My host is a Leap 15.5 machine. The firmware description files used by libvirt
for auto-selection can be listed like this:

```
qy@yinspiron:~$ grep -R "description" /usr/share/qemu/firmware
/usr/share/qemu/firmware/50-seabios-256k.json:    "description": "SeaBIOS",
/usr/share/qemu/firmware/60-seabios-128k.json:    "description": "SeaBIOS",
/usr/share/qemu/firmware/50-ovmf-x86_64-secure-ms.json:    "description": "UEFI firmware for x86_64, with Secure Boot, SMM, and MS certs enrolled",
/usr/share/qemu/firmware/50-ovmf-x86_64-secure-opensuse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, SMM, and openSUSE certs enrolled",
/usr/share/qemu/firmware/50-ovmf-x86_64-secure-suse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, SMM, and SUSE certs enrolled",
/usr/share/qemu/firmware/50-ovmf-x86_64-secure.json:    "description": "UEFI firmware for x86_64, with Secure Boot and SMM",
/usr/share/qemu/firmware/50-xen-hvm-x86_64.json:    "description": "UEFI firmware for x86_64 XENFV",
/usr/share/qemu/firmware/60-ovmf-x86_64-2m-ms.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and MS certs enrolled (legacy 2MB flash size)",
/usr/share/qemu/firmware/60-ovmf-x86_64-2m-opensuse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and openSUSE certs enrolled (legacy 2MB flash size)",
/usr/share/qemu/firmware/60-ovmf-x86_64-2m-suse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and SUSE certs enrolled (legacy 2MB flash size)",
/usr/share/qemu/firmware/60-ovmf-x86_64-2m.json:    "description": "UEFI firmware for x86_64 (legacy 2MB flash size)",
/usr/share/qemu/firmware/60-ovmf-x86_64-ms.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and MS certs enrolled",
/usr/share/qemu/firmware/60-ovmf-x86_64-opensuse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and openSUSE certs enrolled",
/usr/share/qemu/firmware/60-ovmf-x86_64-suse.json:    "description": "UEFI firmware for x86_64, with Secure Boot, and SUSE certs enrolled",
/usr/share/qemu/firmware/60-ovmf-x86_64.json:    "description": "UEFI firmware for x86_64",
qy@yinspiron:~$
```

Among them, `60-ovmf-x86_64.json` describes the one without Secure Boot, namely
`/usr/share/qemu/ovmf-x86_64-4m-code.bin`:

```
qy@yinspiron:~$ cat /usr/share/qemu/firmware/60-ovmf-x86_64.json
{
    "description": "UEFI firmware for x86_64",
    "interface-types": [
        "uefi"
    ],
    "mapping": {
        "device": "flash",
        "executable": {
            "filename": "/usr/share/qemu/ovmf-x86_64-4m-code.bin",
            "format": "raw"
        },
        "nvram-template": {
            "filename": "/usr/share/qemu/ovmf-x86_64-4m-vars.bin",
            "format": "raw"
        }
    },
    "targets": [
        {
            "architecture": "x86_64",
            "machines": [
                "pc-i440fx-*",
                "pc-q35-*"
            ]
        }
    ],
    "features": [
        "acpi-s3",
        "acpi-s4",
        "amd-sev",
        "amd-sev-es",
        "verbose-dynamic"
    ],
    "tags": [

    ]
}
qy@yinspiron:~$
```

But how do I make the VM configuration file use a specific firmware? I got the
answer by creating a new VM in Virtual Machine Manager, with **this firmware**
explicitly specified:

![VM with firmware explicitly specified](/assets/images/2024-04-01/vm_creation_20240401.jpg)

This new VM has Secure Boot disabled. Comparing its configuration XML file with
the one of a VM that has Secure Boot, I found the related item:

```
  <os firmware='efi'>
    <type arch='x86_64' machine='pc-q35-7.1'>hvm</type>
  </os>
```

After applying this change, my existing VM also got Secure Boot disabled!

# Reference

- [UEFI Support in Libvirt](https://that.guru/blog/uefi-secure-boot-in-libvirt/):
  it mentions the firmware auto-selection feature, the hint I needed.
- [Domain XML format](https://libvirt.org/formatdomain.html#bios-bootloader):
  it has detailed information about the VM XML file.
