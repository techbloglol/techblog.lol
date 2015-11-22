+++
author = "vmrob"
date = "2015-11-07T13:16:31-07:00"
description = "So you've heard of next gen filesystems like ZFS and BTRFS and can't help but to get a little jealous that your simple-minded, consumer filesystem doesn't have newfangled features like copy-on-write, snapshots, de-duplication, and block-level compression. Using these features, I was able to halve the storage of my virtual machines saving me over 125 GB. In this article, I'll show you how to master ZFS on OS X using nothing but your existing Apple hardware."
tags = ['zfs', 'osx', 'workflow', 'power user']
title = "Getting started with ZFS on OS X"

+++

So you've heard of "next gen" filesystems like [ZFS](https://en.wikipedia.org/wiki/ZFS) and [BTRFS](https://en.wikipedia.org/wiki/Btrfs) and can't help but to get a little jealous that your simple-minded, consumer filesystem doesn't have newfangled features like [copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write), snapshots, de-duplication, and block-level compression. Using these features, I was able to halve the storage of my virtual machines saving me over 125 GB. In this article, I'll show you how to master ZFS on OS X using nothing but your existing Apple hardware.

# Disclaimer

In my personal experience, ZFS on OS X is still somewhat buggy. I've never lost any data, but the occasional virtual filesystem (vfs) deadlock isn't too uncommon and when that happens, the only thing you can really do is shut down the computer. When the vfs locks up, everything that uses it, including just about all of your user-level programs, will block on vfs calls indefinitely. Your computer will appear to function but then the beach ball of death will silently take over everything.

