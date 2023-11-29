## FORMATING A USB FOR FLASHING AN ISO

### In Linux Terminal

1. List your block storage devices by issuing the command `lsblk`, `fdisk -l`, or `df` if mounted.
2. Then identify your pen drive by it's SIZE. I'll use `/dev/sdx` from here onwards.
3. If the drive is mounted, unmount it.
> ```
> #: umount /dev/sdx1
> ```
4. Erase everything in the pen drive by putting zeros with dd (This step is Optional):
Note: You can check what your disk cache/buffer size is by using `#: hdparm -i /dev/sdx`
> ```
> #: dd if=/dev/zero of=/dev/sdx bs=4k status=progress
> ```
5. Make a new partition table in the device:
> ```
> #: fdisk /dev/sdx
> ```
6. Then press letter `o` to create a new empty DOS partition table.
7. Make a new partition:
> - Press letter `n` to add a new partition.
> - Press `Enter` to use for the size of the partition. Making a primary partition when prompted, if you are not sure.
> - Then press letter `w` to write table to disk and exit.
8. Format your new partition.
  - See your new partition label with the command `lsblk`. In my case it is `/dev/sdx1`. Pay attention to this name as there will not be any protection to prevent you to erase an other disk.
  - Issue the command below to format the new volume:
> ```
> #: mkfs.vfat /dev/sdx1 -L "LABEL NAME"
> ```
9. Eject the device:
> ```
> #: eject /dev/sdx
> ```

### In Windows

1. Open Command Prompt as Administrator.
2. Use `diskpart` on the command prompt.
3. Use `list disk` to identify your disk by the size.
4. Select the drive to format with `select disk 2` (in this case the name of the drive is Disk 2).
5. You can make sure you have selected the drive if you run `list disk` again.
6. Erase all partitions with the `clean` command.
7. Convert from Master Boot Record partition scheme to GUID Partition Table with `convert gpt`.
8. Create a new partition with the `create partition primary` command.
9. Format the drive with the desired file system:
> ```
> format fs=ntfs
> ```
9. That is it, you can eject your new drive.
