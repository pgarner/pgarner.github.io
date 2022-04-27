---
title: Partitions
layout: post
---

# The old ages

Back in the day, in order to install Linux, I would partition a hard drive with four partitions:

1. A dedicated, small, *boot* partition, because either [MBR](https://en.wikipedia.org/wiki/Master_boot_record) or [grub](https://en.wikipedia.org/wiki/GNU_GRUB) can't see very far into the disk
1. A dedicated *swap* partition, because the memory was small and a partition was taken to perform better than a swap file.
1. A main *root* partition
1. A distinct *home* partition for user directories, because sooner or later there would be a bad sector, and it was just easier if it didn't take out both the home directories and the OS.  Partitions and indeed encryption are robust to hardware errors; filesystems are not (although see below).

It was [BIOS](https://en.wikipedia.org/wiki/BIOS), so it only knew about MBR, and MBR only supported four (primary) partitions.  It all made sense.  All the partitions were formatted ext3 or ext4, except perhaps the boot partition that didn't need a journal.

# The middle ages

Since about 2010, a few things changed:

1. BIOS has become [UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface), in turn UEFI knows about [GPT](https://en.wikipedia.org/wiki/GUID_Partition_Table) and [FAT32](https://en.wikipedia.org/wiki/File_Allocation_Table#FAT32), it makes sense to use GPT and keep the old boot partition but formatted FAT32.  So now the firmware knows about the boot partition.
1. UEFI also means no need for grub.  There was something called [gummiboot](https://en.wikipedia.org/wiki/Systemd-boot) that could be loaded directly by UEFI; that is now `systemd-boot`.  It comes with the OS; use it.
1. Eventhough GPT allows lots of partitions, it's easier to simply have one more and use [LVM](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) to create swap, root and home.  In principle they can be dynamically resized.

So, there are still four partitions.  But that last point also enables encryption: LVM can sit on top of [LUKS](https://gitlab.com/cryptsetup/cryptsetup/blob/master/README.md).  So the second GPT partition is first encrypted as one big block, then LVM further partitions things once decrypted.

The boot partition had to be FAT32 (no problem); I tended to format root and home as [BTRFS](https://en.wikipedia.org/wiki/Btrfs), but simply as a basic filesystem.

# Modern times

In the incarnation above, we still have a swap partition and two main partitions.  However, in 2022 (but I am late!) there are two other important considerations:

1. A contiguous swap file is now [just as efficient](https://wiki.archlinux.org/title/Swap) as a dedicated partition.  As long as fragmentation can be avoided, it makes much more sense to allocate it as a file.  A laptop can even hibernate to that file.
1. BTRFS claims to be error resilient.  This in principle removes the main justification for separating the root and home partitions from way back.

The way BTRFS mimics the partitions that LVM would normally manage is by use of [subvolumes](https://btrfs.readthedocs.io/en/latest/Subvolumes.html).  A subvolume is not a partition; it's just a separate filesystem in the same "namespace".  Although I can't find an authoritative answer, it is reasonable to assume that a hardware error should only affect one subvolume.  As long as that assumption holds then there is no need for partitions other than boot and "main" (I tend to name it "data").

One more consideration is that my machines tend to contain two hard disks.  Typically this might be an SSD with boot, root and home, and a larger HDD with "common" things.  Common things can be a media library or backup partition.  This leads to the root partition containing two subvolumes acting as mountpoints: `@home` with the home directories, and `@data` with assorted common things.

# BTRFS

A key tool that has become part of the BTRFS infrastructure is [snapper](http://snapper.io/).  The BTRFS literature makes a big thing of subvolumes and the ability to make snapshots of them.  Especially the [OpenSuse infrastructure](https://documentation.suse.com/sles/12-SP3/html/SLES-all/cha-advdisk.html) is built around the root partition being snapshotted.  This in turn results in lots of other subvolumes that only exist to prevent them being snapshotted as part of the root subvolume.  If (read: like me) you don't want to snapshot the root partition, then this can be nonsensical.

Rather, for me, there are two main and one minor reasons to use subvolumes:

1. Top level subvolumes are analogous to what would have been partitions in one of the older schemes above.  They tend to be called `@`, `@home`, &c, and are mounted using `fstab`.
1. Lower level subvolumes, notably user directories, are not individually mounted, but exist so that snapshots can be made.  The `useradd` utility has an explicit option to add a user as an explicit subvolume.  This is sensible.
1. Within that hierarchy, it is possible to make subvolumes for things that either need not be snapshotted or have particular attributes.  Good examples are virtual box's `Virtual Machines` directory and the `@swap` top level volume.

Note that all this is considered stable in BTRFS.  By contrast [managment of multiple disks](https://arstechnica.com/gadgets/2021/09/examining-btrfs-linuxs-perpetually-half-finished-filesystem/) is not, along with other features such as compression.  The [Debian wiki](https://wiki.debian.org/Btrfs) is quite explicit about this.  The [Arch wiki](https://wiki.archlinux.org/title/Snapper) is, as ever, the voice of reason.