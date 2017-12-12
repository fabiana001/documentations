# Handle Logical Volumes with lvm

LVM (Logical Volume Manager) is a system of managing logical volumes, or filesystems, that is much more advanced and flexible than the traditional method of partitioning a disk into one or more segments and formatting that partition with a filesystem.

There are three concepts that lvm manages:
1. Physical Volumes: multiple hard disks can be associated to one o multiple physical volumes. A physical volume can be associated to only one hard disk.
2. Volume Group: physical volumes (with the exception of the */boot* partition) can be combined and associated to a volume group.  
3.  Logical Volumes:  a volume group can be partitioned in logical volumes. Logical volumes correspond to partitions, i.e. they hold a filesystem. Unlike partitions though, logical volumes get names rather than numbers, they can span across multiple disks, and do not have to be physically contiguous. When a physical volume became full you can add free space of the volume group to a physical volume. Moreover, when a volume group reaches is max capacity, it is possible add on the fly new physical volumes to a volume group.

**Note** that hard disks used in the production cluster are not managed by lvm. In the following, it will be described how extending a volume.

## Extend a volume with lvm
Create a new physical volume, e.g. /dev/vdd:
>$ pvcreate /dev/vdd 
 
Check all physical volumes:
> $ pvdisplay
This command checks how your LVM is set up.

Create a new virtual group, e.g. kafka01:
> $ vgcreate kafka01 /dev/vdd

and check if it is correctly created using the command *vgs* or *vgdisplay*

Create a new logical volume, e.g kafka:
>$ lvcreate -l 100%VG -n kafka kafka01

where *-l 100%VG* means that the logical volume will use all the space of the virtual group.

Therefore the last step is to format the new logical volume with a file system:

> $ mkfs.xfs /dev/mapper/kafka01-kafka 

Create a mount point and then mount the volume somewhere you can use it. However, before to make this step, we need to copy all data of the volume we want extending (e.g. /kafka).


> $ mount /dev/mapper/kafka01-kafka /mnt/
> 
> $ rsync -av --progress /kafka/* /mnt/ #kafka is the src directory

Now, we want to associate the dir */kafka* with the new physical volume. To do this, we have to edit the file */etc/fstab* with this instruction:

> /dev/mapper/kafka01-kafka /kafka xfs noatime 0 0

that is, we are associating the dir */kafka* with the new logical volume *dev/mapper/kafka01-kafka*, rather the old volume (e.g. /dev/vdc1)

Finally, we can umount the old volume and restart the machine

> $ umount /dev/vdc1

> $ shutdown now -r
 
 or if we cannot restart the machine, execute the following commands:
> $ umount /dev/vdc1

> $ umount /dev/mapper/kafka01-kafka 

> $ mount /dev/mapper/kafka01-kafka /kafka

## References

[1] [Logical Volume Manager (LVM)](http://web.mit.edu/rhel-doc/3/rhel-sag-it-3/ch-lvm-intro.html)
