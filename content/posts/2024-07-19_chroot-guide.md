+++
title = "Chroot: Your friend when it comes to troubleshooting and recovering broken linux installations."
date = 2024-07-19
+++

TODO: blah blah blah stuff about explaining what we will do today.

# Prerequisites:

- A bootable USB drive with a linux distro on it.
- Using the same linux distro as the broken install is not necessary but recommended, since we are trying to recover a broken install.
- Using the same architecture is necessary. You cannot chroot into an ARM machine with a x86_64 based linux bootable USB, or vice-versa.

# Steps:

## Step 1: Preparation

- Poweroff the machine if it is on.
- Go to BIOS and disable secure boot if possible. Some linux distros do not
  have "trusted certificates", so we'll just disable this for now. You may keep
  this on if the distro on your live environment distro has signed certificates
  for secure boot.

## Step 2: Starting the Live Environment

- Plug in your bootable usb
- Power on your machine
- Go to boot menu (in most machines you have to press a certain key when the machine is powering on)
- Select your bootable usb
- Boot into the live environment (In most distros this will be the default "Try or install" option)

## Step 3: Locating the necessary partitions

- Run `fdisk -l` or `lsblk` to see the partitions on the disk.
- We can identify partitions by disk sizes (lsblk command outputs a nice looking tree-like structure for our disks with sizes)
- If you chose to go with `fdisk`, it doesn't directly give us partition sizes
  but it does give us file system type, which can help us identify the ESP (EFI
  System Partition (also called boot partition)) and other partitions.

## Step 4: Identifying the home and root partitions

- This step requires a knowledge about the way the linux was installed on the
  host machine. It is neccessary to know whether we had a seperate home
  partition or not.
- Most linux installations have either a 2 partition install (/ (root
  partition) and /boot (ESP)) or a 3 partition install (/ (root partition),
  /boot (ESP), and /home (home partition)).
- If you are unaware of which one is home and which one is root, you can just
  mount (mounting is discussed later, just replace `/mnt` with any directory
  you want and run the mount command) both to some temporary path and look for
  `/etc/fstab` file on either of the mountpoints (if your system is really
  broken, this file may not even exist so try to identify other directories
  that usually only exist in root partition, like /var, /sys, etc.), upon
  looking at the contents of this file you can see what all gets mounted on
  every boot, you can ignore the SWAP partition as it's not a necessity.
- The home partition will typically have a bunch of directories corresponding
  to linux usernames of their respective users.
- The root partition will have directories like `/etc`, `/var`, `/bin` and so
  on. If you have a 2 partition install, it may also have a `/home` directory.

## Step 5: Mounting the partitions

- Create the necessary directories:

> For a 2 partition install:
>
> ```bash
> mkdir -p /mnt/home
> ```

> For a 3 partition install:
>
> ```bash
> mkdir -p /mnt/{home,boot}
> ```

> This creates a directory /mnt, and a subdirectory /mnt/boot (and /mnt/home if
> you have a 3 partition install)

- Mount the partitions:

> First we must mount the root partition (for example our root partition is /dev/sda2)
>
> ```bash
> mount /dev/sda2 /mnt
> ```

> Next we must mount our boot partition (ESP) (for example our boot partition is /dev/sda1)
>
> ```bash
> mount /dev/sda1 /mnt/boot
> ```

> If we have a home partition (3 partition installs only), we must mount that
> too. (for example our home partition is on a separate hard disk at /dev/sdb,
> so our partition is /dev/sdb1)
>
> ```bash
> mount /dev/sdb1 /mnt/home
> ```

- Mount (bind) the necessary system directories:

> /dev has files representing our hardware devices
>
> ```bash
> mount --bind /dev /mnt/dev
> ```

> /proc has files representing processes
>
> ```bash
> mount --bind /proc /mnt/proc
> ```

> /sys has files representing the kernel
>
> ```bash
> mount --bind /sys /mnt/sys
> ```

## Step 6: Chroot

- This is the final step, now we can chroot into the root partition and start troubleshooting/recovery.

> ```bash
> chroot /mnt
> ```
