# Creating, Managing, and Mounting Filesystems

![*Easy as cake...*](images/Chapter-Header/Chapter-11/server_problem-2.png "Server Problem")

## Objectives

* Compare and contrast different Linux filesystems
* Understand how to create and attach virtual disks
* Understand how the `fdisk` command is used to list, modify, and create filesystem partitions
* Understand Linux tools relating to filesystems, disk utilization, and mounting
* Understand how Logical Volume Management, extents, and disk based partitions differ
* Understand compression and archiving tools and their use on the command line

## Outcomes

At the conclusion of this chapter you will be able to add additional virtual disks to any Linux version installed in VirtualBox.  You will be able to format devices and create filesystems on newly formatted devices.  You will understand how to partition a device and install filesystems.  This chapter will give you experience with the tools needed to perform these actions.  We will study the LVM -- Linux Volume Manager and its new concept for dealing with disks.  Finally we will learn the concept of mounting and unmounting of disks.

## Storage Types

In looking at the prices of disk based storage from the Fall of 2015, a single terabyte Western Digital Blue hard drive was selling for ~$50.  In the time since, the price of storage has only decreased. Storage is cheap and with that in mind, adding *storage* or *capacity* to a system is trivial.  In fact the types of storage available since 2015 have changed drastically.

### Mechanical Hard Drives HDDs

The original, cheapest, and densest storage type in dollars per megabit is still a mechanical hard drive.  Often referred to as an HDD[^135] or magnetic drive.

![*Hard Drive*](images/Chapter-11/hdd/256px-Laptop-hard-drive-exposed.jpg "Hard Drive Internals")

These are made up of spinning platters where bits are stored via magnetic charge.  These systems have down sides in that parts of the surface wear out over time as well as they have mechanical parts (servos) that can fail over time.  They require constant amounts of power and if you scale this over large data centers, these costs can quickly add up.  In addition as the size of storage density has increased, the need to develop new storage mediums has arisen.  What was once a single magnetic platter, became three parallel platters, then became five spinning platters, which then became glass platters, which then became vertical electron based storage (somehow they got electrons to stand up instead of lay flat...), then moving to the latest [helium filled 10 TB disks](https://www.anandtech.com/show/9955/seagate-unveils-10-tb-heliumfilled-hard-disk-drive "Seagate 10TB helium filled disks").

As disks became bigger, new patterns for reading and writing data were created: SATA 1, 2, and 3.x bus technology was introduced for transmitting data faster from the disk to the CPU.  On-disk cache memory of 16-64mb was added to the disk for caching frequently accessed data.  On top of that we have [NCQ](https://sata-io.org/developers/sata-ecosystem/native-command-queuing "Native Command Queueing"). Native Command Queueing was introduces which looks at disk requests on the drive and reorders them to reduce round trips that a disk needs to make.  All of this is happening at 5400-7200 rpms, revolutions per minute, at a fly height of 8 Pico meters!

### Solid State Drives SSDs

In early 2012, a new medium called a Solid State Drive, or SSD was released.  These drives were different than mechanical disks because they relied on Flash Memory (SLC or MLC) and had disk based controllers to address this data.  The immediate advantage was that electrons move at the speed of light so the access time of any single bit was identical, compared to a mechanical drive which had to rotate into position to read the correct bits.  This increased speeds dramatically and while the original SSDs storage was low and comparatively today the dollar per megabit ration is not as good as a HDD, the read based access time was orders of magnitude faster.  Version of SATA beyond 3.2 introduced a process called [SATAe](https://en.wikipedia.org/wiki/SATA_Express "SATA Express wikipage"), which focuses on using the PCI Express bus to increase performance, rather than changing the SATA standard.

  SATA Version              Throughput
---------------------  -----------------------
  SATA revision 3.4      features added
  SATA revision 3.3      features added
  SATA revision 3.2      16 Gbit/s 1.97 GB/s
  SATA revision 3.0      6 Gbit/s  600 MB/s
  SATA revision 2.0      3 Gbit/s  300 MB/s
  SATA revision 1.0      1.5 Gbit/s 150 MB/s

![*Standard SSD connector*](images/Chapter-11/ssd/256px-Super_Talent_2.5in_SATA_SSD_SAM64GM25S.jpg "Standard SSD connector")[^134]

The original SSD disks were put out by RAM manufacturers and had RAM controller write issues.  The good ones were built by Intel and Samsung.  The other manufactures have caught up and you can get drives in 256, 512, and 1 TB sizes.  Since there are no moving parts the battery or power usage is far reduced from an HDD.  SSDs do have potential issues with wear leveling and block failure, but in the chips controlling these devices the manufacturers have built in protection against failing flash memory chips to spread out the disk writes to prolong their lives. These SSD drives use the standard SATA data transfer protocols allowing them to be drop in replacements and allowing the initial bandwidth limitation that HDDs suffered to be overcome.

As a price point marketing creation you might see SSHDs which are called Hybrid Drives.  They contain 4 to 16 GB of flash disk and the rest of the drive is a mechanical 3 to 5 platter disk.  This is supposed to give you the advantage of caching frequent data on the flash data, but the idea never caught on as the price of SSD and HDD both have dropped drastically, making this solution not necessary.

### NVMe

The latest incarnation of SSD disk based technology is [NVMe](https://en.wikipedia.org/wiki/NVM_Express "Wikipedia Article for NVMe") which stands for Non-Volatile Memory Express. These are solid state drives, but instead of sending data over the SATA interface, they connect directly to the high-speed PCIe bus, just like your video card does.  The advantage is in using the PCIe bus[^136] you gain the ability to transmit in 1 to 4x parallel transfer lanes as opposed to a serial fashion which SATA was designed for.  NVMe comes in expansion card factor and in the smaller [M.2 form factor](https://en.wikipedia.org/wiki/M.2 "M.2 form factor")[^137]. The price is high as the technology is still relatively new, but the performance gain is worth the investment.

   PCIe Version     Per Lane Throughput        1x        4x          8x        16x
-----------------  ----------------------  ---------- ---------- ---------- -----------
     PCIe 1.0       2 Gbit/s (250 MB/s)     2 Gbit/s   8 Gbit/s   16 Gbit/s  32 Gbit/s
     PCIe 2.0       4 Gbit/s (500 MB/s)     4 Gbit/s   16 Gbit/s  32 Gbit/s  64 Gbit/s
     PCIe 3.0       7 Gbit/s                7 Gbit/s   28 Gbit/s  56 Gbit/s  112 Gbit/s
     PCIe 4.0       15 Gbit/s               15 Gbit/s  60 Gbit/s  120 Gbit/s 240 Gbit/s

![*mini-Sata and M.2*](images/Chapter-11/ssd/M.2_and_mSATA_SSDs_comparison.jpg "mini-Sata and M.2")

The future of disk is something of a hybrid between ram and solid state/flash memory.  Intel launched a trademarked platform called [Optane](https://www.howtogeek.com/317294/what-is-intel-optane-memory/ "Intel Optane").  The target is cloud based servers running OS Containers and Virtualized platforms.  The idea is to increase the speed of disk to the point that is is close to or equal in speed to RAM (Non-volitile memory), thereby eliminating potential memory bottlenecks.

### Virtual Hard Drives

When dealing with Virtual Machines, we can attach and detach storage very easily.  With large deployments of VMware, and Cloud based services, in-place disk can be reformatted and used to attach virtual disks to a virtual machine - with the added ability to manipulate these hard drives as if they were simple files.  This [interface was standardized](https://lwn.net/Articles/239238/ "virtuio announcement webpage") and opensourced as part of the `virtio` package.  

### Disk Management in VirtualBox

For this chapter we will assume that you are using VirtualBox 6.x, but these concepts apply to any virtual machine or Hypervisor.  This also assumes you have free space on your computer.  Since the point of this lab is to explore and not production usage, you may want to get an external USB hard drive and use that for this chapter so as not to fill up your hard drive.

With your Ubuntu or Fedora virtual machine powered down, let's add some new disks (virtually) to your Linux system.  The first thing to do is locate the *SETTINGS* button on the VirtualBox main menu.

![*VirtualBox settings panel*](images/Chapter-11/virtual-box/settings.png "Settings")

The next menu to come up will show the *SETTINGS* options and the name of the virtual machine you are working on.

![*VirtualBox settings menu*](images/Chapter-11/virtual-box/settings-menu.png "Settings Menu")

\newpage

Select the *STORAGE* option from the menu on the left--this is where you can attach, detach, and modify virtual disks in VirtualBox.  In most cases these will be hard drives, but there is the ability to attach ISO images to a virtual cd-rom device as well.  That option is in the top half where you see *Controller: IDE*. Under that you might see the term *EMPTY* or you might see a virtualbox-guest-additions.iso attached.

![*Storage menu*](images/Chapter-11/virtual-box/storage.png "Storage")

We will be working with attaching virtual hard drives so we are interested in the bottom portion of the menu which is identified by *Controller: SATA* which is your [Serial ATA](https://en.wikipedia.org/wiki/Serial_ATA "Serial ATA") hard drive bus connection.  As a refresher, Serial ATA is the name of the signaling protocol the operating system uses to retrieve data from a hard drive. Go ahead and highlight the SATA Controller entry.

In order to add a new hard drive to your virtual machine, click the blue HDD icon with a __+__ sign at the bottom of the menu.

![*Add Storage Icon*](images/Chapter-11/virtual-box/add-storage-icon.png "Storage Icon")

Upon completion of that step a new menu will pop out of the HDD icon and give you and option to *Add Hard Disk*.

![*Add Storage*](images/Chapter-11/virtual-box/add-storage.png "Add Storage")

\newpage

Once you have selected *Add Hard Disk* a familiar set of screens come up, these are the same screens you walked through in chapter 2, and the same set of screens you walk through when setting up a new virtual machine.  You are presented first with an option to create a new disk or attach an existing one.  Usually you want to create a new disk.

![*Create New Disk*](images/Chapter-11/virtual-box/create-new.png "Create New")

Once that is selected you will be presented with the virtual disk type screen.  Since we will be working with VirtualBox, the default setting of VDI (VirtualBox Disk Image) will be the best selection.  But if you know this VM will be moving to another platform--you may want to choose accordingly.

![*VDI step-through*](images/Chapter-11/virtual-box/vdi.png "VDI")

\newpage

The next page allows you to choose either a dynamic or static allocated hard drive.  Dynamic is usually the best when you are working on a laptop or other development system, as you will be creating and destroying virtual machine rapidly, and static allocations of multiple gigabytes can become an issue after some time due to your disk filling up.

![*Dynamic Filesystem*](images/Chapter-11/virtual-box/dynamic.png "Dynamic")

This next screen allows you to change the location of where the virtual hard disk will be stored, as well as adjust the size of this new partition.  
![*Hard Drive Size*](images/Chapter-11/virtual-box/size.png "Size")

The final step is to choose to attach the virtual disk you just created to the virtual machine.

![*Choose new Virtual Disk to attach*](images/Chapter-11/virtual-box/choose-new-disk.png "Image showing how to attach new virtual disk to VM")

## Disk Partitioning and Formatting

Adding a virtual disk is only the first step, there are three more steps before we can use this disk.  Though the first two steps have been largely merged into a single step over the last decade.

1) Partition the disk / create a filesystem
1) Mount the disk so that it can be used by your operating system.

According to the ```fdisk``` man page, ```fdisk``` is a dialog-driven program for the creation and manipulation of partition tables.  The term __partition__ in relation to a hard drive is an important concept.   You can think of a brand new hard drive as a large plot of land, multiple acres of land.  The land itself in that form is not very useful, just as a new hard drive added into your system is not very useful.  Just as that land needs to be partitioned up into different uses and functions, a hard drive needs to know where its partitions are. Each disk can have multiple partitions.  The `fdisk` method here is mentioned for historic reasons but is not used the installation of modern operating systems, but it is good to know where we came from.

Linux inherited a way to name each device and reference certain partitions attached to a system.  Windows simply uses the letter C, D, E, and so forth.  Linux and Unix use a device/partition nomenclature.  You can see this currently by typing the ```lsblk``` command, which will print out currently all the block devices, their device name and their partitions in a nice tree based format.

![*lsblk output from a virtual machine with 2 additional drives attached*](images/Chapter-11/fdisk/lsblk.png "lsblk")

Here you will note that the drives are references by the prefix __sdx__ with the __x__ being the alphabet letter in incremental order.  Meaning that the first disk drive that your system detects in labeled __sda__, the next one would be __sdb__, and can you guess what the third and fourth system would be?  In the image above you notice that __sda__ has 3 partitions, sda1, sda2, and sda5.  These three partitions were created at installation time by the default Linux installer.  The first partition you can see has the character ```/``` in the far right column.  That is where the __root__ partition is mounted (meaning your entire filesystem). The second partition is where the ```/boot``` partition is mounted, and the final partition says __SWAP__ in the far right meaning this is a Linux SWAP partition--used by the operating system for moving data in and out of RAM in chunks at a time or called *pages*.

You can create partitions on a new disk for a fresh OS installation or just create a single partition to contain data.  The program mentioned above to create partitions is a program called ```fdisk```.  The ```fdisk``` command is considered an essential and standard Linux tool and is part of the [util-linux](https://en.wikipedia.org/wiki/Util-linux "Util Linux") package.  The best command to get started with when dealing with new disks and creating partitions is ```sudo fdisk -l```.  This command will list the current existing disks and any partitions they may have.  It will also report the undetermined state of any newly attached disks.  See the image below for a sample output.  If you are using Fedora 22/23 you will see a bit of a different output, you will see partitions labeled __LVM__ which will be explained at the end of the chapter.  
\newpage

![*sudo fdisk -l*](images/Chapter-11/fdisk/valid-partition.png "fdisk")

![*sudo fdisk -l*](images/Chapter-11/fdisk/not-valid-partition.png "fdisk")

The history of the Linux ```fdisk``` command goes way back.  Stemming from the early 1990's hard drives at that time using the standard BIOS of the day were only allowed 4 __primary partitions__ on the operating system.  At those times, hard drives were small, and devices were expensive, and things we take for granted now, like optical drives, didn't really exist, so 4 primary partitions was thought to be more than anyone would ever need.  A primary partition could be broken up into an __extended partition__. Then each __extended partition__ could be further sub-divided into as many __logical partitions__ that fit on the drive.  At that time only one __primary partition__ could be active (or bootable and seeable) at a time, all other primary partitions would be hidden from the currently active operating system.  In this world ```fdisk``` was built, hence its concern with partitioning.  There has been an improvement since 2000 called LVM, which is covered and thankfully used almost exclusively now by default.

To work/modify a device that has no existing partitions (say ```sdb``` in the image above). From the TLDP documentation regarding how to use ```fdisk```: [^ch11f122].  `fdisk` is started by typing (as root) fdisk device at the command prompt. Device might be something like /dev/hda or /dev/sda (see Section 2.1.1). The basic fdisk commands you need are:

* p print the partition table
* n create a new partition
* d delete a partition
* q quit without saving changes
* w write the new partition table and exit

To successfully create a partition on a new drive, let's select ```sdb``` in the example above.  The command ```sudo fdisk /dev/sdb``` will enter into ```fdisk``` and operate on this device.  Remember all *devices* are accessed through file handles in the ```/dev``` directory. Upon executing this command you are greeted with a status message reporting that the partition type cannot be detected or is not valid.  The error message seems a bit dated because you notice that it mentions DOS, SUN, SGI, and OSF--all outdated or unused partition types.  Similar to different languages or dialects, a partition also has to speak to an Operating system, and each operating system does it a bit different because of how particular filesystems are architected.  Fortunately this is a simple choice for us as we only need a Linux and a Linux SWAP partition for our uses--the rest are just artifacts of the past.

![*sudo fdisk /dev/sdb*](images/Chapter-11/fdisk/fdisk.png "fdisk")

  Always type __m__ for menu because the single letter commands are not intuitive.

![*m for menu*](images/Chapter-11/fdisk/m-for-menu.png "Menu")

  If you type the letter __l__ you will see the entire list of possible partitions, we are only interested in the value hex 82 and 83.  The next command to type is __p__ for printing out the current partition table--which will be blank.

\newpage

![*p for print*](images/Chapter-11/fdisk/p-for-print.png "Print")

The next step is to type the __n__ command to create a new partition.  You will be presented with two choices for your new partition.  In this case you can select __primary partition__.  In most cases in creating data drives you can select primary partitions without concern.  If you find yourself creating many data drives or creating triple and quad bootable systems (multiple operating systems)  then you will want to conserve those primary partitions and use __extended/logical__ partitioning.

![*n is for new partition*](images/Chapter-11/fdisk/n-for-new.png "New")

You are then presented with a series of options to choose a partition number, the beginning sector of your partition (usually best to choose the default) and the finishing sector (how big do you want your disk).  You can specify in a known quantity of K=kilobytes, M=megabytes, G=Gigabytes and prefacing that value with a __+__.  Selecting the default for the option of last sector will automatically fill up your disk with 1 single large partition.

![*New Partition Options*](images/Chapter-11/fdisk/n-options.png "Options")

Let's see if our partition was created successfully.  You can type __m__ to display the menu again or type the __p__ directly to print out the current partition table.  You will notice that it has been modified.

![*Successful Partition Creation*](images/Chapter-11/fdisk/p-finished.png "Finished")

\newpage

Everything looks good, but DON'T QUIT YET!  If you type __q__ now your changes will not be saved, and no partition information will be written.   Now you need to type __w__ to write the new partition data to the disk you are working on. The __w__ command will write and quit out automatically for you. After writing this partition data, you will see if show up in the ```sudo fdisk -l``` command.  After you see your new partition in ```fdisk``` of ```lsblk``` you are ready to move on to the next step of formatting a partition with a filesystem.

![*Write the Partition table data to disk*](images/Chapter-11/fdisk/w-for-write.png "Write")

### Drawbacks of disk partitions

Using the ```fdisk``` command does have its drawbacks.  The tool was designed in the day when systems had 1 or 2 hard drives.  Filesystems handled small files and the idea of large files partitions that spanned multiple disks was not possible due to software and processor limitations.  But we see with the cost of disk alone, systems having 24 or more hard drives is not unheard of.  Most filesystems were designed around the limitations of ```fdisk``` and since then new solutions have been designed to overcome the limitations of partitions such as LVM and filesystem extents, which we will cover at the end of this chapter.

## Logical Volume Manager

In order to enhance processing you may in your partitioning decisions want to place certain portions of the file-system on different disks.  For instance you may want to place the ```/var``` directory on a different disk so that system log writing doesn't slow down data stored in the users home directories.  You may be installing a MySQL database and want to move the default storage to a second disk you just mounted to reduce write ware on your hard disks.  These are good strategies to employ, but what happens as the hard disks in those examples begin to fill up?  How do you migrate or add larger disks?

The answer is that under standard partitioning and partitions you don't. You simply backup and reinstall the Operating System on a bigger drive.  This is very time consuming and a risky operation that is not taken lightly.  What to do?  A solution to this problem and the limitations of traditional disk partitions is called LVM, [Logical Volume Management](http://tldp.org/HOWTO/LVM-HOWTO/ "LVM"), created in 1998.  LVM version 2 is the current full featured version baked in to the [Linux kernel since version 2.6](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux) "LVM 2").

LVM is a different way to look at partitions and file-systems.  Instead of the standard way of partitioning up disks, instead we are dealing with multiple large disks.  As technology progressed, we took our single large disk that we had split into partitions with __fdisk__ and now we supplemented it with multiple disks in place of those partitions.  The Linux kernel needed a new way to manage those multiple disks, especially in regards to a single file system.  *"Logical volume management provides a higher-level view of the disk storage on a computer system than the traditional view of disks and partitions. This gives the system administrator much more flexibility in allocating storage to applications and users[^130]."*  In order to install LVM2 on Fedora/CentOS and Ubuntu you can type:

* ```sudo apt-get install lvm2```
* ```sudo dnf install lvm2```
* ```sudo yum install lvm2```

![*LVM diagram*](images/Chapter-11/LVM/LVM.png "LVM")

This diagram creates three concepts to know when dealing with LVM:

* Volume Group (VG) - The Volume Group is the highest level abstraction used within the LVM. It gathers together a collection of Logical Volumes and Physical Volumes into one administrative unit [^131].
* Physical Volume (PV) - A physical volume is typically a hard disk, though it may well just be a device that 'looks' like a hard disk (eg. a software raid device) [^132].
* Logical Volume (LV) -  The equivalent of a disk partition in a non-LVM system. The LV is visible as a standard block device; as such the LV can contain a file system (eg. /home) [^133].
  * Physical Extent (PE) - This is the unit of storage (blocks) that a PV is split into
  * Logical Extent (LE) - This matches the PE and is used when multiple PVs are added to an LG, to make the *logical disk*.  The LVM counts how many extents are possible and makes this its *disk* so to speak.

### Physical Volumes

The first thing to do in creating an LVM partition is to figure out what kind of disks you have and what kind of partition scheme you want to use.  Note that you can choose to use the entire disk ```/dev/sdb``` for instance or you can create a partition on the disk for use with LVM; if you do make sure to create the partition of type __0x8E__ LVM and not a standard Linux partition. In order to use entire disks you need to use the ```pvcreate``` command to *create* physical volumes, same case with the partition.

### Volume Groups

Once you have added the disks/partitions to the PV, now you need to create a Volume Group (VG) to add those PVs to.  The command to add PVs to an LG is: ```vgcreate VOLUME-GROUP-NAME /dev/sdx /dev/sdy```. You can extend this volume group by simply adding another ```pvcreate /dev/sdx1``` command for example and then using the ```vgextend VOLUME-GROUP-NAME /dev/sdz```.  There is also a ```vgreduce``` command that will remove a PV from a Volume Group.  The volume group allows for a single logical management unit for multiple disk/partitions.  This is useful as well for adding additional storage and removing storage devices that may have failed or are not performing at required parameters.

### Logical Volumes

From within our Volume Group (VG) we can now carve out smaller LV (Logical Volumes).  The nice part here is that the Logical Volumes don't have to match any partition or disk size--since they are logically based on the combined size of the Volume Group which has extents mapped across those disks.  Use the command ```lvcreate -n LOGICAL-VOLUME-NAME --size 250G VOLUME-GROUP-TO-ATTACH-TO```. The ```vgdisplay``` command will show what had been created and what is attached to where. There are options to make the LV striped extents as opposed to linear, but that is an application based decision.  Since LVs are logical they can also be extended and reduced on the fly--that alone is a better replacement for standard partitioning.  The command ```lvextend -L 50G /dev/VOLUME-GROUP-NAME/LOGICAL-VOLUME-NAME``` will extend the LV to become 50 GB in size.  Using ```-L+50G``` will add 50 additional gigabytes to an existing LV's size.

Once you have successfully created an LV, now it needs a filesystem installed.  Here you can add XFS, Ext4, Ext2, or any other file-system.   You would use the ```mkfs``` proper tool for your filesystem.  Once you have the filesystem created then you need a mount point.  Each filesystem type has tools that allow you to extend the file-system automatically without the need to reformat the entire system, if the underlying LV or traditional partition is modified.  Not all file-systems have the built in ability to shrink an existing partition.

### LVM Snapshots

One definite feature not included in traditional partitioning is the concept of **snapshots**.  **Snapshots** exist at the filesystem level in Btrfs and ZFS, but not XFS or ext4 as they are too old.  The command ```sudo lvcreate -s -n NAME-OF-SNAPSHOT -L 5g VOLUME-GROUP-NAME``` creates a LV volume that is a snapshot or CoW, Copy-on-Write partition.  It often can be smaller, because this new LV is only going to copy the changes, or deltas, from the original LV, not duplicating data but sharing it between the two LVs.   This delta can be merged back in, returning you to a point in time state, via the ```sudo lvconvert --merge``` command.  Also snapshot can be *promoted* to be a full LV that can be copied and mounted itself as a full LV.

#### Filesystem Snapshots

When dealing with LVM there is an ability to provide a snapshot, that is a point in time exact copy of a logical volume[^139].  Assuming your have your physical volumes, your volume groups, and logical volumes created, lets now create a snapshot of a logical volume (assume that we formatted the logical volume with ext4 and mounted it to ```/mnt/disk1```).

```bash
cd /mnt/disk1
# command to create a random file of 5MB
head -c 5MB /dev/urandom > datafile.txt
ls -lh
# Picking up from the tutorial at http://tldp.org/HOWTO/LVM-HOWTO/snapshots_backup.html
lvcreate -L592M -s -n disk-backup /dev/volgroupname/logicalvolname
# Type the random command to change the data since the original snapshot
head -c 25MB /dev/urandom >> datafile.txt
# You will now have a new device called /dev/volgroupname/disk-backup
# Mount this to /mnt/disk2
sudo mkdir -p /mnt/disk2
sudo mount -t ext4 /dev/volgroupname/disk-backup /mnt/disk2
cd /mnt/disk2
ls -l
ls -l /mnt/disk1
# What do you see?  Why?
```

You can remove the snapshots by unmounting the partition ```umount``` and the using the ```lvremove``` command.  How would you do this same process using ext4 without LVM?

## Filesystems

To extend our analogy of a disk drive being like land, and a partition being like different lots of land sold off to different people, then a filesystem would be the actual building that is built on the property to make use of the land, be it farm land, nature preserve, solar plant, or factory.  A __filesystem__ is the way that an operating system addresses, stores, and retrieves data stored on a disk.  It is an in-between layer so the operating system can have an addressing scheme for data, without having to know the exact mapping of the particular disk drive in question.

If you have used Windows before you are familiar with FAT32 and NTFS filesystems. Since Windows is created and curated by Microsoft, there has only been two different filesystems in the history of Windows.  Linux on the other-hand supports multiple different filesystems that serve many different purposes.

### ext/ext2

The MINIX filesystem was the first Linux based filesystem released in 1991.  It was borrowed conceptually from the Minix operating system that Andrew Tanenbaum had created. It had severe limitations since MINIX filesystem was engineered to be *ultra* backwards compatible--hence had 16 bit offsets and had a maximum partition size of 64 megabytes.  By 1992 and Linux 0.96c a new filesystem replacement called __ext__ was created and brought into Linux as the native filesystem.   By January of 1993, __ext2__ had been created and additional features added, including future proofing the system by adding unused options that could later on be tested and added as need arose.  Like most operating systems, data is broken up into __blocks__, which is the smallest sized piece of data that can be read or written by the operating system[^ch11f123].

: Limits of ext2

----------------------  ------- ------- ------- -------
Block size:              1 KiB   2 KiB   4 KiB   8 KiB
max. file size:          16 GiB 256 GiB  2 TiB   2 TiB
max. filesystem size:    4 TiB  8 TiB    16 TiB  32 TiB
----------------------  ------- ------- ------- -------

Traditionally your ```/boot``` partition is formatted as __ext2__ because it is only used for a short time to load your *initrd* and *kernel image* into memory, so the overhead of __ext4__ is not needed.  You can use the built in ```sudo mkfs``` command to format a partition with __ext2__.

### ext3/ext4

By 2001, as filesystems became larger, the amount of data being written increased and the chances for data corruption or disk writes to fail became more evident and critical.  The CPU could now handle to overhead of managing data writes to disk to ensure that those operations actually happened.

 *"A journaling file system is a file system that keeps track of changes not yet committed to the file system's main part by recording the intentions of such changes in a data structure known as a "journal", which is usually a circular log. In the event of a system crash or power failure, such file systems can be brought back online quicker with lower likelihood of becoming corrupted[^124][^125]."*

Not to be confused with journald from systemd, the __ext3__ filesystem, introduced to the Linux kernel the journaling feature in 2001. Being an extension basically of __ext2__, __ext3__ began to inherit legacy problems of __ext__ and __ext2__ as they were now over a decade old.

: Limits of ext3

Block size   Max file size   Max file system size
----------- --------------- ----------------------
  1 KiB         16 GiB             4 TiB
  2 KiB         256 GiB            8 TiB
  4 KiB         2 TiB             16 TiB
----------- --------------- ----------------------

By 2008 it became apparent that ext3 has reached the end of its development, and [Theodore Ts'o](https://en.wikipedia.org/wiki/Theodore_Ts%27o "Ts'o") announced that __ext4__ would extend the __ext__ filesystem a bit longer, but the growth of __ext__ had hit the end, and a newer filesystem needed to be developed to handle the larger sets of data and the massively improved hardware that existed since 1992, when ext was developed.  A major drawback of ext4 is the lack of snapshot support (like LVM) and the need to be backward compatible with ext2.

Ext4 saw the capacity extension of __ext3__ and introduction to __extents__. The __ext4__ filesystem can support volumes with sizes up to 1 exibyte (EiB) and files with sizes up to 16 tebibytes (TiB).

In ext4, __extents__ replaced the traditional block mapping scheme used by ext2 and ext3. An extent is a range of contiguous physical blocks, improving large file performance and reducing fragmentation. A single extent in __ext4__ can map up to 128 MiB of contiguous space with a 4 KiB block size [^126].  The advantage is that the smaller 4KB blocks are bound contiguously on disk together.

Theodore Ts'o is a respected developer in the open source community, who currently is the maintainer of __ext4__ and is employed by Google to develop filesystems.  __Ext4__ is the current default file system for most Linux distros, though that is changing as OpenSuse and Fedora 33 have adopted using Btrfs and Ubuntu is working on adopting OpenZFS as their default filesystem.  The advantage of ext4 is it is well tested and a well known quantity and is currently used by Google in Android devices as well.  To format a partition using the __ext4__ filesystem you would simply type `mkfs.ext4` and the partition will be formatted. You normally don't format entire devices, just partitions, which can take up entire disks. There are three additional competing filesystems since 2008 that fill the void left by __ext4__.

### Giga vs Gibi

These matter because humans and computers (and marketing) use different numbering systems.

: [Multiples of bytes](https://en.wikipedia.org/wiki/Gibibyte "Mulitples of bytes")

    Metric                IEC                    JEDEC
------------------- ----------------------- -----------------------
1000 kB kilobyte     1024 KiB kibibyte          KB kilobyte
1000 MB megabyte     1024 MiB mebibyte          MB megabyte
1000 GB gigabyte     1024 GiB gibibyte          GB gigabyte
1000 TB terabyte     1024 TiB tebibyte
1000 PB petabyte     1024 PiB pebibyte
1000 EB exabyte      1024 EiB exbibyte
1000 ZB zettabyte    1024 ZiB zebibyte
1000 YB yottabyte    1024 YiB yobibyte
------------------- ----------------------- -----------------------

### XFS

XFS is a robust and highly-scalable single host 64-bit journaling file system. It is entirely extent-based, so it supports very large files and file system sizes. The maximum supported file system size is 100 TB. The number of files an XFS system can hold is limited only by the space available in the file system[^127].

XFS was originally created by SGI (Silicon Graphics Inc) back in 1993 to be a high-end Unix work station filesystem.  SGI was the company that made computers in the 1990's for high end move special effects and graphical simulation.  They had their own version of Unix called IRIX, and needed a filesystem capable of handling large files at that time, and places like NASA which had large amounts of data to store and access.  SGI created XFS to suit that need.  XFS excels in the execution of parallel input/output (I/O) operations due to its design, which is based on allocation groups (a type of subdivision of the physical volumes in which XFS is used-also shortened to AGs). Because of this, XFS enables extreme scalability of I/O threads, file system bandwidth, and size of files and of the file system itself when spanning multiple physical storage devices[^127].

XFS was ported to Linux in 2001 as SGI and IRIX went out of business and the filesystem languished.  It was opensourced and GPL'd in 2002.  Red Hat began to see this filesystem as an alternative to ext4 and more mature than other replacements since it had over 10 years of development from the start to handle large sized files.  Red Hat also hired many of the SGI engineers and developers who created this filesystem and brought it back into production quality.  Red Hat began with RHEL 7 to deprecate ext4 as the default filesystem and implement XFS as their standard filesystem on the RHEL product.

XFS is notoriously bad at being used by an everyday computer because its strength is build on using a system storing large database files or archiving large files.  You can install the tools needed to make a partition of the XFS format by typing ```sudo apt-get install xfsprogs```; the XFS tools are already installed on Fedora and CentOS by default.  You can create an XFS filesystem using the ```sudo mkfs.xfs``` command.  We can grow an XFS filesystem with the command ```xfs_growfs /mount/point -D size```.

### Next Generation Linux Filesystems

The ext4 filesystem served its purpose well but by 2008 became apparent that ext4 was not the right filesystem design for taking full advantage of memory, disk, and processor improvements--as well as the changing use case of computing focusing on large dynamic clusters such as service companies like Google, Facebook, Twitter, and other social media companies. These new generation of filesystems combine  the filesystem, volume management, data compression, volume snapshots, and datafile integrity.  The two main candidates that are opensource and designed to take advantage of this current technological environment are [Btrfs](https://btrfs.wiki.kernel.org/index.php/Main_Page "btrfs wiki page") and [OpenZFS](https://openzfs.org/wiki/Main_Page "openZFS wiki page").

### Btrfs

This project was initially created by Chris Mason at Oracle in 2007, for use on their own storage products to compete against SUN Microsystems ZFS filesystem.  By 2013 it was considered stable and included in the Linux Kernel.  Facebook is currently where Chris Mason is employed and they are championing the use of this operating system on [their infrastructure](https://facebookmicrosites.github.io/btrfs/docs/btrfs-facebook.html "Facebook use of btrfs website").  

Btrfs is a modern copy-on-write (CoW) filesystem for Linux. Copy-on-write is at its core and optimization pattern that uses pointers instead of making multiple copies of data on a disk, therefore reducing write operations.  When the original file is modified then a true on disk copy of the file is made[^ch11f121]. Chris Mason said the goal of Btrfs is, *"to let Linux scale for the storage that will be available. Scaling is not just about addressing the storage but also means being able to administer and to manage it with a clean interface that lets people see what's being used and makes it more reliable[^128]."*

Btrfs adds support for resource pooling and using extents to make logical drives across physical devices removing the need for the use of LVM, volume management is now built in. Recently openSUSE and Fedora have adopted Btrfs as a filesystem, but support for Btrfs was remove in RHEL 8 (in favor of XFS and LVM).

In order to format a system using Btrfs you need to install ```btrfs-progs``` on Fedora 32+ and Ubuntu 20.04.  

       Install Btrfs tools
----------------------------------
`sudo dnf install btrfs-progs`
`sudo yum install btrfs-progs`
`sudo apt-get install btrfs-tools` - Ubuntu 18.04
`sudo apt-get install btrfs-progs` - Ubuntu 20.04

Table:  Demonstration of Btrfs syntax

### Btrfs Creation Commands

* Create a btrfs file system on a single device. For example:
  * `mkfs.btrfs /dev/sdb1`
* Create a btrfs file system with a label that you can use when mounting the file system. For example:
  * `mkfs.btrfs -L myvolume /dev/sdb2`
  * Note: The device must correspond to a partition if you intend to mount it by specifying the name of its label.
* Create a btrfs file system on a single device, but do not duplicate the metadata on that device. For example:
  * `mkfs.btrfs -m single /dev/sdc`
* Stripe the file system data and mirror the file system metadata across several devices. For example:
  * `mkfs.btrfs /dev/sdd /dev/sde`
* Stripe both the file system data and metadata across several devices. For example:
  * `mkfs.btrfs -m raid0 /dev/sdd /dev/sde`
* Mirror both the file system data and metadata across several devices. For example:
  * `mkfs.btrfs -d raid1 /dev/sdd /dev/sde`
* Stripe the file system data and metadata across several mirrored devices. You must specify an even number of devices, of which there must be at least four. For example:
  * `mkfs.btrfs -d raid10 -m raid10 /dev/sdf /dev/sdg /dev/sdh /dev/sdi /dev/sdj /dev/sdk`

Btrfs extensive documentation can be found at Oracle's website: [https://docs.oracle.com/cd/E37670_01/E37355/html/ol_create_btrfs.html](https://docs.oracle.com/cd/E37670_01/E37355/html/ol_create_btrfs.html "Oracle's Btrfs website")

### ZFS

ZFS is a filesystem originally developed by Sun for their Solaris Unix operating system and in use since 2001.  ZFS was opensourced by SUN in 2005 under the CDDL license (similar to the Mozilla Public License) but incompatible with the GPL. Oracle inherited ZFS when it invaded SUN in 2010. It is not licensed under the GPL but under a Sun/Oracle license called [CDDL](https://en.wikipedia.org/wiki/Common_Development_and_Distribution_License "CDDL"), which is similar to the GPL, but allowed Sun and Oracle to include proprietary parts of the operating system with opensource code. The CDDL is an opensource license, but not a copy-left license and thus GPL incompatible.  Recently Canonical, Ubuntu's parent company, disagreed with this, and began to offer OpenZFS natively as part of their operating system.  The argument of integrating CDDL based ZFS code into GPLv2 Linux Kernel was extensive with the FSF coming down in opposition of Ubuntu's interpretation of the GPL.

* [GPL Violations Related to Combining ZFS and Linux](https://sfconservancy.org/blog/2016/feb/25/zfs-and-linux/ "GPL Violations Related to Combining ZFS and Linux")
* [Interpreting, enforcing and changing the GNU GPL, as applied to combining Linux and ZFS](https://www.fsf.org/licensing/zfs-and-linux "Interpreting, enforcing and changing the GNU GPL, as applied to combining Linux and ZFS")
* [Ubuntu ZFS announcement](https://blog.ubuntu.com/2016/02/16/zfs-is-the-fs-for-containers-in-ubuntu-16-04 "Ubuntu ZFS Annoucement")

FreeBSD didn't have this restriction under the BSD license and they have had native kernel based support for ZFS since version 9 of FreeBSD and ZFS is a supported filesystem type on MacOS. As of Ubuntu 16.04, you can install ZFS via apt-get and include the CDDL licensed ZFS code on Linux as a loadable kernel module.  Ubuntu now supports the root partition being ZFS as well.  

Development of all ZFS code now lives in the upstream [OpenZFS project](https://openzfs.org/wiki/Main_Page "OpenZFS wikipage").  Since the ZFS code was opensourced, when Oracle tried to "closesource" the code base in 2010, essentially what Oracle did was make a fork of the project and keep their changes proprietary.  The rest of the community took ZFS and made the OpenZFS community, which now consolidates various ZFS code bases into a single repo for MacOS, FreeBSD, and Linux as a loadable kernel module.  RedHat has not participated in the Linux development of OpenZFS as most companies are afraid of potential litigation and lawsuits from Oracle (real or perceived). There is even a ZFS developer port who brought [ZFS to Windows](https://github.com/openzfsonwindows/ZFSin "ZFS on Windows"), using the latest Windows 10 OS.

ZFS is an elegantly designed filesystem: *"ZFS is a combined file system and logical volume manager designed by Sun Microsystems. The features of ZFS include protection against data corruption, support for high storage capacities, efficient data compression, integration of the concepts of filesystem and volume management, snapshots and copy-on-write clones, continuous integrity checking and automatic repair, Software based RAID, (RAID-Z)[^129]."*

#### ZFS Installation

Here is an example to install the ZFS module, load the module, and then format and create a zpool logical mirror (RAID1) in a few steps, tutorial comes from here: [https://wiki.ubuntu.com/Kernel/Reference/ZFS](https://wiki.ubuntu.com/Kernel/Reference/ZFS "ZFS Tutorial").

```bash
sudo apt install zfsutils-linux

# Now check to see if the zfs module is loaded
modprobe zfs
lsmod | grep zfs
# The name of the zfspool is: mydatapool
# The /dev/sdX and /dev/sdZ can be replaced by the actual device names
# found at the output of the command: lsblk
sudo zpool create mydatapool mirror /dev/sdX /dev/sdZ
lsblk
zfs list
df -h | grep mydatapool
```

Much like LVM, ZFS has native support for snapshots.  ZFS has a series of commands such as:

* ```zpool create | list | destroy | status```
  * this command creates a zpool
* ```zfs create | list | destroy```
  * used to create a ZFS filesystem on a zpool
* ```zfs snapshot volume@snap-name```
  * ```zfs snapshot mydatapool@snap1```
  * ```zfs list -t snapshot```
* ```zfs rollback```

ZFS also has a mechanism to send and receive snapshots, which done in a small enough increments which effectively creates a serialized synchronization feature.  This can be done on the same system as well as over a network connection to a remote computer. To synchronize a ZFS filesystem:

* First create a snapshot of a zpool
* Using the ```zfs send``` and ```zfs receive``` commands via a pipe you can send your snapshot to become another partition
  * ```zfs send datapool@today | zfs recv backuppool/backup```  
* You can pipe the command over ```ssh``` to restore to a remote system
  * ```zfs send datapool@today | ssh user@hostname sudo zfs recv backuppool/backup```

#### ZFS ZIL and SLOG

"*ZFS Intent Log, or ZIL- A logging mechanism where all of the data to be the written is stored, then later flushed as a transactional write. Similar in function to a journal for journaled filesystems, like ext3 or ext4. Typically stored on platter disk. Consists of a ZIL header, which points to a list of records, ZIL blocks and a ZIL trailer. The ZIL behaves differently for different writes. For writes smaller than 64KB (by default), the ZIL stores the write data. For writes larger, the write is not stored in the ZIL, and the ZIL maintains pointers to the synched data that is stored in the log record[^140]*".

"*Separate Intent Log, or SLOG- A separate logging device that caches the synchronous parts of the ZIL before flushing them to slower disk. This would either be a battery-backed DRAM drive or a fast SSD. The SLOG only caches synchronous data, and does not cache asynchronous data. Asynchronous data will flush directly to spinning disk. Further, blocks are written a block-at-a-time, rather than as simultaneous transactions to the SLOG. If the SLOG exists, the ZIL will be moved to it rather than residing on platter disk. Everything in the SLOG will always be in system memory[^140]*".

#### Additional ZFS Features

In addition there is an L2ARC cache for caching most recent and most frequently used data blocks.  This is a separate SSD based disk and can speed up data access[^141] [^142].  The ZIL and the L2ARC if not defined on separate disks will take a small portion of the each zpool created.  This is fine for low volume disk writes, but puts extra overhead on the system.  If you have fast SSDs that are small in size, say 30 to 80 GB, you can stripe them and place the L2ARC cache and or ZIL on these disks.  ZIL is a write only function and L2ARC cache becomes read predominant.   Using a zpool called **datapool** we are attaching two additional disk /dev/sde and /dev/sdf. You can add the directives for the log and cache after the zpool create command:  ```zpool add datapool cache /dev/sde2 /dev/sdf2 log mirror /dev/sde1 /dev/sdf1```.  You can use the /dev/ locations of disks, but disks can move around and be renamed.  It is often better to use the unique user ID or uuid for a disk, which doesn't change.  You can see your UUIDs with the ```blkid``` or `lblkd --fs` command[^141].

ZFS supports disk scrubbing.  Which will check every block of data against its own checksum meta-data and clean up any silent corruption. ZFS has a known good list of checksums of all blocks of data, and is constantly watching for corruption of data. Scrubs do not happen automatically but can be scheduled to run periodically.  You can check the status of a disk with the command ```zpool status datapool``` and execute a scrub command ```zpool scrub datapool```.

ZFS can enable transparent compression using GZIP or LZ4 with a simple set command: ```zfs set compression=lz4 datapool```.  This can help and there is little overhead.  Finally ZFS supports data-deduplication on a file basis.  If enabled each file is hashed with sha-256 and any files that match, only 1 of the files is kept, the others have markers pointing back to this original file.  This saves the overall amount of data you are storing and can reduce costs but the cost is high in amount of ram needed to store the de-dupe tables.

#### Finding a physical disk

![*HP HP EVA4400 storage array*](images/Chapter-11/disk/278px-HP_EVA4400-1.jpg "HP storage array")[^144]

ZFS, Btrfs, and LVM have the ability to remove disks from pools and volumes.   The trouble is you can remove the disk logically--but how do you identify which physical disk it is?  Luckily each disk has a serial number printed on the top of it.  When working in these scenarios you should have all of these serial numbers written down as well as the location of where that disk is.   You can find the serial number of the disk via the ```hdparm``` tool.  This script would enumerate through all of the disks you have on a system and print the values out.  Note the a, b, c, are a list of the device names.  In this case there is hard drive ```/dev/sda``` through ```/dev/sdg```[^143] run the command on your system and see what comes out.

```bash
  for i in a b c d e f g;
  do
    echo -n "/dev/sd$i: "
    hdparm -I /dev/sd$i | awk '/Serial Number/ {print $3}'
  done
```

```bash
#If you are missing those tools, just install following packages
sudo apt-get install hdparm
sudo apt-get install smartmontools
sudo apt-get install lshw

# These commands all will show you information relating to the disk serial number
# https://unix.stackexchange.com/questions/121757/harddisk-serial-number-from-terminal
sudo lshw -class disk
sudo smartctl -i /dev/sda
lsblk --nodeps -o name,serial
```

### HFS+, UFS, and APFS

The BSD systems has its own filesystem, UFS: [the Unix File System](http://www.ivoras.net/blog/tree/2013-10-24.why-ufs-in-freebsd-is-great.html "Unix Filesystem"). This filesystem was native to Unix going back to the System 7 Unix release.  Though UFS has been updated since and is officially UFS2, which was released around 1994 when BSD split from Unix due to the AT&T lawsuit.  UFS2 is considered similar to ext4 on Linux in capabilities at the current time.  FreeBSD and then other BSDs adopted ZFS to be able to extend the filesystem, capability of UFS.  All Illumos, or OpenSolaris based distros use ZFS natively as well, but [BSD based systems have switched](https://www.phoronix.com/scan.php?page=news_item&px=FreeBSD-ZFS-On-Linux "BSD rebases to ZFS on Linux") their ZFS to be based on the [ZFS on Linux](https://zfsonlinux.org/ "ZFS on Linux") code base due to a larger developer and feature base.

Apple had been using their own filesystem called HFS+ which was introduced in 1998 in MacOS 8.1 in 1998.  Features were added over time and in each release to keep the filesystem with feature parity for ext4.  This was used as the standard file system on all Mac and iOS devices until 2016/2017 when a new Apple designed filesystem was released across devices. Why a new filesystem?  Think about it, by 2016 what was the primary device that Apple was selling compared to 1998?  What type of storage media had HFS+ been designed for?   What type of storage media was running on the new Apple systems, in laptops and mobile?

Though Apple Was one of the first companies to [port ZFS to MacOS](http://dtrace.org/blogs/ahl/2016/06/15/apple_and_zfs/ "Apple ports ZFS to MacOS"), which is BSD based, eventually they decided to deploy a new filesystem called APFS (apple filesystem) which has a mix of ZFS like features that were home grown by Apple and more importantly not under the control of another company or any particular free or opensource licenses.  The goal for Apple was to use this filesystem across their platform, from MacOS to iOS devices with a focus on Flash bashed (SSD and NVMe) devices that HFS+ wasn't designed for.  The system has a similar feature set to ZFs and Btrfs such as encryption and snapshots, but [is missing data integrity](http://dtrace.org/blogs/ahl/2016/06/19/apfs-part1/ "Review of APFS").

### F2FS and EROFS

Due to the increased use of NAND or Flash based memory storage such as SSD, eMMC, and SD cards, a consortium of companies dealing with mobile devices came together in 2012 to create a filesystem which takes the nature of Flash into direct account in order to overcome some of the weaknesses of Flash based storage. F2FS stands for Flash-Friendly File System, and was developed by Samsung Electronics, Motorola Mobility, Huawei and Google.  The F2FS systems was adopted for some devices in 2012-2016 and saw a resurgence in use by 2018 led by the Pixel 3 and Motorola devices.

Hauwei replaced F2FS on their Honor devices with the [Enhanced Read-Only File System](https://en.wikipedia.org/wiki/EROFS "EROFS wiki page"), or EROFS.  It is designed and maintained by Huawei and is used on their own Android based devices introduced in 2018 and is part of the Linux Kernel.

### DragonFly BSD and Hammer FS

DragonFly BSD developer Matthew Dillion has been spearheading the development of his own distributed cluster based filesystem called [Hammer](https://www.dragonflybsd.org/hammer/ "Hammer Filesystem").  His goal is to have finer grained snapshoting--even per file on a constant basis and make snapshots almost a constant occurrence across the filesystem for easy migration of a filesystem to a different Hammer Cluster (migration over a network).   Work has recently finished on the Hammer 2 file system which is now an option for installation on DragonFly BSD 5.2+.  The Clustering feature is still a work in progress though, but Hammer has similar ZFS and Btrfs style features of file system and volume-management.  HammerFS is DragonFly BSD code, though it could be ported to other BSDs, the DragonFly internals have diverged so far that this becomes essentially impossible.

## Mounting and Unmounting of disks

Once a disk is partitioned, and formatted with a filesystem, it now needs to be mounted.   The concept of mounting came from the UNIX days of carrying a large reel of magnetic tape, and physically mounting it on a tape reader.  You can see all the mount points currently attached to your system by typing ```/etc/mtab```.  A filesystem needs to be mounted to a directory location.  Technically your root filesystem is mounted to the ```/``` partition.

In the previous examples we we have created partitions and filesystem, now let us mount them.  The first step we need to do is provide a mount point.  Traditionally that is done in the ```/mnt``` directory.  You should create your __mountpoints__ here.   Let's type ```sudo mkdir -p /mnt/data-drive```.  The name *data-drive* is an arbitrary name I have given my newly created __mountpoint__.  The ```-p``` flag will auto-create any subdirectory under ```/mnt``` that doesn't already exist.  Why did I type ```sudo```?  Who owns the ```/mnt``` directory?

Once this directory is created, you can use the ```mount``` command like this: ```sudo mount -t ext4 /dev/sdb /mnt/data-drive```.  The ```-t``` flag tells this mount that the filesystem is of type __ext4__ and the operating system needs to know so that it can interface correctly with the filesystem.  Once this is done, the directory will still be owned by root, you probably need to change the ownership of the directory so that you own and can write to it. How would you do that based on last chapter?  You could type ```sudo chown controller:controller /mnt/data-drive```, assuming your username is *controller*.

The partition can be unmounted by typing the ```umount``` command--yes it is missing the __n__.  Be careful you don't try to unmount the device while your pwd is in a directory on that mount--otherwise you will get a *device is busy error.*

### /etc/fstab

The ```/etc/fstab``` file controls the automatic mounting of your filesystems at boot.  Every time your system boots, technically each partition is remounted every time too.  If you create your own filesystem and want it mounted automatically on boot, then you would need to add an entry here. The ```/etc/fstab``` file has 6 columns containing values listed here: ```<device> <mount point> <fs type> <options> <dump> <pass>```.

An example entry could contain these values: ```/dev/sdb1 /mnt/data-drive  ext4  defaults  0   0```.  Devices now are typically listed by their UUID, which can be found by typing ```ls -l /dev/disk/by-uuid``` or `lsblk --fs /dev/sda`.  That is the actual command not a place holder. This is where the long strings you see in the ```/etc/fstab``` file in place of the device name.  

The use of UUIDs is a better idea as once a UUID is given, even if the device name changes the UUID will not.  Here is a sample `/etc/fstab` file showing UUID based fstab with an additional disk being mounted.

```bash
# <file system> <mount point> <type> <options> <dump> <pass>
# / was on /dev/sda1 during installation
UUID=e720a798-b02c-4e3f-8132-8f67f2be0c2c /           ext4    errors=remount-ro 0 1
/swapfile                                 none        swap    sw                0 0
UUID=003e6f67-9d31-4198-b3dd-4447f2337445 /mnt/disk2  btrfs   defaults          0 0
```

There are many options that can be set in the place of ```defaults``` or appended via a "," such as:

1. sync/async - All I/O to the file system should be done (a)synchronously.
2. auto - The filesystem can be mounted automatically (at bootup, or when mount is passed the -a option). This is really unnecessary as this is the default action of mount -a anyway.
3. noauto - The filesystem will NOT be automatically mounted at startup, or when mount passed -a. You must explicitly mount the filesystem.
4. dev/nodev - Interpret/Do not interpret character or block special devices on the file system.
5. exec / noexec - Permit/Prevent the execution of binaries from the filesystem.
6. suid/nosuid - Permit/Block the operation of suid, and sgid bits.
7. ro - Mount read-only.
8. rw - Mount read-write.
9. user - Permit any user to mount the filesystem. This automatically implies noexec, nosuid,nodev unless overridden.
10. nouser - Only permit root to mount the filesystem. This is also a default setting.
11. defaults - Use default settings. Equivalent to rw, suid, dev, exec, auto, nouser, async.
12. netdev - this is a network device, mount it after bringing up the network. Only valid with fstype nfs.

### systemd Mounting Units

Usually systemd will strive to absorb functions, but according to the man page for systemd.mount, *"Mounts listed in /etc/fstab will be converted into native units dynamically at boot and when the configuration of the system manager is reloaded. In general, configuring mount points through /etc/fstab is the preferred approach. See systemd-fstab-generator(8) for details about the conversion.*"   ZFS will take care of automounting its own partitions.  Btrfs you will need to add an entry using the UUID instead of the dev name which you can find via the ```blkid``` command.

Systemd has absorbed the purpose of the `/etc/fstab` file by creating **.mount** files.  Systemd processes `.mount` files first then processes the `/etc/fstab` file.  Let's look at an example where we have already created a Btrfs filesystem on a new virtual disk.  By using the `lbslk --fs` command we can retrieve the UUID of the disk.  The reason we would want look at using the UUID instead of the `/dev/sdx` is that once assigned the UUID won't change.  The device designation can change.  Say for instance that a hard drive fails, on reboot your system will re-enumerate the devices and now hard drive names will be changed.

In this example the output of the command, `lsblk --fs` will look like this (assuming you have attached the virtual disk and formatted it as Btrfs):

```bash
sda
sda1   ext4           e720a798-b02c-4e3f-8132-8f67f2be0c2c /
sdb    btrfs          003e6f67-9d31-4198-b3dd-4447f2337445
sdc    btrfs          62bab009-e64e-445c-bf33-5f2143569a83
```

```bash
# Create this file: /etc/systemd/system/mnt-databasedisk.mount
# Note that the .mount file name needs to be the same as the "Where" location
# Replace the / with -
# Make sure to enable the .mount at boot
# sudo systemctl enable mnt-databasedisk.mount
[Unit]
Description=Disk that was added to host the storage of our database

[Mount]
What=/dev/disk/by-uuid/003e6f67-9d31-4198-b3dd-4447f2337445
Where=/mnt/databasedisk
Type=btrfs
Options=defaults

[Install]
WantedBy=multi-user.target
```

**Remember!** ZFS handles explicit mounting of volumes/zpools as part of the act of creating zpools and do not need to have `.mount` files created.

### Disk related tools

There are two useful commands to use in regards to understanding the disk resource use in regards to the filesystem.  The ```df``` command will list the disk usage.   There is an optional ```-H``` and ```-h``` which presents the file-system usage in Gigabytes (-H is metric: giga, -h is binary, gibi).  When you use ```df``` without any directories, it will list all file-systems.  The command below lists the file-system that contains the user's home directory: ```/home/controller``` for example.

![*df -H /home/controller*](images/Chapter-11/du/df-h.png "df")

The ```du``` command is disk usage.  This is a helpful command to show the exact *byte-count* that each file is actually using.  When using `ls -l` Linux reports only 4096 kb for a directories size, this does not actually reflect the size of the content inside the directory.  The ```du``` command will do that for you.

These commands might not report completely accurate information when dealing with next generation filesystems like ZFS and Btrfs. use the commands: `sudo btrfs filesystem usage [mountpoint]` or `sudo zfs list [poolname]`

```bash
sudo btrfs filesystem usage /mnt/databasedisk
Overall:
    Device size:         3.00GiB
    Device allocated:    331.12MiB
    Device unallocated:  2.68GiB
    Device missing:      0.00B
    Used:                320.00KiB
    Free (estimated):    2.68GiB (min: 1.35GiB)
    Data ratio:          1.00
    Metadata ratio:      2.00
    Global reserve:      3.25MiB (used: 0.00B)

Data,single: Size:8.00MiB, Used:64.00KiB
   /dev/sdc    8.00MiB

Metadata,DUP: Size:153.56MiB, Used:112.00KiB
   /dev/sdc  307.12MiB

System,DUP: Size:8.00MiB, Used:16.00KiB
   /dev/sdc   16.00MiB

Unallocated:
   /dev/sdc    2.68GiB
```

> *When you compare the space consumption that is reported by the df command with the zfs list command, consider that df is reporting the pool size and not just file system sizes. In addition, df doesn't understand descendent file systems or whether snapshots exist. If any ZFS properties, such as compression and quotas, are set on file systems, reconciling the space consumption that is reported by df might be difficult[^138].*

```bash
# df -H /data1
Filesystem      Size  Used Avail Use% Mounted on
data1           1.9G  132k  1.9G   1% /data1

# sudo zfs list data1
NAME    USED  AVAIL     REFER  MOUNTPOINT
data1   104K  1.75G       24K  /data1
```

## Compression and Archiving tools

If you remember the history of Unix and history of technology you remember that hard drive space was at a premium for many many years.  Also the concept of discrete hard drives did not come about until the early to mid 1980s.  Things we take for granted now--such as zipping a series of folders and attaching them as a single file in email were unthinkable in 1979.

### tar

By 1979 local storage had increased to the point where it was conceivable that a **t**ape **ar**chive or tar file could be taken of a directory structure for backup purposes preserve the directory structure. The ```tar``` command was created and first included in Unix System 7 release.  The added advantage was that the tar file could be transferred as a single file, thereby reducing network overhead and os seek-time but retain a hierarchy of directories.  This method also became the preferred way to distribute code that was used to compile applications.  One could just un-tar an archive and then compile the code knowing that the directory structure of the included files was correctly preserved.  

The tar command only does archiving and does not do any compression--only preserving of file structure in the same way that an ISO file preserves structure.  The tar archive by convention is assigned a file extension of __.tar__ but this is not added automatically.

> __Example Usage:__ ```tar -cvf code.tar ./code-directory``` This command will create a ```tar``` archive of the directory called code-directory. tar \[options\] \[archive name and location\] \[what to archive\]

> __Example Usage:__ ```tar -xvf code.tar``` This command will unpack or extract a ```tar``` archive of the directory called code-directory and place it in the ```pwd```.

### compress

As file sizes grew the need to compress redundant data became apparent.  In dealing with compression you have two sides and you have to choose one.  Either the fast time to compress and larger file sizes, or slower time to compress and smaller file sizes. The initial compression algorithms went for the faster compression but larger file option.  The first compression tool on Unix was called ```compress```.  But it was encumbered by a patent on the [LZW](https://en.wikipedia.org/wiki/Lempel%E2%80%93Ziv%E2%80%93Welch "LZW") compression algorithm, the same patent on that lead to the creation of the jpeg image standard to replace the encumbered GIF image format.  Because of this compress was never *free* and outside of its invention and inclusion in commercial Unix in 1985, use could never catch on.

> Compress/Uncompress is a Unix shell compression program based on the LZW compression algorithm. Compared to more modern compression utilities such as gzip and bzip2, compress performs faster and with less memory usage, at the cost of a significantly lower compression ratio [^72].

### gzip

By 1991, Phil Katz had created an opensource implementation of LZW called [DEFLATE](https://en.wikipedia.org/wiki/DEFLATE "DEFLATE").  This was the basis of the popular PKZip program and the origin of the .zip compression extension.  The [GNU project](https://en.wikipedia.org/wiki/Gzip "GNU") began its own GPL based implementation using the DEFLATE algorithm and completed it by October of 1992.  It was named gzip [(GNU zip)](https://www.gnu.org/software/gzip/ "GNU zip").

> gzip is based on the DEFLATE algorithm, which is a combination of LZ77 and Huffman coding. DEFLATE was intended as a replacement for LZW and other patent-encumbered data compression algorithms which, at the time, limited the usability of compress and other popular archivers [^73].

### bzip2

This is a similar algorithm to gzip in that they do compression only.  It differs from gzip in that it uses the [LZMA](https://en.wikipedia.org/wiki/Lempel%E2%80%93Ziv%E2%80%93Markov_chain_algorithm "LZMA") compression algorithm which produces smaller sized files but take more CPU time and memory to compress.  This is deemed an acceptable trade off as computers are only getting faster.  Decompression is very fast compared to compression.  It was released in 2001[^74] and is available under a BSD-like license.  

### xz

The [xz](http://tukaani.org/xz/format.html "xz") compression tool is using the [LZMA2](https://en.wikipedia.org/wiki/Lempel%E2%80%93Ziv%E2%80%93Markov_chain_algorithm "LZMA2") compression algorithm which is superior in decreasing size at the expense of compute time.  The xz algorithm uses LZMA2 which has support for multithreading in compressing and decompressing in parallel. Back in 1985 when the first Compress program was created processing power was slow compared to the speed of the networks.  Now the speed of the processor is much faster relative to the speed of our networks. On today's networks we are moving multiple gigabytes around at a time.  The xz compression tool is the tool for the future.  In December 2013, [kernel.org](http://kernel.org "kernel.org") announced the addition of xz compressed files and ending bzip2 compressed files for distributing the Linux kernel archive files. The xz tool works only on single files and cannot be used for archiving.  In this case you would compress an archive file (like a tar file) to maximize usage.

### zstd

[Zstandard](https://facebook.github.io/zstd/ "Zstandard compression website") is a real-time compression algorithm, providing high compression ratios created at Facebook in 2015. It offers a very wide range of compression / speed trade-off, while being backed by a very fast decoder (see benchmarks below). It also offers a special mode for small data, called dictionary compression, and can create dictionaries from any sample set. Zstandard library is provided as open source software using a BSD license.  It can be installed via the command: ```sudo apt-get install zstd```.

### tarballs

A tape archive that is additionally compressed by another tool is called a __tar ball__ and the compression method is usually appended to the end of the filename.  

>  __Example Usage:__ ```tar -cvzf code.tar.gz ./code-directory``` This command will create a ```tar``` archive of the directory called code-directory and will compress it using the gzip compression algorithm by default.  *Note the -z option added.   Add a lowercase -j for bzip2 and uppercase -J for xz. Make sure to change the file extensions.

> __Example usage:__ Each one of these tar archives has been further compressed by one of the 4 Unix/Linux compression methods ```file linux-4.3-rc3.tar.Z; file linux-4.3-rc3.tar.gzip; file linux-4.3-rc3.tar.bzip2; file linux-4.3-rc3.tar.xz```

> __Example usage:__  Previously you had to pass a flag to the ```tar``` command to tell it what type of compression algorithm to decompress with but now ```tar``` is smart and will autodetect for you.  The flags you need to simply pass are -x for extract, -v for verbose (optional), and -f for file (optional) ```wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.11.22.tar.xz; tar -xvJf linux-5.11.22.tar.xz```  

As of version > 1.31 of `tar` there is also support for zstd.  Which can be filter and archive via the ```--zstd`` flag.

## Chapter Conclusions and Review

In this chapter we learned and mastered the concepts of disk based hardware.  We learned about the types of Linux filesystems and the features of the new generation of filesystems.  We learned how to install and configure volumes, snapshots, and compression. You have been prepared to with the basics of how to manage and understand the file-system on your Linux distro.

### Review Questions

1) What is the `fdisk` program?

   a) a dialog-driven program for the creation and manipulation of partition tables.
   b) a filesystem creation tool
   c) a tool for formatting floppy disks
   d) a tool for removing disks from a system

2) What is the default VirtualBox disk type?

   a) VDI
   b) HDD
   c) COW
   d) VDD

3) After attaching a new virtual disk, what is the next step?

   a) partitioning
   b) make a filesystem
   c) mount a filesystem
   d) add an extent

4) Which command will print out currently all the block devices, their device name, and their partitions in a nice tree based format?

   a) lspci
   b) lsblk
   c) lsusb
   d) lstree

5) `fdisk` is part of what package?

   a) utils
   b) GNU
   c) utils-linux
   d) utils-unix

6) What would be the name of the second SATA disk attached to your system?

   a) sda
   b) sdb
   c) sdc
   d) sdd

7) What is the name of the first native Linux filesystem released in 1992?

   a) ext2
   b) minix
   c) ext4
   d) ext

8) What is the name of the current default Linux Filesystem?

   a) ext
   b) btrfs
   c) ext3
   d) ext4

9) Ext4 breaks up data into __________, which is the smallest sized piece of data that can be read or written?

   a) sectors
   b) tracks
   c) blocks
   d) clusters

10) If you use the ext2 filesystem and choose a 4 KiB block, what is the maximum filesystem size?

    a) 2 TiB
    b) 16 TiB
    c) 4 TiB
    d) 4 GiB

11) What is the name of the maintainer of the ext4 filesystem?

    a) Brian Kernighan
    b) Theodore Ts'o
    c) Andrew Tanenbaum
    d) Google

12) What is the name of the filesystem that the ext4 maintainer, Theodore Ts'o, is recommending to replace ext4?

    a) XFS
    b) Btrfs
    c) ZFS
    d) HAMMER

13) What is the name of the filesystem that Red Hat adopted on their RHEL 7 platform to replace ext4 and support better performance on large filesystems?

    a) ZFS
    b) XFS
    c) Btrfs
    d) HAMMER

14) Which is the correct command needed to install on Ubuntu to be able to create XFS filesystems?

    a) `sudo apt-get install xfsprogs`
    b) `sudo apt-get install zprogs`
    c) `sudo apt-get install file-progs`
    d) `sudo apt-get install zfsprogs`

15) What is the name of the combined filesystem and logical volume manager designed by Sun Microsystems?

    a) XFS
    b) SunFS
    c) ZFS
    d) Btrfs

16) Which is the correct command for making an ext4 filesystem on a partition /dev/sdb1?

    a) `sudo mkfs.ext4 /dev/sdb`
    b) `sudo mkfs.ext4 /dev/sdb1`
    c) `sudo mkfs /dev/sdb1`
    d) `sudo makefs`

17) Which is the correct command to mount an ext4 filesystem, /dev/sdb1 on a mount point /mnt/data-drive-2?

    a) `sudo mnt /dev/sdb1 /mnt/data-drive-2`
    b) `sudo mnt -t ext4 /dev/sdb1 /mnt/data-drive-2`
    c) `sudo mount -t ext4 /dev/sdb1 /mnt/data-drive-2`
    d) `sudo mount /dev/sdb1 /mnt/data-drive-2`

18) Which file contains the mountpoints that will be mounted automatically at boot?

    a) /etc/mtab
    b) /etc/default/grub.conf
    c) /etc/fstab
    d) /etc/tab

19) What is the command used to create a LVM physical volume?

    a) `pvcreate`
    b) `pvck`
    c) `pvadd`
    d) `pvscan`

20) What is the command used to create a LVM volume group?

    a) `vgcreate`
    b) `vgck`
    c) `vgdisplay`
    d) `vgmknodes`

### Podcast Questions

[DragonFly BSD](https://dragonflybsd.org "Dragon Fly BSD") - Listen to this podcast: [https://ia802605.us.archive.org/9/items/bsdtalk248/bsdtalk248.mp3](https://ia802605.us.archive.org/9/items/bsdtalk248/bsdtalk248.mp3 "DragonFly BSD")

* ~1:25 What did DragonFly BSD drop with the 4.0 release?
* ~1:40 What was the other major feature that DragonFly BSD added?
* ~3:40 What modification did they add to the Packet Filter?
* ~10:00 What is the largest system DragonFly BSD has access to?
* ~11:45 What is the difference between DragonFly BSD's network stack compared to BSD and Linux?
* ~13:25 What is the limitations of the Hammer 1 Filesystem?
* ~13:45 What features will Hammer 2 Filesystem add?
* ~15:45 What is the intended use case of Hammer 2 FS?
* ~18:00 What sub-system is still in the works needed to make DragonFly BSD a stable work station?
* ~25:00 What is package-ng?
* ~30:00 How does DragonFly BSD handle suspend and resume functions common to laptops?
* ~35:50 What is the growing issue about systemd in relation to BSD?
* ~38:00 Of the 20,000 packages available in DragonFly BSD where are they primarily targeted?
* ~38:30 Out of FreeBSD, OpenBSD, NetBSD, and DragonFly -- what is each project focusing on?
* ~40:23 How does GPL based Linux software cross over into BSD distros?

### Lab

#### Lab 11 Objectives

* Creating virtual disks in VirtualBox
* Creating new partitions in fdisk
* Creating new filesystems with mkfs
* Creating new filesystems in ZFS and Btrfs
* Mounting new filesystems
* Editing `/etc/fstab` and using systemd .mount files to make our mounts permanent

#### Lab 11 Outcomes

At the conclusion of this lab you will have successfully created a new virtual disk in VirtualBox, created new partitions using fdisk, formatted those partitions using mkfs, XFS, and ZFS, and mounted all those partitions manually and automatically using the `/etc/fstab`.

#### Lab 11 Activities

For each of the bullet points, take a screenshot of the output of the commands to display the content to demonstrate the concepts.  Note - make your screenshot efficient, and capture only relevant data along with numbering the output.  All disks that are created can be 2 GB unless noted.

1. Create 1 virtual drive in VirtualBox:

   a. Use fdisk to create a primary partition
   b. Format it with ext4
   c. Mount it to /mnt/disk1
   d. Add it to your fstab

2. Create 2 more virtual drives:

   a. Create a single volume group named vg-group
   b. Create 1 logical volume named lv-group using the two drives
   c. Format it with XFS
   d. Mount it to /mnt/disk2
   e. Add the lv-group to your fstab, reboot the system and `cat` the  `/etc/fstab` and show that your entry is present.

3. Using the same LVM as before:

   a. Add an additional VirtualBox disk and the create a LVM physical disk
   b. Grow the volume group and logical volume
   c. Grow the XFS file system

4. Using LVM of the previous exercise on the logical volume lv-group

   a. Using either `fallocate` or `truncate` commands, create a file 25 megabytes in size and name it **datadump.txt** on your Logical Volume
   b. Reference Section 11.5.4 LVM Snapshots: create an LVM snapshot of the logical volume named `lv-backup`
   c. Mount the snapshot to `/mnt/disk3` (create this location if not existing)
   d. `ls -l` the contents of `/mnt/disk3`

5. Using Ubuntu 20.04 and ZFS, attach four 1 GB disks and create RAID 10 (a mirrored stripe). Display the `zpool status` and take a screenshot of the output.

6. Using Ubuntu 20.04, attach 4 virtual disks of 1 GB each. Create two Btrfs mirrored drives named disk1 and disk2.  Take a screenshot of the output of the `btrfs filesystem show` command for each disk.

7. From the previous question, attach another 1 GB virtual disk. Create a Btrfs partition on this disk named disk3.  Create a snapshot of disk1 save it to the newly created disk.  Use the `btrfs subvolume list` command to generate output for a snapshot. Reference [https://docs.oracle.com/cd/E37670_01/E37355/html/ol_use_case3_btrfs.html](https://docs.oracle.com/cd/E37670_01/E37355/html/ol_use_case3_btrfs.html "btrfs documentation for subvolumes")

8. Using Fedora, attach 4 1 GB disks in a Btrfs stripe.  

    a. Take a screenshot of the `btrfs filesystem df` command for this volume.  
    b. Then remove one of the virtual disks from the stripe.  Take a screenshot of the `btrfs filesystem df` command for this volume.
    c. Attach an additional 2 gb disk to the Btrfs stripe. Take a screenshot of the `btrfs filesystem df` command for this volume.  
    d. Extend the Btrfs filesystem to encompass using all of the new disk space. Take a screenshot of the `btrfs filesystem df` command for this volume.

9. From the previous exercise using your ZFS pool named datapool, create a 25 megabyte file named datadump.txt:

   a. Attach a third virtual disk to the system and create a zpool named backup
   b. Execute the `ls -lh` command to display the file and its size
   c. Take a ZFS snapshot of the datapool named @today - take a screenshot of the command output: `zfs list -s snapshot`
   d. Using the ZFS send and recv commands copy the @today snapshot to the zpool named backup
   e. Execute `ls -lh` command on the zpool backup
   f. Using the commandline, append an additional 25 mb to `/datapool/datadump.txt`
   g. Execute an `ls -lh` on zpool datapool and backup to compare the two files
   h. Take a screenshot of the command output: `zfs list -s snapshot` to show the growth in the snapshot

10. Create a systemd .mount unit file for the Btrfs partitions created in question number 8

    a. List the content of the .mount files in a screenshot(s)
    b. Reboot the system and take a screenshot of the df -H to see the .mount files work

11. You will need two systems with bridged networking and ssh enabled, create a 25 megabyte file named databasedump.txt on the zpool datapool:

    a. On the first system (the system without zpool datapool), create a datapool name backuppool (you might need to attach a virtual disk to do this)
    b. Take a snapshot of the zpool datapool and name it @now
    c. Execute the remote send and recv command over ssh to migrate the snapshot to the pool backuppool (You may need to exchange SSH keys via `ssh-keygen` and `ssh-copy-id` first to make this work)

12. On the zpool named datapool on Ubuntu:

    a. Execute a ```zpool status``` command
    b. Enable LZ4 compression on the zpool datapool
    c. Execute a `zfs get all | grep compression` command to display that compression is enabled

13. On the zpool named datapool, execute a `zpool status` command:

    a. Execute a scrub of the zpool datapool
    b. Create a cron job that executes a zfs scrub on the zpool datapool at 3 am every Sunday morning

14. Using the sample from the text on your Ubuntu 20.04 system, add two additional virtual disks:

    a. Create two partitions on each of these devices
    b. Then using the sample code add these two devices as a log and a cache to the zpool datapool
    c. Execute a `zpool status` command for the zpool named datapool

15. On your Fedora system execute, any of the commands listed to print out the disk serial numbers.

16. Using an Ubuntu system of your choice, create two pair of four 2-GB virtual disks.  Create a ZFS stripe on one of the four disk arrays and create a ZFS equivalent of a RAID 10 (striped mirror) on the other 4 disk array.  Run the command `sudo zpool status` and capture the output.  Name the first zpool, **zstripe** and the second zpool, **zmirror** -- make sure to `chown` to your user `/zstripe` and `/zmirror`

    a. Install the Nginx webserver.  We will modify the file `/etc/nginx/sites-enabled/default` and change the default root setting to serve from the directory `/zstripe`.

17. Attach an additional 2 GB virtual disk and format it with Btrfs and we will mount is in read-only mode. Using the command `lsblk --fs /dev/sdX` determine the UUID of the newest virtual disk you just created.  Add an entry for this disk to the `/etc/fstab` file with the following values:

    a. file system is UUID=
    b. mount point is `/mnt/disk100` (create this partition if it doesn't exist)
    c. type is btrfs
    d. options: defaults,ro  (ro for read-only)
    e. dump and pass fields can be 0
    f. Change owner and group to your username for `/mnt/disk100` (using `chmod`)
    g. Reboot your system. Change directory to `/mnt/disk100` and take a screenshot to demonstrate that the disk is in read-only mode by trying to create a file via this command:  `touch demo.txt`

18. Using an OS of your choice, create 4 2 GB Virtual Disks.  Create a [Btrfs RAID 10](https://btrfs.wiki.kernel.org/index.php/UseCases#How_do_I_create_a_RAID10_striped_mirror_in_Btrfs.3F "btrfs RAID 10") (mirror and stripe) on these four disks. Download one of the Ubuntu 20.04 ISO files onto your Btrfs partition.  Using the [btrfs-replace command](https://btrfs.wiki.kernel.org/index.php/Manpage/btrfs-replace "btrfs-replace"). Add a fifth virtual disk and replace device `/dev/sde` with the new virtual disk.

    a. Run the `btrfs filesystem show` command and capture the output.
    b. Using the UID of the btrfs device created in the previous step, add the mount point to the `/etc/fstab` and add the `nodatacow` attribute. Mount point options are listed here: [btrfs mount-point options](https://btrfs.wiki.kernel.org/index.php/Manpage/btrfs(5) "btrfs mount-point options")

19. Download a copy of Ubuntu 20.04 and when going through the installer, choose the [EXPERIMENTAL erase disk and use zfs](https://ubuntu.com/blog/enhancing-our-zfs-support-on-ubuntu-19-10-an-introduction "ZFS on Ubuntu Root").  Once the install is complete, upon first login, execute the `sudo zpool status` command and capture the output.

20. Using the `wget` command, retrieve this URL: https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.11.19.tar.xz

   a. Untar/uncompress this archive.
   b. Tar the directory and compress it using bzip2, make sure to keep the original input
   c. Tar the directory and compress it using gzip, make sure to keep the original input
   d. Tar the directory and compress it using ztd, make sure to keep the original input
   e. Tar the directory and compress it using xz, make sure to keep the original input

#### Footnotes

[^ch11f121]: [https://btrfs.wiki.kernel.org/index.php/Main_Page](https://btrfs.wiki.kernel.org/index.php/Main_Page "btrfs wiki page")

[^ch11f122]: [http://tldp.org/HOWTO/Partition/fdisk_partitioning.html](http://tldp.org/HOWTO/Partition/fdisk_partitioning.html)

[^ch11f123]: [https://en.wikipedia.org/wiki/Ext2](https://en.wikipedia.org/wiki/Ext2)

[^124]: [https://en.wikipedia.org/wiki/Journaling_file_system](https://en.wikipedia.org/wiki/Journaling_file_system)

[^125]: [https://en.wikipedia.org/wiki/Ext3](https://en.wikipedia.org/wiki/Ext3)

[^126]: [https://en.wikipedia.org/wiki/Ext4](https://en.wikipedia.org/wiki/Ext4)

[^127]: [https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-storage-xfs.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/s-storage-xfs.html)

[^128]: [https://en.wikipedia.org/wiki/Btrfs](https://en.wikipedia.org/wiki/Btrfs)

[^129]: [https://en.wikipedia.org/wiki/ZFS](https://en.wikipedia.org/wiki/ZFS)

[^130]: [http://tldp.org/HOWTO/LVM-HOWTO/whatisvolman.html](http://tldp.org/HOWTO/LVM-HOWTO/whatisvolman.html)

[^131]: [http://tldp.org/HOWTO/LVM-HOWTO/vg.html](http://tldp.org/HOWTO/LVM-HOWTO/vg.html)

[^132]: [http://tldp.org/HOWTO/LVM-HOWTO/pv.html](http://tldp.org/HOWTO/LVM-HOWTO/pv.html)

[^133]: [http://tldp.org/HOWTO/LVM-HOWTO/lv.html](http://tldp.org/HOWTO/LVM-HOWTO/lv.html)

[^134]: <a href="https://commons.wikimedia.org/wiki/File:Super_Talent_2.5in_SATA_SSD_SAM64GM25S.jpg">By photo: Qurren (talk)Taken with Canon IXY 10S (Digital IXUS 210)</a> or <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0 </a>

[^135]: <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0 By Evan-Amos</a> <a href="https://commons.wikimedia.org/wiki/File:Laptop-hard-drive-exposed.jpg">from Wikimedia Commons</a>

[^136]: [https://www.lifewire.com/pci-express-pcie-2625962](https://www.lifewire.com/pci-express-pcie-2625962 "PCi Express")

[^137]: <a href="https://creativecommons.org/licenses/by-sa/4.0">CC BY-SA 4.0 By Anand Lal Shimpi, anandtech.com</a> <a href="https://commons.wikimedia.org/wiki/File:M.2_and_mSATA_SSDs_comparison.jpg">via Wikimedia Commons</a>)

[^138]: [https://docs.oracle.com/cd/E23824_01/html/821-1448/gbchp.html](https://docs.oracle.com/cd/E23824_01/html/821-1448/gbchp.html "Compare the difference between df and zfs list")

[^139]: [http://tldp.org/HOWTO/LVM-HOWTO/snapshotintro.html](http://tldp.org/HOWTO/LVM-HOWTO/snapshotintro.html "TLDP LVM Snapshots")

[^140]: [https://pthree.org/2012/12/06/zfs-administration-part-iii-the-zfs-intent-log](https://pthree.org/2012/12/06/zfs-administration-part-iii-the-zfs-intent-log "ZFS ZIL LOG")

[^141]: [https://pthree.org/2012/12/07/zfs-administration-part-iv-the-adjustable-replacement-cache/](https://pthree.org/2012/12/07/zfs-administration-part-iv-the-adjustable-replacement-cache/ "L2ARC ZFS")

[^142]: [http://www.c0t0d0s0.org/archives/5329-Some-insight-into-the-read-cache-of-ZFS-or-The-ARC.html](http://www.c0t0d0s0.org/archives/5329-Some-insight-into-the-read-cache-of-ZFS-or-The-ARC.html "Original L2ARC cache data")

[^143]: [https://pthree.org/2012/12/11/zfs-administration-part-vi-scrub-and-resilver/](https://pthree.org/2012/12/11/zfs-administration-part-vi-scrub-and-resilver/ "hdparm")

[^144]: <a href="https://creativecommons.org/licenses/by-sa/3.0">CC BY-SA 3.0 By Redline</a> <a href="https://commons.wikimedia.org/wiki/File:HP_EVA4400-1.jpg">from Wikimedia Commons</a>
