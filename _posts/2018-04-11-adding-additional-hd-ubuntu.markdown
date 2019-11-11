---
layout: post
title:  "Ubuntu - Adding additional hard drives"
date:   2018-04-11 15:33:55 -0600
categories: ubuntu storage
---

- Power off the virtual machine.

- Add a new disk, and power back on the virtual machine.

- Next run `fdisk -l`

- Note the new disk that has been added. For now lets assume its `/dev/sdb`

- Create a filesystem using `mkfs -t ext4 /dev/sdb`

- Next edit the `/etc/fstab` and add the following line into it
  - `/dev/sdb /storage ext4 defaults 0 0`


- Finally reboot the vm, and the disk should be attached `sudo shutdown -r now`

At this point, the new disk should be mounted onto the `/storage`
