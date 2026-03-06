---
title: "Install UISP on Ubuntu 25.10"
date: 2026-03-06T16:46:00-06:00
draft: false
tags:
  - ubuntu
  - uisp
---

## Objective
The goal of this article is to go over the setup of uisp on Raspberry Pi 4, that is running ubuntu 25.10 server.

The process is simple, but needs some minor tweaks.

So let's begin!

## Setting up sudo access
With 25.10, you will need to change the way you authenticate using `sudo` in 25.10

> sudo update-alternatives --set sudo /usr/bin/sudo.ws

## Next ensure your time is in sync. If you have an NTP server at home, then use that for the `NTP_SERVER` variable

```
NTP_SERVER=pool.ntp.org

sudo apt-mark auto chrony && sudo apt install -y systemd-timesyncd
sudo apt install -y ntpsec-ntpdate
sudo ntpdate -u $NTP_SERVER
sudo systemctl enable systemd-timesyncd
sudo systemctl start systemd-timesyncd
sudo systemctl status systemd-timesyncd
```

## Let's install docker

Since uisp needs docker installed, its better to do this beforehand. To do this follow the instructions on the [docker website](https://docs.docker.com/engine/install/ubuntu/)

OR run the following commands

```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

## Installing UISP

* This is a bit of an involved process. First download the script from the [uisp site](https://help.uisp.com/hc/en-us/articles/22591008678039-UISP-First-Time-Setup-Installation)

    > curl -fsSL https://uisp.ui.com/install > /tmp/uisp_inst.sh && sudo bash /tmp/uisp_inst.sh

* Next terminate the script

    > vi /tmp/unms-install/install-full.sh

* Replace all occurances of `$(LC_CTYPE=C tr -dc "a-zA-Z0-9" < /dev/urandom | fold -w 48 | head -n 1 || true)` by `$(openssl rand -base64 36)`

* Replace all occurances of `$(LC_CTYPE=C tr -dc "a-zA-Z0-9" < /dev/urandom | fold -w 100 | head -n 1 || true)` by `$(openssl rand -base64 36)`

* Finally run the installation

    > sudo /tmp/unms-install/install-full.sh

If all goes well with your network connection, all the images will be pulled, containers will be started, and you will be able to access the site, using `https://<your-pi-host>/nms/dashboard`

## Conclusion
This setup could have worked without any tweaks, if the password generation logic bundled in the install script did not use `urandom`. 

Anyway I hope this helped!