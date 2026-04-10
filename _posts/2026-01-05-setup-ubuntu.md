---

title: "⚙️ Setup Ubuntu 25.10 to Run Ansible Scripts"
date: 2026-01-05T19:31:56-06:00
draft: false
tags:
  - devops
  - homelab
  - ci-cd
categories:
  - Homelab

---

## 🔄 Switch Sudo Into Manual Mode
```
sudo update-alternatives --set sudo /usr/bin/sudo.ws
```

## Fix the system time on the ubuntu OS
```
sudo apt-mark auto chrony && sudo apt install -y systemd-timesyncd
sudo apt install -y ntpsec-ntpdate
sudo ntpdate -u 172.16.0.22
sudo systemctl enable systemd-timesyncd
sudo systemctl start systemd-timesyncd
sudo systemctl status systemd-timesyncd
```