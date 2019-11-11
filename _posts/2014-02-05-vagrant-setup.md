---
layout: post
title:  "Vagrantup - Multiple machine setup"
date:   2014-02-05 14:57:00 -0600
categories: vagrant setup
---

Time to spin up multiple virtual machines with use of open source.

Download and Install Vagrant from http://www.vagrantup.com/
Download and Install VirtualBox from https://www.virtualbox.org/

Now that we have required tools, you can choose the virtual box that you would like to spin up.
The list is available at http://www.vagrantbox.es/

Now lets setup a few boxes ubuntu machines: using your terminal (UNIX/Mac OS), fire the following commands:

vagrant box add ubuntu http://files.vagrantup.com/precise64.box
vagrant init ubuntu

At this point, you can find a file with the name `"Vagrantfile"` created in your home directory.
Edit this file and update the configuration to look like this:

```
config.vm.define "mc1" do |mc1|
mc1.vm.network "private_network", ip: "10.10.50.70"
mc1.vm.box = "ubuntu"
end

config.vm.define "mc2" do |mc2|
mc2.vm.network "private_network", ip: "10.10.50.71"
mc2.vm.box = "ubuntu"
end
```

What we did in this file is: we specified 2 machines mc1, mc2 which will use the ubuntu box as their OS.
Next we specified the Network configuration to be `hostonly` and specified a static IP. This way if the machine is restarted, we don't get a different ip.

If you want to use an IP assigned by the network, instead of `hostonly` use `bridged` and remove the static IP address:
for ex: mc2.vm.network :bridged

Once you are done with this, fire the following commands:

```
vagrant up mc1
vagrant up mc2
```

Once the machines are up, you can ssh into the machines using the commands:

```
vagrant ssh mc1
vagrant ssh mc2
```

Once in the box, don't forget to update the `/etc/hosts` and `/etc/hostname` files.

Now you have 2 ubuntu VM's up and running. All set for clustering?
