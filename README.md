## FIRST THING FIRST

This is a quick personal reference for creating a bootable Kali USB Drive (Kali Live) with Persistence. For a few years I've wanted to have a live Kali that I can take with me anywhere and be able to boot anywhere. New year, fresh new release (without root as the main user), so this is it.

This guide is mainly based (mostly just copy pasted actually) on Kali's documentation that can be found [here](https://www.kali.org/docs/usb/kali-linux-live-usb-install/).

<br />

### What You’ll Need

1. A *verified* copy of the appropriate ISO image of the latest Kali build image for the system you’ll be running it on: see the details on [downloading official Kali Linux images](https://www.kali.org/docs/introduction/download-official-kali-linux-images/).

2. If you’re running under Windows, you’ll also need to download the [Etcher](https://www.balena.io/etcher/) imaging tool. On Linux and OS X, you can use the dd command, which is pre-installed on those platforms.
   - I am going to do the install using Etcher on Windows.

3. A USB thumb drive, 4GB or larger. (Systems with a direct SD card slot can use an SD card with similar capacity. The procedure is identical.)
   - I am using a 32GB drive because I am interested in adding an encripted persistence partition on it.

---

<br />

## KALI LINUX LIVE USB INSTALL PROCEDURE

The specifics of this procedure will vary depending on whether you’re doing it on a Windows, Linux, or OS X system. *I am just focusing on the procedure for creating the USB Live on Windows at the moment.*

<br />

### Creating a Bootable Kali USB Drive on Windows

1. Plug your USB drive into an available USB port on your Windows PC, note which drive designator (e.g. “F:\“) it uses once it mounts, and launch Etcher.
2. Choose the Kali Linux ISO file to be imaged with “select image” and verify that the USB drive to be overwritten is the correct one. Click the “Flash!” button once ready.
3. Once Etcher alerts you that the image has been flashed, you can safely remove the USB drive and proceed to boot into Kali with it.

The reason I am using Etcher for this instead of any other procedure is because it manipulates the USB drive in a different way than other software like Rufus. If you open the *Disk Management* on Windows you'll notice that Etcher has created 2 partitions on the USB drive, one that is aprox 2.7GB (Kali Live) and has the rest of the drive in an unallocated partition.

---

<br />

## ADDING PERSISTENCE TO A KALI LINUX "LIVE" USB DRIVE

> Kali Linux “Live” has two options in the default boot menu which enable persistence — the preservation of data on the “Kali Live” USB drive — across reboots of “Kali Live”. This can be an extremely useful enhancement, and enables you to retain documents, collected testing results, configurations, etc., when running Kali Linux “Live” from the USB drive, even across different systems. The persistent data is stored in its own partition on the USB drive, which can also be optionally LUKS-encrypted.

> To make use of the USB persistence options at boot time, you’ll need to do some additional setup on your “Kali Linux Live” USB drive; this article will show you how.

> This guide assumes that you have already created a Kali Linux “Live” USB drive as described in [the doc page for that subject](https://www.kali.org/docs/usb/kali-linux-live-usb-install/). For the purposes of this article, we’ll assume you’re working on a Linux-based system.

--- [Kali.org/](https://www.kali.org/docs/usb/kali-linux-live-usb-persistence/) ---

<br />

### Adding USB Persistence with LUKS Encryption

We'll be creating a LUKS-encrypted persistent storage area. The idea is to add an extra layer of security to any sensitive file when traveling with Kali Live. We'll create a new partition to store the persistent data into, starting right above the second Kali Live partition for the rest of the USB, set up LUKS encryption on the new partition, put an ext3 file system onto it, and create a **persistence.conf** file on it. All this based on Kali's [instructions](https://www.kali.org/docs/usb/kali-linux-live-usb-persistence/).

1. Image the latest Kali Linux ISO (currently 2020.1) to your USB drive as described in [this article](https://www.kali.org/docs/usb/kali-linux-live-usb-install/).

2. Make sure that your Kali Live USB runs fine by booting into it at least once.

3. Create the new partition in the empty space above our Kali Live partitions. Run the comand `fdisk -l` to verify the USB drive's name in Linux. We'll continue with the assumption that it is **/dev/sdb**. We'll be using the command `cfdisk` to create the new partition since some graphical tools will see the drive as just the ISO.

```
sudo cfdisk /dev/sdb
```

4. Using the arrow keys on your keyboard to navigate, go to the bottom `Free space` in the device list and select `New`.

5. *Enter* to select the partition size.

6. *Enter* again so select `primary` as the partition type. The partition is now named **/dev/sdb3** with a **Linux** type.

7. Select `Write` and then type `yes` to confirm the changes; finally `Quit`.

8. **Time for the encryption**. Initialize the LUKS encryption on the newly-created partition. You’ll be warned that this will overwrite any data on the partion. When prompted whether you want to proceed, type “YES” (all upper case). Enter your selected passphrase twice when asked to do so, and be sure to pick a passphrase you’re going to remember: if you forget it, your data will still be persistent, just irretrievable (and unusable).

```
sudo cryptsetup --verbose --verify-passphrase luksFormat /dev/sdb3
sudo cryptsetup luksOpen /dev/sdb3 my_kali
```

9. Now we need to format the partition. Create the ext3 filesystem, and label it “persistence”.

```
sudo mkfs.ext3 -L persistence /dev/mapper/my_kali
sudo e2label /dev/mapper/my_kali persistence
```

10. Create a mount point, mount our new encrypted partition there.

```
sudo mkdir -p /mnt/my_kali
sudo mount /dev/mapper/my_kali /mnt/my_kali
```

11. Create and set up the persistence.conf file. We're using `nano` here because `echo` has rights problems on my system. Add the line `/ union` on the file save and exit.

```
sudo nano /mnt/my_kali/persistence.conf
```
```
/ union
```

12. *Ctrl+X* to **exit**, then **save** :floppy_disk: the file as `/mnt/my_kali/persistence.conf`

13. Unmount the partition.

```
sudo umount /dev/mapper/my_kali
```

14. Close the encrypted channel to our persistence partition.

```
sudo cryptsetup luksClose /dev/mapper/my_kali
```

<br />

:tada: That’s really all there is to it! To use the persistent data features, simply plug your USB drive into the computer you want to boot up Kali Live on — make sure your BIOS is set to boot from your USB device — and fire it up. When the Kali Linux boot screen is displayed, choose the persistent option you set up on your USB drive, either normal or encrypted.
