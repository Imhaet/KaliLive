## FIRST THING FIRST

This is a quick personal reference for creating a bootable Kali USB Drive (Kali Live) with Persistence. For a few years I've wanted to have a live Kali that I can take with me anywhere and be able to boot anywhere. New year, fresh new release (without root as the main user), so this is it.

This guide is mainly based (mostly just copy pasted actually) on Kali's documentation that can be found [here](https://www.kali.org/docs/usb/).

<br />

### What You’ll Need

1. A *verified* copy of the appropriate ISO image of the latest Kali build image for the system you’ll be running it on: see the details on [downloading official Kali Linux Live Boot](https://www.kali.org/get-kali/#kali-live).
   - Don't forget to do a checksum of your file. In Windows run `certutil -hashfile Example.txt SHA256` on the terminal.

2. If you’re running under Windows, you’ll also need to download the [Etcher](https://www.balena.io/etcher/) imaging tool. On Linux and OS X, you can use the dd command, which is pre-installed on those platforms.
   - I am going to do the install using Etcher on Windows.

3. A USB thumb drive, 8GB or larger. (Systems with a direct SD card slot can use an SD card with similar capacity. The procedure is identical.)
   - I am using a 64GB drive because I am interested in adding an encripted persistence partition on it.

---

<br />

## KALI LINUX LIVE USB INSTALL PROCEDURE

The specifics of this procedure will vary depending on whether you’re doing it on a Windows, Linux, or OS X system.
*I am just focusing on the procedure for creating the USB Live in Windows at the moment.*

<br />

### Creating a Bootable Kali USB Drive on Windows

1. Plug your USB drive into an available USB port on your Windows PC, note which drive designator it uses once it mounts (e.g. “F:\“), and launch Etcher.
2. Choose the Kali Linux ISO file to be imaged with “select image” and verify that the USB drive to be overwritten is the correct one. Click the “Flash!” button once ready.
3. Once Etcher alerts you that the image has been flashed, you can safely remove the USB drive and proceed to boot into Kali with it.

The reason I am using Etcher for this instead of any other procedure is because it manipulates the USB drive in a different way than other software like Rufus. If you open the *Disk Management* on Windows you'll notice that Etcher has created 2 partitions on the USB drive, one that is aprox 2.7GB (Kali Live) and has the rest of the drive in an unallocated partition.

<br />

*The rest is done in Linux*
---

<br />

## ADDING PERSISTENCE TO A KALI LINUX "LIVE" USB DRIVE

> Kali Linux “Live” has two options in the default boot menu which enable persistence — the preservation of data on the “Kali Live” USB drive — across reboots of “Kali Live”. This can be an extremely useful enhancement, and enables you to retain documents, collected testing results, configurations, etc., when running Kali Linux “Live” from the USB drive, even across different systems. The persistent data is stored in its own partition on the USB drive, which can also be optionally LUKS-encrypted.

> To make use of the USB persistence options at boot time, you’ll need to do some additional setup on your “Kali Linux Live” USB drive; this article will show you how.

> This guide assumes that you have already created a Kali Linux “Live” USB drive as described in [the doc page for that subject](https://www.kali.org/get-kali/#kali-live). For the purposes of this article, we’ll assume you’re working on a Linux-based system.

--- [Kali.org/](https://www.kali.org/docs/usb/usb-persistence/) ---

<br />

### Adding USB Persistence with LUKS Encryption

We'll be creating a LUKS-encrypted persistent storage area. The idea is to add an extra layer of security to any sensitive file when traveling with Kali Live. We'll create a new partition to store the persistent data into, starting right above the second Kali Live partition for the rest of the USB, set up LUKS encryption on the new partition, put an ext3 file system onto it, and create a **persistence.conf** file on it. All this based on Kali's [instructions](https://www.kali.org/docs/usb/usb-persistence-encryption/).

1. Image the latest Kali Linux ISO (currently 2020.1) to your USB drive as described in [this article](https://www.kali.org/docs/usb/kali-linux-live-usb-install/) (or as above).

2. Make sure that your Kali Live USB runs fine by booting into it at least once.

3. Create the new partition in the empty space above our Kali Live partitions. Run the comand `fdisk -l` to verify the USB drive's name in Linux. We'll continue with the assumption that it is **/dev/sdb**. We'll be using the command `cfdisk` to create the new partition since some graphical tools will see the drive as just the ISO.

```
sudo cfdisk /dev/sdb
```

4. Using the arrow keys on your keyboard to navigate, go to the bottom `Free space` in the device list and select `New`.

5. *Enter* to select the partition size.

6. *Enter* again so select `primary` as the partition type. The partition is now named **/dev/sdb3** with a **Linux** type.

7. Select `Write` and type `yes` to confirm the changes; finally `Quit`.

8. **Time for the encryption**. Initialize the LUKS encryption on the newly-created partition.
```
#: cryptsetup --verbose --verify-passphrase luksFormat /dev/sdb3
```
You’ll be warned that this will overwrite any data on the partion. When prompted whether you want to proceed, type “YES” (all upper case). Enter your selected passphrase twice when asked to do so, and be sure to pick a passphrase you’re going to remember: if you forget it, your data will still be persistent, just irretrievable (and unusable).

9. Open the encrypted channel to our persistence partition. I am using `my_usb` as the label, but you can use whatever you want.
```
#: cryptsetup luksOpen /dev/sdb3 my_usb
```

10. Now we need to format the partition. Create the ext3 filesystem, and label it “persistence”.
```
#: mkfs.ext3 -L persistence /dev/mapper/my_usb
#: e2label /dev/mapper/my_usb persistence
```

11. Create a mount point, mount our new encrypted partition there.
```
#: mkdir -p /mnt/my_usb
#: mount /dev/mapper/my_usb /mnt/my_usb\
```

12. Create and set up the persistence.conf file. I'm using `nano` here because `echo` has rights problems on my system. Add the line `/ union` on the file, **save** :floppy_disk: the file as `/mnt/my_usb/persistence.conf` and exit.
```
#: nano /mnt/my_usb/persistence.conf
```
```
/ union
```

13. Unmount the partition.
```
#: umount /dev/mapper/my_usb
```

14. Close the encrypted channel to our persistence partition.
```
#: cryptsetup luksClose /dev/mapper/my_usb
```

15. Eject the usb stick
```
#: eject /dev/sdb
```

<br />

That’s really all there is to it! To use the persistent data features, simply plug your USB drive into the computer you want to boot up Kali Live on — make sure your BIOS is set to boot from your USB device — and fire it up. When the Kali Linux boot screen is displayed, choose the persistent option you set up on your USB drive, either normal or encrypted.

* Note: If you need to change the passphrase on the LUKS drive, open a terminal and run the `sudo cryptsetup luksChangeKey /dev/sdb3` command. First, you’ll be prompted to enter your existing passphrase. Then, you can create a new one.

:tada:

<br />

---
<br />

# KALI PROCEDURES #

### Reset Windows 10 Local Password

First, your Win10 might not be set up to boot from the USB, so you may need to press a function key (e.g., Esc, F2, F8, F10, etc.) to get into the **BIOS** to change the boot order, then restart again.
1. After booting from USB, the Kali Linux Boot Menu appears. Choose `Live (forensic mode)`
2. Open a terminal with root privileges and navigate to the directory where SAM is saved, usually */Windows/System32/Config*. (e.g., */media/root/44B85...D34/Windows/System32/config/*)
3. Type `chntpw -l SAM`, it will show a lilst of usernames found in the SAM or Win10.
4. Run the `chntpw -u username SAM` to reset the password. Change *username* to the actual name of the Windows account.
5. Press `1` to clear the user's password, then `q` to quit and finally `y` to save changes to the SAM.
6. Reboot and log into windows, the user should log in without a password now.

:octocat:
