---
layout: post
title:  "Assigining a static IP and allowing password based Auth on Ubuntu 23.10"
date:   2023-10-23 10:27:00 +0530
categories: ubuntu, ssh, network
---

Recently, I installed ubuntu 23.10 on my Raspberry Pi 4, and I realized there were a few changes that were done with netplan configurations. 

Also I was unable to ssh using password to any of the Pi's.

So it made me think to blog this and keep it handy for myself and others like me who are in the same boat.

## Setting up network after the first boot

When you connect to your Pi after the first boot, and login with the default username / password - `ubuntu / ubuntu`, you are prompted to change the login password.

* On competing this step, validate the contents of your network. If your network supports DHCP, you will see something like this in `sudo cat /etc/netplan/50-cloud-init.yaml`

```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: true
            optional: true
    version: 2
```

* Before we change any settings in there, you first want to edit `sudo vi /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg` and add the line into this file. By doing this, we are disabling the cloud-init's configuration (if you are using it)

```
network: {config: disabled}
```

* Next, let us configure the Pi to have a static IP. So let us edit the file `sudo cat /etc/netplan/50-cloud-init.yaml` and add the values for
- addressess - IP address with the subnet range
- via - gateway IP goes in here
- nameservers - DNS server
- search - domains
- dhcp4 - set to **false**
- optional - set to **false**

```
# This file is generated from information provided by the datasource.  Changes
# to it will not persist across an instance reboot.  To disable cloud-init's
# network configuration capabilities, write a file
# /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg with the following:
# network: {config: disabled}
network:
    ethernets:
        eth0:
            dhcp4: false
            addresses: [10.0.10.16/19]
            routes:
            - to: default
              via: 10.0.10.1
            nameservers:
              addresses:
              - 10.0.10.1
              search:
              - mylab.local
            optional: false
    version: 2
```

* Finally, run `sudo netplan apply`, and then try pinging your Pi on the IP address that you configured it with

## Enabling password based ssh

If you inspect the `sudo cat /etc/ssh/sshd_config.d/50-cloud-init.conf`, you will notice that the PasswordAuthentication is disabled. This setting will prevent anyone to ssh onto the Pi using username / password.

```
PasswordAuthentication no
```

The security guidelines dictate that you should use key based authentication to ssh into your Ubuntu OS. 

I wanted to allow both, as this is for my non-routable internal lab only.

To do this, simply modify the file `sudo vi /etc/ssh/sshd_config.d/50-cloud-init.conf` and change the value from **no** to **yes**

```
PasswordAuthentication yes
```

Finally, restart the ssh service, and now you can ssh to the machine using username / password combo!

```
sudo systemctl restart ssh
```