---
author: yamila
comments: true
date: 2014-12-22 09:19:41+00:00
slug: archlinux-installation
title: ArchLinux installation
wordpress_id: 652
tags:
- archlinux
- kaleidos.net
- my modus
- open source
---

Two years ago (TWO YEARS!! auch) I wrote this tutorial of [Archlinux installation](http://www.kaleidos.net/blog/18/instalar-archlinux-de-novato-a-novato/) (in Spanish). At that time, I kept using MBR instead of GPT (recommended).

Some weeks ago, [Kaleidos](http://kaleidos.net) renewed our computers, so I had to install [ArchLinux](https://www.archlinux.org/) again. It happened I tried several times before finding the "key" (dammed key!). So here is a new brand tutorial to install Archlinux with GPT to save some of your precious time.
<!-- more -->

**Disclaimer**: this tutorial covers the manual installation of ArchLinux. If you want to make it easier, I recommend you to try [Manjaro Linux](http://manjaro.org/), which is an Arch with an assistant for the installation.

**Disclaimer**: some of the readers may have their own locale configuration; this tutorial is set in spanish, and it's your responsability to choose your own language settings.

1) Get the **source**

You can download the iso from the [official page](https://www.archlinux.org/download/). Then you need to copy to a pendrive:


```
    dd bs=4M if=/path/to/archlinux-dual.iso of=/dev/sdc && sync
```



**Achtung!!**: I wrote _/dev/sdc_ but it's your responsability to find the device (maybe it's in _/dev/sdb_). Also, you don't copy into a specific partition inside the drive but in the whole drive; and all the information in the drive will be deleted with this command.

2) **Load** this OS from USB

3) Set the **keyboard display**

    $ loadkeys es

4) Let's make the **partition table**

    $ gdisk /dev/sda # usually is /dev/sda but make sure!

    # and in the interactive console
    > ? # for help
    > n # for new partition
    > particion number # enter for recommended
    > starting sector (xxx) # enter for recommended sector
    > ending sector # +1MB
    > type # ef02 (bios boot)

In my case, I created these partitions:

  * /dev/sda1 - 1Mb - EF02 (bios boot). **this is a mandatory partition for GPT**
  * /dev/sda2 - 2Gb - 8200 (swap)
  * /dev/sda3 - 50Gb - 8300 (linux system). This is the **/** (root) particion
  * /dev/sda4 - 250Gb - 8302 (home). This is the **/home** partition

After creating the partitions, enter **w** to save changes.

5) **Format** the partition table and enable swap
Given the former partition table, you will need to formart partitions 3 and 4:

    $ mkfs.xfs /dev/sda3
    $ mkfs.xfs /dev/sda4




Also, you need to enable the swap partition:



    $ mkswap /dev/sda2 # set the swap area
    $ swapon /dev/sda2 # Enable swap.
    # This last step is optional because systemd
    # will enable automatically the swap after reboot.




6) **Mount** the partitions


    $ mount /dev/sda3 /mnt
    $ mkdir /mnt/home
    $ mount /dev/sda4 /mnt/home




7) **Let's install!**
For this step you need internet. If you use ethernet connection, the computer is connected automatically; if you use WiFi, you can use **wifi-menu** to connect to a network. And then:


    $ pacstrap /mnt base base-devel



(you may wait a little... be patient)

8) Done! Now we are going to create a **fstab** file


    $ genfstab -Up /mnt > /mnt/etc/fstab



9) We leave the ISO environment and **enter the fresh installation**


    $ arch-chroot /mnt



**Congratulations!! You have just installed ArchLinux**. Now it's time to grab a coffee and walk the dog. Next steps are mostly about configuration and details.

10) Set the **hostname** of the machine


    $ vi /etc/hostname
    # add the name for your machine. I chose "aran"



11) Set the **timezone** (Spain for my case)


    $ ln -s /usr/share/zoneinfo/Europe/Madrid /etc/localtime



12) Configure **locales**


    vi /etc/locale.conf
    # and write the following
    LANG="en_US.UTF-8"
    LC_TIME="es_ES.UTF-8"
    LC_COLLATE="C"

    vi /etc/vconsole.conf
    # and add the following
    KEYMAP=es

    vi /etc/locale.gen
    # In this file, it's necessary to uncomment
    # the locales we want to be available;
    # at least, we need to uncomment the
    # locales used in the `locale.conf` file.



After this editions:



    $ locale-gen



13) We need to create a **boot image**


    $ vi /etc/mkinitcpio.conf
    # we add necessary MODULES. At least,
    # we have to add all filesystems we are using
    MODULES="libata ext3 ext4 xfs"



After this edition:


    $ mkinitcpio -p linux



14) Now, it's **grub** time


    $ pacman -S grub
    $ grub-install --target=i386-pc --recheck /dev/sda



After installing, we create the configuration:


    $ grub-mkconfig -o /boot/grub/grub.cfg



15) Set **root password**


    $ passwd
    # and enter a new and secure password, for instance dragon,
    # mom, password or the very very secure p4ssw0rd



(well, it's a joke, those are not definitely secure passwords)

16) And...


    $ exit
    $ umount /dev/sda4
    $ umount /dev/sda3
    $ shutdown -r now # reboot!!



As you can see, there are some steps, but all of them are small, so be patient and read carefully. Would you like to continue? :)

17) Create **new user** and give it some privileges group related


    $ useradd -m -g wheel -s /bin/bash yami
    $ usermod -aG network yami
    $ usermod -aG video yami
    $ usermod -aG audio yami
    $ usermod -aG optical yami
    $ usermod -aG storage yami
    $ usermod -aG power yami
    $ usermod -aG dbus yami



18) Enable **multilib repository** for pacman


    $ vi /etc/pacman.conf

    # and uncomment next lines
    #[multilib]
    #SigLevel = PackageRequired
    #Include = /etc/pacman.d/mirrorlist



And update the system:


    $ pacman -Syu



And that's all! From this point, you may want to install a _desktop environment_ like [Gnome](http://www.gnome.org/) or a window manager like [DWM](http://dwm.suckless.org/), and your themes and favourite applications.
