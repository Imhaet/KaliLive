## FORMATING A USB FOR FLASHING AN ISO

### In Linux Terminal KDE

1. List your block storage devices by issuing the command `lsblk`, `fdisk -l`, or `df` if mounted.
2. Then identify your pen drive by it's SIZE. In my case its `/dev/sdb`.
3. If the drive is mounted, unmount it.
> ```
> #: umount /dev/sdb1
> ```
4. Erase everything in the pen drive (This step is Optional):
> ```
> #: dd status=progress if=/dev/zero of=/dev/sdb bs=4k && sync  
> ```
5. Make a new partition table in the device:
> ```
> #: fdisk /dev/sdb
> ```
6. Then press letter `o` to create a new empty DOS partition table.
7. Make a new partition:
> - Press letter `n` to add a new partition. You will be prompted for the size of the partition. Making a primary partition when prompted, if you are not sure.
> - Then press letter `w` to write table to disk and exit.
8. Format your new partition.
  - See your new partition label with the command `lsblk`. In my case it is `/dev/sdb1`. Pay attention to this name as there will not be any protection to prevent you to erase an other disk.
  - Issue the command below to format the new volume:
> ```
> #: mkfs.vfat /dev/sdb1 -L "Silver"
> ```
9. Eject the device:
> ```
> #: eject /dev/sdb
> ```
