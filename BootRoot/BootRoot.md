# Playing with the first Linux distribution (Boot/Root)

> This article is a free translation of a blog post titled [Ne moc hladká instalace Linuxu](https://www.abclinuxu.cz/blog/squeaker/2023/3/ne-moc-hladka-instalace-linuxu) published on [abclinuxu.cz](https://www.abclinuxu.cz/)

I tried another Linux installation, and although I managed to get it working, the result was definitely not satisfactory. I have encountered compatibility problems, bugs that fundamentally degrade performance, misleading documentation and version confusion.

## Choosing a distribution
After careful consideration and exploration of possible alternatives, I decided on the Boot/Root distribution from H.J. Lu. This is the oldest, and therefore undoubtedly the best tested distribution.

## Download, attempt one
A straightforward Google search referred me to the download and information on [The Programmer's Corner](https://www.pcorner.com/list/LINUX/HU985-5R.ZIP/INFO/) website. It contains a detailed description and offers an archive with the relevant files. Unfortunately, as it turns out right away, the download is only available after logging in. After all, such an important and popular distribution must be available from a number of other sources, so I fire up the search engine, only to discover, to my disappointment and no small surprise, that this assumption is wrong. So I finally gave in, registered and downloaded the archive.

There are actually two archives on pccorner.com. One is the [HU985-5R.ZIP](https://www.pcorner.com/list/LINUX/HU985-5R.ZIP/INFO/) mentioned above, and the other is [HU985-3R.ZIP](https://www.pcorner.com/list/LINUX/HU985-3R.ZIP/INFO/). They contain kernel 0.98.5; the first one is supposed to be from December 1992, so it is a relatively late version. The `3R` version contains an image for a 3.5" floppy, while the older `5R` for a 5.25" floppy of 1.2 MB and is slightly more stripped down compared to the `3R`. They boot without problems.

But wait! After all, period literature like [Rebel Code](https://www.goodreads.com/book/show/289947.Rebel_Code) talks about this Boot/Root proto-distribution being made up of two 5.25" floppies. The first one is supposed to be bootable and contains only the kernel, and the second one is supposed to contain the actual root system. Something is wrong here. What I managed to download is apparently a heavily modified, non-original version. I feel legitimately cheated and keep looking.

## Download, attempt two
My next search led me to [www.oldlinux.org](http://www.oldlinux.org), where a certain enthusiast has collected historical Linux material. The Boot/Root images are then in the [http://www.oldlinux.org/Linux.old/images/](http://www.oldlinux.org/Linux.old/images/) directory. There are even downloadable [virtual machines for Bochs](http://www.oldlinux.org/Linux.old/bochs/), where `bootroot-0.11-040928.zip` is Boot/Root. This also seems like a somewhat suspicious version and I would like to try installing my own.

A look at the image folder immediately reveals that they are mostly images of 3.5" floppies, which doesn't look at all original, so I reached for a pair of [bootimage-0.97](http://www.oldlinux.org/Linux.old/images/Original/bootimage-0.97) and [rootimage-0.97](http://www.oldlinux.org/Linux.old/images/Original/rootimage-0.97). Certainly not the most trustworthy (read oldest) pair, but it will likely be less problematic, as the Boot/Root system was standard at the time. These are the images for the 5.25", so finally, hurray for the installation.

## Hardware compatibility
I didn't have a suitable physical machine on hand to match the hardware requirements, so I resorted to a virtual environment of my favourite [PCem](https://www.pcem-emulator.co.uk/) (version 1.7). However, with it I stumbled. I played around with various virtual hardware configurations for quite a while but never managed to successfully boot the first floppy. Most often, it stalls at the third dot of the kernel loading procedure. Of course, VirtualBox is a no-go too, so I reach for the proven Bochs program, hoping that once I go through the installation and get a hard disk image, it will hopefully run on the PCem.

I created a 40 MB image; if this capacity was enough for Linus, it should be enough for me. The boot disk really only contains the LILO boot loader and the kernel. Once the kernel is loaded, it is necessary to insert the root floppy into the drive and continue loading it. This takes the user to the base environment, which they can try as root, user, or perform a disk install.

## Installation
This version is from August 1992.
```
Jim Winstead Jr. - 4 August 1992

This root disk is in many ways a bug fix to the 0.96 version - no
significant changes.  Maybe I should charge $50 for it and call it a
major new release - it worked for Microsoft and Windows 3.1. :)
```

The installation to disk is crude but not tricky, and the installer, if you can even call it like that, will guide you through it easily. The first step is to manually partition the disk using the classic text fdisk. I'm not going to complicate my life; one partition is enough, no need for a swap thanks to the absurdly large RAM (16 MB). After partitioning the disk, a reboot is required. Not that the installer will do it by itself. Waiting half a minute and resetting the computer is a normal procedure.

Next comes the formatting of the partition, which is simplified by the fact that the only supported file system is MINIX. `mkfs` is entered with the selected partition and number of blocks. And, oops, a hitch. MINIX only supports a maximum of 65535 blocks or about 32 MB. So the selected disk and partition capacity is more than I can use.

Another hitch, formatting is incredibly slow. A message keeps popping up
```
HD timeout
HD-controller reset
```
I prefer to make the disk image smaller and re-partition, but that doesn't help. The system keeps grumbling during formatting, even though it is moving slowly along. Eventually, out of laziness, I dismiss possible alternatives, like booting a newer system and performing the format there, and just let Bochs chug along in the background.

In a couple of hours, it's done, and my 30.6 MB disk is finally formatted! The installer then proceeds to copy the files. It basically takes the files from the floppy and copies them to the selected partition (`/dev/hda1` in my case), or partitions. Then all you have to do is select the system name.
```
This is probably the most important question in this install script:

What would you like to name your system? [linux]:

The installation is complete and the installer prints:

Okay, maybe it wasn't that vital.  :)

Generating /etc/rc.local...

Generating /etc/fstab...

That should be just about it...  Now you should have a usable
filesystem on your new Linux partition(s).

To be able to boot with your new Linux partitions as root, you need to
follow the instructions for editing your boot image to boot with the
hard drive as root.  This will most likely require using a DOS-based
binary editor such as Norton's Disk Editor, or a similar program.  In
the future, however, you can use the setroot program under Linux,
which allows you to set the root device of a boot image.  (You cannot
use this now, because the root image is in the drive the boot image
goes - you can't have two disks in there at once!)


[ press return to continue ]
```
And damn, I'm beginning to realize that the rumors about manual editing of disk contents that were mentioned on the internet were not entirely fictitious. At first, I didn't understand what exactly the text is trying to tell me; I'm supposed to be editing the contents of a hard drive. But then it dawns on me...

Installing to a hard drive means creating a MINIX partition with the root system copied to it. But that just replaces the rootimage floppy. You still need the bootimage floppy to boot! Moreover, it has to be manually repaired to use the hard disk instead of the second floppy. I have come to this description:
```
4) You should now have a complete (but very basic) root filesystem on
    your harddrive.  To be able to boot from floppy with this as your
    root filesystem, you will have to edit the boot diskette.  This is
    done by modifying the word at offset 508 (decimal) with a program
    such as Norton's Disk Editor, or use pboot.exe (available where
    you got this file, the boot disk and the root disk, hopefully.)

    This word is in 386-order (that is, least-significant byte first),
    which means it should look like one of the following:

       LSB MSB - device
       --------------------------
	01 03 - /dev/hda1 LSB = Least-Significant Byte
	02 03 - /dev/hda2 MSB = Most-Significant Byte
	03 03 - /dev/hda3
	04 03 - /dev/hda4

	41 03 - /dev/hdb1
	42 03 - /dev/hdb2
	43 03 - /dev/hdb3
	44 03 - /dev/hdb4

    The numbers are in hex, and if you're editing the boot diskette by
    hand, these two bytes should initially be 00 00 00 (and are followed
    by two non-zero bytes).
```

Of course, editing the disk image is a piece of cake. In its day, this system made quite a bit of sense, as it allowed a very simple and reliable method of running Linux alongside another operating system, most often MS-DOS. However, it dashed my hopes of using a PCem for the installed system.

The installed system is a crumb. It also can't do much. It is designed to allow users to copy other programs from floppy disks that they have downloaded from the Internet somewhere on campus. For example, Emacs, GCC or kernel source files.

```
Filesystem 1024-blocks  Used Available Capacity Mounted on
/dev/fd0          1200  1166        34      97% /
/dev/hda1        31364  1427     29937       5% /mnt
```
Surprisingly, the modified floppy works and the system boots from it. But with a timeout and subsequent reset of the disk controller on virtually every access, as it did already when formatted, the boot takes several tens of seconds, and any command (loaded from disk) does the same. So the system is unusable as a result.

I've been looking into what this could be. Supposedly it's a problem occurring since Bochs version 2.2.6, but I tried a lower one, and it didn't help. It does help to set the disk as a slave, but that didn't help either. Another possibility is a minor kernel modification, but I can't compile it on that system.

## Conclusion
So I did install Linux, but I wouldn't call it a success. Maybe try a different disk geometry or some other similar monkey business. If you have any tips, please share them. Anyway, I have mixed feelings about this. On the one hand, I was pleasantly surprised at how fairly simple Boot/Root installation is, if "hardware" compatibility issues didn't come into play. I'd definitely like to try it out on real hardware sometime. Now You might want too.


