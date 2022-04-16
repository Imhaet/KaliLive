## FIRST THING FIRST

This is a quick personal reference for creating a bootable Kali USB Drive (Kali Live) with Persistence. For a few years I've wanted to have a live Kali that I can take with me anywhere and be able to boot anywhere. New year, fresh new release (without root as the main user), so this is it.

This guide is mainly based (mostly just copy pasted actually) on Kali's documentation that can be found [here](https://www.kali.org/docs/usb/kali-linux-live-usb-install/).

<br />

### What You’ll Need

1. A *verified* copy of the appropriate ISO image of the latest Kali build image for the system you’ll be running it on: see the details on [downloading official Kali Linux images](https://www.kali.org/docs/introduction/download-official-kali-linux-images/).



---
<br />
# FORMATING A USB FOR FLASHING AN ISO #

### In Linux KDE ###
sudo fdisk -l - to figure out the disk
sudo umount  /dev/sdb1 - just in case
sudo fdisk /dev/sdb
Command (m for help): 0 -create new empty DOS partition
Comand (m helo): n -add a new partition
Select (default p): p   -primary
Partition number (1-4, default 1): 1
First sector (x,y): ENTER
Last sector (x,y): ENER
w


In case you can't get your device formatted from the GUI, try this way.

    Open the Terminal (Ctrl+Alt+T)

    List your block storage devices by issuing the command lsblk
    Then identify your pen drive by it's SIZE. In my case its /dev/sdb

    enter image description here

    Erase everything in the pen drive (This step is Optional):

    sudo dd status=progress if=/dev/zero of=/dev/sdb bs=4k && sync  

    Replace /dev/sdb with your corresponding device.

    Type very carefully this name or your may end up erasing one of your other disks. This will take some time. (option status=progress is not mandatory but provide you some feedback)

    It will pretend to be stuck. Just be patient.
    for example:

dd if=/dev/zero of=/dev/sdb bs=4k && sync
dd: error writing '/dev/sdb': No space left on device

1984257+0 records in
1984256+0 records out
8127512576 bytes (8.1 GB) copied, 1236.37 s, 6.6 MB/s

Make a new partition table in the device:

sudo fdisk /dev/sdb

Then press letter o to create a new empty DOS partition table.

Make a new partition:

    Press letter n to add a new partition. You will be prompted for the size of the partition. Making a primary partition when prompted, if you are not sure.

    Then press letter w to write table to disk and exit.

Format your new partition.

    See your new partition label with the command lsblk
    In my case it is /dev/sdb1. Once again pay attention to this name as there will not be any protection to prevent you to erase an other disk.


    Issue the command below to format the new volume:

    sudo mkfs.vfat /dev/sdb1  

    Please replace /dev/sdb1 with your corresponding device.

    Eject the device:

    sudo eject /dev/sdb

