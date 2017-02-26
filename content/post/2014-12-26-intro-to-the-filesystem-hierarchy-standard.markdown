---
author: lekum
comments: true
date: 2014-12-26 11:05:04+00:00
layout: post
slug: intro-to-the-filesystem-hierarchy-standard
title: Intro to the Filesystem Hierarchy Standard
wordpress_id: 708
categories:
- General
---

Have you ever wondered why some programs are installed in /bin, /sbin, /usr/bin or /opt? Or why there is a /tmp directory? The answer is in the [Filesystem Hierarchy Standard](http://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard), a standard whom most Linux and Unix-like OS's adhere to.<!-- more -->

Although it is a widespread standard, direct succesor of the Filesystem Standard (FSSTND) some Linux distribution don't follow it strictly. However, the standard provides general guidelines about the distribution of directories in the filesystem tree, and very specific details about some critical directories, that are generally accepted and thus implemented.

The standard, which can be accesed [here](http://www.pathname.com/fhs/), defines these properties of a directory:




  * It can have **static** files (that don’t normally change except through direct intervention by the system administrator) or **variable** files (that can be changed manually by users and also by automated scripts or services)
  * It can be **shareable** between computers or **unshareable** (for example, directories than contain system-specific information).


Thus, each directory can be classified in a 2x2 matrix. Here is an example of some directories in a FHS-compliant system:
<table frame="hsides" border="1" class="CALSTABLE" >


<tr >

**shareable**
**unshareable**
</tr>

<tbody >
<tr >

<td >**static**
</td>

<td >/usr
</td>

<td >/etc
</td>
</tr>
<tr >

<td >
</td>

<td >/opt
</td>

<td >/boot
</td>
</tr>
<tr >

<td >**variable**
</td>

<td >/var/mail
</td>

<td >/var/run
</td>
</tr>
<tr >

<td >
</td>

<td >/var/spool/news
</td>

<td >/var/lock
</td>
</tr>
</tbody>
</table>
The specification then lists the most important directories, and the kind of content that is supposed to be found there. This list, although not complete or exhaustive, can give you a taste of the specification:




  * **/ **This is the top of the tree. Every Linux filesystem should have one, and all Linux directories branch off this one. Certain other critical subdirectories, such as /etc and /sbin, should be on the same partition as /, but other can optionally be in other partitions.
  * **/bin** It contains certain critical system-wide used binary files (for example, ls, cp, mount, ...) and other low-level simple applications. It is static and normally not shared.
  * **/sbin** Similar to /bin but with programs that are normally run only by the system administrator.
  * **/lib** It contains program libraries. It is usually not shared.
  * **/etc** It contains host-specific system configuration: static files that are not binary and control the operation of a program.
  * **/usr** This directory stores the major part of the Linux computer programs. It is static and shareable. Sometimes it is stored in a separate partition so that it can be easily shared by NFS. Is has a nested structure of /usr/bin and /usr/lib similar to the one in /, but the difference is that the binaries here are not critical for the system.
  * **/usr/local** This special subdirectory of /usr stores a complete tree (/usr/local/bin, /usr/local/lib) of binaries, similar to the /usr content, but for programs that the administrators install locally, for example, packages compiled for a target computer. The idea behind this directory is that this packages are not modified during a system upgrade.
  * **/opt** Similar to /usr/local but specially tailored for third-party applications that don't ship with the base OS, like browsers, word proccessors, games, etc...
  * **/boot** This directory contains static and unshareable files related to the initial booting of the computer. Some old systems impose restrictions on this directory so that it could be necessary to store it on a separate partition
  * **/home** It holds the users' data, and it's shareable and variable. Frequently found in a separate partition.
  * **/root** The home of the root user. Not shareable.
  * **/var** This directory contains temporary and transient files like system logs, print spool files, mail files, application cache data and so on. It is variable and some of its subdirectories are shareable. It is advised to have a separate partition in the case of servers in which it should be precisely monitored.
  * **/srv** Contains site-specific data which is served by this system. In theory, the readable configuration for services and scripts that they use should reside in the tree structure branching from this directory.
  * **/tmp** Many programs need to create temporary files and this is the right place to do it.  Its contents are periodically wiped out. Sometimes stored in a separate partition.
  * **/mnt** and **/media** Preferred root folders for mounting temporary and removable-media filesystems, such as shares and floppys, cdroms, etc...
  * **/dev** Linux interface treats most hardware devices as if they were files, so this is the place where the OS stores these files.
  * **/proc** This special directory is a virtual filesystem created by Linux to provide access to certain types of hardware information that aren’t accessible via /dev, such as the cpu and memory of the system.


There are also common deviations from the FHS such as the **/sys** directory usage as a virtual filesystem similar to /proc and **/run** as a temporary filesystem. The FHS current version is [2.3](http://www.pathname.com/fhs/pub/fhs-2.3.html), the next 3.0 version is still a future draft under development.
