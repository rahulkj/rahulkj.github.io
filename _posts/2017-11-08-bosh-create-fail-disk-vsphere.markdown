---
layout: post
title:  "'bosh create' fails due to disk format issue on vSphere"
date:   2017-11-08 17:31:00 -0600
categories: bosh error vSphere
---

This is one crazy issue where I saw that bosh create on `vSphere` resulted in a stemcell creation, but when the bosh director vm was being created, I saw the error the bosh director vm that looked like

> Errors were found while checking the disk drive for /tmp. Press F to attempt to fix the errors, I to ignore, S to skip mounting or M for manual recovery

To debug this, we updated the MTU settings on the vlan adapters to the right MTU settings on both network adapters and storage network.

Then we tried to create a VM using a Ubuntu image on one of the hosts, and we saw that, when the VM was starting, its time was really out of sync, and on investigating it more, the ESXi hosts had no NTP server defined and they were set to a date that was way off than current time. After setting the NTP server settings for each host, the bosh create ran fine... Woaaah!!!!

So the issue was the NFS time and the ESXi host time were not in sync, and that was causing disk provisioning errors, and hence the bosh director vm had disk issues.

Hope this helps more people who are in this similar state while spinning an OSS Bosh
