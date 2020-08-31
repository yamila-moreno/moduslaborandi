---
author: yamila
title: Instalación de Arch con cifrado
slug: instalacion-arch-2020
date: 2020-06-25 08:00:00+00:00
comments: true
tags:
- archlinux
- tutorial
---

De nuevo tengo un ordenador para estrenar entre mis manos, así que de nuevo realizo el proceso de instalación manual. Esta vez, además, tenía como tarea cifrar el disco, así que, como en ocasiones anteriores, detallo los pasos para quien pueda ser útil.

<!--more-->

**Atención** Este no es un tutorial básico; aunque va paso a paso, si acabas de empezar con Linux o con Arch seguramente se te haga cuesta arriba y no entiendas muchos conceptos.

**Nota** La instalación la hice en mi nuevo Lenovo T495. Además, durante la instalación vi que había varias formas de cifrado y la que me funcionó fue el cifrado con luks. Además, la instalación se basa en arranque UEFI y el esquema de particionado GPT.

¡Empezamos!

### Imagen de ArchLinux
Descarga la [iso de Arch](https://www.archlinux.org/download/) y métela en un USB:
```sh
(root) $ dd bs=4M if=/path/to/archlinux-dual.iso of=/dev/sdc && sync
```
Para poder arrancar desde el USB, asegúrate de que `Secure Boot` en la BIOS está deshabilitado y puede arrancar el ordenador desde el USB.

### Settings Básicos
```sh
$ loadkeys es
$ timedatectl set-ntp true
```

### Conectar a la wifi
```sh
$ wifi-menu
$ echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```

### Crear tabla de particiones
```sh
$ lsblk # para ver qué discos (ojo, /dev/sda será el propio USB)
$ gdisk /dev/xxxxx # primero borrar las particiones existentes con la opción `o`
```
- crear una partición para boot ef00 de 1gb
- crear una partición para swap 8200 de 2gb
- crear una partición para el resto 8300 con el tamaño restante
- command `w` para escribir y salir

### Cifrar el disco principal
```sh
$ cryptsetup luksFormat --type luks1 /dev/nvme0n1p3 (el disco grande)
```
En este punto aviso de que sólo he conseguido que funcione con luks1.
**OJO**: aquí pide una contraseña, pero asegúrate de configurar bien el keymap para los caracteres especiales, o pon una contraseña segura sin caracteres especiales.
```sh
$ cryptsetup open /dev/nvme0n1p3 cryptroot # ahora tenemos /dev/mapper/cryptroot
$ mkfs.ext4 /dev/mapper/cryptroot
$ mount /dev/mapper/cryptroot /mnt
```

### Configurar Boot
```sh
$ mkfs.fat -F32 /dev/nvw..  # (la primera partición, la de boot)
$ mkdir /mnt/boot/
$ mount /dev/nvme0n1p1 /mnt/boot
```
Es importante que esté en `/boot` y no en `/boot/efi` como pone algunos tutoriales que no cifran disco.

### Instalar Linux
```sh
$ pacstrap /mnt base linux linux-firmware base-devel efibootmgr vim netctl dialog wpa_supplicant dhcpcd
```
Instalamos además otros paquetes imprescindibles para disponer de wifi-menu.
Nota: Si va lento o da timeout, edita `/etc/pacman.d/mirrorlist` y pon arriba del todo otra fuente (Belgium, France...)

### Generar fstab
```sh
$ genfstab -U /mnt >> /mnt/etc/fstab
```
Ver que aquí está puesto el UUID de `/dev/mapper/cryptroot`; en `/etc/fstab` se indican los discos ya descifrados.

### Entrar en el sistema
```sh
$ arch-chroot /mnt
```

### Preparar la partición Swap
Para que la partición de swap funcione como una partición cifrada, hay que añadirla a `/etc/crypttab`; el sistema usará una password random que se descarta al apagar con lo que la partición de swap queda también cifrada.
```sh
$ vim /etc/crypttab
```
En este fichero hay que descomentar la línea que pone `swap` y cambiar en esa línea el device por el correspondiente device cifrado
(según este tutorial era /dev/nvme0n1p2)

Después hay que añadir la línea de swap a `/etc/fstab`; de nuevo en `/etc/fstab` el disco de dev/mapper que es el que está ya descifrado.
```
# /dev/mapper/swap
/dev/mapper/swap            none           swap           defaults      0      0
```
Es posible que esto requira reiniciar para coger los cambios.

### Configurar timezone
```sh
$ ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
$ hwclock --systohc
```

### Configurar hostname
```sh
$ vim /etc/hostname
```
Y poner el hostname que queráis.

### Configurar locales
Editar el fichero `/etc/locale.conf` y escribir algo similar a (con tus propios valores):
```
LANG="en_US.UTF-8"
LC_TIME="es_ES.UTF-8"
LC_COLLATE="C"
```

Después, editar `/etc/vconsole.conf` y añadir lo siguiente:
```
KEYMAP=es
```

Finalmente, editar `/etc/locale.gen`; en este fichero hay que descomentar los locales que quieras tener disponibles. Como mínimo hacen falta los locales usados en `locale.conf` un poco más arriba:

Después de configurar, hay que generar los locales:
```sh
$ locale-gen
```

### Preparar la boot image
Editar el fichero `/etc/mkinitcpio.conf` y poner:
```
MODULES=(ext4)
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
```

OJO: es importante el orden de los hooks. Tras esto:
```sh
$ mkinitcpio -p linux
```

### Configurar una password de root
```sh
$ passwd
```

### Actualizar microcode
Instalar el paquete con actualizaciones de seguridad al microcódigo del procesador Intel
https://wiki.archlinux.org/index.php/Microcode
```sh
$ pacman -S intel-ucode
```

El grub detectará que este paquete existe y añadirá automáticamente una línea para activarlo cuando genere el fichero con grub-mkconfig. Para ver si se ha cargado bien, una vez arrancamos linux en host final, escribimos
```sh
$ dmesg | grep microcode
```

### Configurar Grub
```sh
$ pacman -S grub
$ blkid > borrar # para guardar temporalmente los uuids y poder copiarlo después
# del fichero `borrar` puedes coger el UUID de /dev/nvem0n1p3 ojo, el uuid y no el ptuuid
$ vim /etc/default/grub
GRUB_CMDLINE_LINUX="cryptdevice=UUID=disc-uuid:cryptroot"
# siendo el disc-uuid el uuid del disco sin cifrar ojo, no el cryptroot. Guardar y:
$ grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck
$ grub-mkconfig -o /boot/grub/grub.cfg
```

### Salir y reiniciar
```sh
$ exit
$ umount -R /mnt
$ reboot
```

Y voilá! Con esto está la instalación hecha. Al reiniciar, te pedirá la contraseña para descifrar el disco y después el login normal (user/pass). Una vez que la instalación está hecha, vendría instalar el entorno gráfico y personalizaciones. En mi caso, instalé [DWM](https://dwm.suckless.org/)

¡Espero que os sea de utilidad!
