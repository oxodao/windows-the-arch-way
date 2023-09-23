# Installing Windows the (real) Arch Linux way

## Introduction

A few days ago I've seen [this really cool post](https://news.ycombinator.com/item?id=37593459) on HackerNews that was entitled "Install Windows the Arch Linux Way". While it was neat, I remained unsatisfied.


Yeah we're setting Windows from the CLI, but why use the Windows tools at all ? Let's install Windows the real Arch Linux way: From Archlinux


**Note**: Once everything is working as expected, I'm planning on creating an ansible playbook to install Windows from the archlinux iso.


**Note**: This guide will only works for **UEFI** systems, not MBR.

## Steps

Boot into the archiso usb stick

First lets do an initial setup:
```sh
$ loadkeys fr # Loading the AZERTY layout as I'm french
$ iwctl # Connecting to Wifi
[...]
$ pacman -Syy wimlib # Installing wimlib (Lets us install Windows)
```

You will need the Windows 10/11 ISO somewhere. Either copy it through the network (scp it as ssh is already installed on the archiso, you just need to setup a password for the root user) or add it on a second USB stick and mount it:

```
$ mkdir /mnt/{usb,iso,win,efi} # Creating some temp folder to do the setup
$ mount [WINDOWS ISO USB STICK PARTITION] /mnt/usb
$ mount -o loop /mnt/usb/Win11_French_x64.iso /mnt/iso # Mounting the Windows ISO
```

Now we can format the ssd:
```sh
$ gdisk /dev/nvme0n1
o # Creating a new GPT partition table
n # new partition 1, should be 100M with the ef00 type code (EFI)
n # new partition 2, should be the rest of the disk with the 0700 type code (Microsoft basic data)
w # Write the new structure to the drive
```

**Note**: As said by `joemelonyeah` on HN:
> Microsoft actually has a guide for manual partitioning, which this guide does not follow. [1] The Microsoft guide cleans the whole disk and ensures the 100MB EFI partition is before the 16MB MSR partition.
> [1] https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/configure-uefigpt-based-hard-drive-partitions?view=windows-11


This guide doesn't do it either. *It works* thus I did not bother, but if someone want to enhance this guide, feel free to create a PR with better explanations ;)


Formatting the partitions correctly:
``` sh
$ mkfs.vfat -F32 /dev/nvme0n1p1
$ mkfs.ntfs -Q /dev/nvme0n1p2
```

Actually installing windows:
```sh
$ wiminfo /mnt/iso/sources/install.wim | less # Find out which version of windows we want to install
$ wimapply /mnt/iso/sources/install.wim [IMAGE NUMBER] /dev/nvme0n1p2 # Image number, for me 6 = W11 Pro
```

Now Windows is installed but we still have a few steps such as setting up the bootloader. Let's mount the newly setup drive:

```sh
$ mount /dev/nvme0n1p1 /mnt/efi
$ mount /dev/nvme0n1p2 /mnt/win
```

We copy the bootloader from the installed Windows:
```sh
$ mkdir -p /mnt/efi/EFI/Microsoft/Boot
$ cp -r /mnt/win/Windows/Boot/EFI/* /mnt/efi/EFI/Microsoft/Boot
```

The bootloader is installed but not configured yet. We still need to create the BCD.

### @TODO: find the equivalent for the following commands:
```
> bootrec /scanos
> bootrec /fixboot
> bootrec /rebuildbcd
> bcdboot C:\Windws /l fr-fr /s G: /f ALL
```
