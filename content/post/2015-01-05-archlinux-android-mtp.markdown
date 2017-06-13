---
author: yamila
comments: true
date: 2015-01-05 20:07:42+00:00
slug: archlinux-android-mtp
title: ArchLinux, Android, MTP
wordpress_id: 756
post_format:
- Aside
tags:
- android
- archlinux
- mtp
---

Simple protip to connect new Androids to the computer, with the MTP (_media transfer protocol_).

1. Install `libmtp` from official repositories
2. Install `siple-mtpfs` from [AUR repositories](https://aur.archlinux.org/packages/simple-mtpfs/)
3. To use it, as a non-superuser:


```
    $ simple-mtpfs /path/to/mount
    $ cd /path/to/mount
```




It mounts device 0 in the indicated path. Enjoy!