<center title="Copyright Allie Brosh -- http://hyperboleandahalf.blogspot.com/2010/05/sneaky-hate-spiral.html">![Copyright Allie Brosh -- http://hyperboleandahalf.blogspot.com/2010/05/sneaky-hate-spiral.html](images/you-must-wait.png)</center>

The maintainers over at the [OpenZFS on OS X project](https://github.com/openzfsonosx/zfs) are quite helpful and would love your stack traces and spindumps if you ever do encounter a problem. With any luck, my continued communication with them will resolve these specific issues.

That said, my desire to stuff as much data as I can into my ssd outweighs the desire for a 100% stable system. As I mentioned before, I saved over 125 GB with compression alone, and that has saved me hours of time trying to perform triage on files for my ever-decreasing storage.

With all of that out of the way, I would still encourage the you to play with the system to see if it satisfies your use cases.

# Installation

The package that we will use here is OpenZFS on OS X (O3X for short). The installation is simple and quite painless. If you're running El Capitan, you'll need to [disable System Integrity Protection](http://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-os-x-10-11) as O3X installs a pair of kernel extensions.

The latest release for O3X can be found [here](https://openzfsonosx.org/wiki/Downloads).

Once you've installed the package, you should have access to a few new tools from the command line. An easy way to test if the installation was successful is to query your system for the status of your (currently nonexistent) zpools.

```bash
$ zpool list
no pools available
```

# Terminology

Before we jump into creating a zpool, why don't we cover some basic ZFS terminology so you actually know what a zpool _is_.

In ZFS terms, a zpool is much like a virtual disk. It is presented as a single logical volume to ZFS but can be composed of multiple disks including caching layers and with some disks reserved for redundancy or even hot spares. Zpools are managed by the `zpool` utility.

A ZFS dataset is much like what would normally be considered a partition on an older filesystem. Unlike a normal disk partition, you can nest datasets. Nested datasets inherit properties from their parents or can have properties set independently. A dataset is typically where your files will live and is the most user facing component of ZFS. Datasets are managed by the `zfs` utility.

A zvol, or ZFS volume, is essentially a dataset presented as a block storage device. On OS X, this means that zvols live in `/dev/disk` and `/dev/rdisk` and can be formatted, partitioned, mounted, and manipulated the same way you work with physical disks. The advantage is that with a zvol, you get the benefits of ZFS at the block level. Zvols are also managed by the `zfs` utility.

A snapshot is exactly what it sounds like: a preserved copy of a zfs dataset or zvol. Snapshots can be mounted (readonly) or cloned (read/write) and are created instantaneously due to ZFS's copy-on-write semantics. Not only can they be created instantaneously, but they also consume no initial space. Only when the underlying data starts to change does the snapshot grow in space proportional to the differences between the snapshot image vs the live image.

# Determine where your zpool will live

If you're like me, you're using FileVault for full disk encryption. While there are some clever [options for disk encryption](https://openzfsonosx.org/wiki/Encryption) with OpenZFS on OS X involving using CoreStorage's encryption under your zpool, I preferred a sparsebundle on top of my existing HFS+ partition. While not the most performant option, it allows me to back up my zpool using Time Machine without any trouble.

Following in my footsteps, you can create a new sparsebundle from Disk Utility or via the command line. It's important to remember that we're not using a sparsebundle for saving space, only for backup savings with Time Machine. Be sure to keep your sparsebundle maximum size something manageable. You can always increase the size later.

If you don't use Time Machine or plan on backing up your zpool using some of the other great methods available, you can just as easily create a raw .img using `mkfile`.

```bash
$ hdiutil create -type SPARSEBUNDLE -layout NONE -size 1G tank.sparsebundle
```

If you created a sparsebundle, you're going to need to mount it before using it with zfs. If you're using a raw image file, you can skip the mount step and go straight to zpool creation.

```bash
$ hdiutil attach -imagekey diskimage-class=CRawDiskImage -nomount tank.sparsebundle
```

This process will output a disk identifier usable with `zpool create` which brings us to the next step.

# Create your first zpool

Creating a zpool is actually fairly simple though does require administrative privilages. In this example, we'll create a pool called tank `backed` by `/dev/disk2`.

```bash
$ sudo zpool create tank /dev/disk2
```

>Please be sure to use the correct device identifier or file when you use this command. I'm not responsible for you wiping your external drive.

# Create your first ZFS dataset

Creating ZFS datasets is also pretty straightforward.

```bash
$ sudo zfs create tank/my-dataset
```

In fact, if you end up using ZFS a lot, you'll probably use the `zfs create` and `zfs destroy` commands pretty often. If you want to avoid using root for these things, you'll probably want to give permission of management over the tank dataset to your user.

```bash
$ sudo zfs allow vmrob create,mount,destroy tank
```

There are a multitude of other permissions you can allow. To see these, run `zfs allow` with no arguments.

# Create your first zvol

As zvols are essentially block storage devices managed by ZFS, they can be extremely useful for minimizing space used by such devices. In fact, you can even create sparse disk images with zero space reservation and allow that to expand in the future.

```bash
$ sudo zfs create -V 100T -s tank/huge-zvol
```

The `-V 100T` argument tells zfs to create a zvol with a logical size of 100 TB. The `-s` argument tells zfs to create a sparse volume. That's to say that no initial space will be reserved, but added as needed when blocks are used.

When creating a zvol, it is accessible from two locations: `/dev/disk*` and `/var/run/zfs/zvol/*`. The second location is my preferred way to access the disk, but note that it is just a symlink to the corresponding location on `/dev/disk*`.

Another important thing to note is that a zvol will increase its space reservation even when writing zeros to the volume. If you enable compression on the volume via `zfs set compression=lz4 tank/huge-zvol`, all of the data stored on the volume will be compressed. When compression is enabled, blocks of storage that are entirely zero-filled are removed.

```bash
$ sudo dd if=/dev/zero dd of=/var/run/zfs/zvol/rdsk/tank/huge-zvol
dd: /var/run/zfs/zvol/rdsk/tank/huge-zvol: Input/output error
2022269+0 records in
2022268+0 records out
1035401216 bytes transferred in 52.048038 secs (19893184 bytes/sec)
```

IO error right around the 1 GB mark--exactly what we expect since tank only has about 1 GB. Because I've completely filled the volume, it seems that I'm unable to set the compression property. If I destroy the volume and recreate it using lz4 compression, we shouldn't get these IO errors writing out zeroes.

```bash
$ sudo zfs destroy tank/huge-zvol
$ sudo zfs set compression=lz4 tank
$ zfs create -V 100T -s tank/huge-zvol
$ dd if=/dev/zero | pv | sudo dd of=/var/run/zfs/zvol/rdsk/tank/huge-zvol
1.26GiB 0:00:59 [20.9MiB/s] [      <=>                                     ]
```

See how happily `dd` will try to fill that volume with zeroes? Despite this, Almost no space has been consumed.

```bash
$ zfs list
NAME             USED  AVAIL  REFER  MOUNTPOINT
tank            1.89M   974M   952K  /Volumes/tank
tank/huge-zvol     8K   974M     8K  -
```

# Enable compression

I touched on how to set some of the various properties that can apply to datasets in the previous section, but note that you need to remember to set these at least once.

ZFS de-duplication is a tricky matter and, honestly, I wouldn't recommend it. The memory and cpu requirements for the dataset get to be pretty big and supposedly there's another stability issue to consider. I thought I would get some pretty big de-duplication ratios, but I've seen space savings of less than 5% on my biggest datasets.

Compression, on the other hand, is where it's at. ZFS allows for the excellent [lz4](https://cyan4973.github.io/lz4/) algorithm which is designed for speed. I would recommend keeping this enabled on all of your pools.

```bash
$ zfs set compression=lz4 tank
```

As long as you don't override the property in a child dataset, you'll inherit that across all of your datasets in the tank zpool.

Another property you might want to consider setting might be `mountpoint` depending on whether or not you want to change the default mount point. For a full list of properties, use `zfs get all tank`.

# Importing and Exporting zpools

Obviously you'll need to be able to import and export pools. Import is basically the means to mount a zpool while export is the equivalent of unmounting it.

```bash
$ sudo zpool import tank
```

```bash
$ sudo zpool export tank
Running process: '/usr/sbin/diskutil' 'unmount' '/Volumes/tank'
Unmount successful for /Volumes/tank
Attempting to eject volume 'tank/huge-zvol'
Running process: '/usr/sbin/diskutil' 'unmountDisk' '/dev/disk3'
Unmount of all volumes on disk3 was successful
```

# Conclusion

I hope this has been a simple introduction to ZFS on OS X. There are many other topics I could cover, but will be saving those for future posts. If you have any questions or tips for our readers, let us know in the comments!
