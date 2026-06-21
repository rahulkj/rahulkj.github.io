---
title: "Avoid Network Lockout on Raspberry Pi Kubernetes Clusters"
date: 2026-06-20T23:17:32-05:00
description: "How to Avoid Network Lockout on Raspberry Pi Kubernetes Clusters"
featured: true
draft: false
toc: true
categories:
  - Technology
tags:
  - Kubernetes
  - Raspberry Pi 5
  - Virtualization
  - Homelab
---

# How to Avoid Network Lockout on Raspberry Pi Kubernetes Clusters

Building a Kubernetes cluster on Raspberry Pis is a rite of passage for homelab enthusiasts. But if you've ever watched your Pi cluster go completely dark — no SSH, no kube-API, nothing — hours after a "successful" setup, you've hit the network lockout problem. It's not one bug; it's six, and they all look the same: unreachable nodes.

Here's how to fix each one, drawn from real production playbooks.

## 1. Pin Your IPs Before Anything Else

The single most common cause of lockout: **DHCP lease renewal**.

Every Kubernetes component — kubelet, etcd, kubelet — binds to the IP address it sees at init time. When DHCP hands out a new address, etcd sockets tear down, the control plane collapses, and SSH stops responding. The fix is brutally simple: capture the current IP from `ansible_facts['default_ipv4']['address']` and write a netplan file with `dhcp4: false`.

```yaml
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 172.16.0.10/24
      routes:
        - to: default
          via: 172.16.0.1
```

Then disable cloud-init network regeneration so it can't overwrite your file on reboot. Apply with `netplan apply` — because the IP hasn't changed, systemd-networkd promotes the lease to permanent without flapping the interface.

**Key insight**: Run this _before_ kubeadm or K3s touches anything. If you pin after init, the cluster is already using the old lease.

## 2. Neutralize the Dracut DHCP Fallback

Raspberry Pi's initramfs drops a file at `/run/systemd/network/zzzz-dracut-default.network` that sets `DHCP=yes` on every non-loopback interface. Normally the netplan-generated file sorts earlier alphabetically and wins. But if that netplan file is ever delayed or absent — say, during a boot after an upgrade — dracut silently reconfigures eth0 with DHCP, blowing away your static config.

The fix: create an `/etc/systemd/network/zzzz-dracut-default.network` override. systemd-networkd gives `/etc/` precedence over `/run/`, so your override wins regardless of boot order. Match it against a non-existent interface name so it never configures anything:

```ini
[Match]
Name=__no-such-interface__
```

## 3. Install the Right Kernel Packages for the Onboard NIC

This one bites silently. The Raspberry Pi 5's onboard NIC driver ships in `linux-modules-extra-raspi` and the RP1 firmware in `linux-firmware-raspi`. These are **tracking metapackages** — they always match the installed kernel version. They are **not** installed by default on Ubuntu Server.

A plain `apt dist-upgrade` bumps the kernel without the matching modules. On the next reboot, the Pi loads a kernel that can't drive its own network card. No SSH. No kube-API. Dead node.

The fix is one line in your provisioning:

```bash
apt install -y linux-modules-extra-raspi linux-firmware-raspi
```

Use the metapackages, not the versioned ones. Pinning to the running kernel version goes stale the moment a newer kernel is installed.

## 4. Validate Your MetalLB Pool (Pre-Flight)

MetalLB in L2 mode answers ARP for every address in its pool. If your `IP_POOL` range includes a node's IP or — worse — the default gateway, MetalLB hijacks that address at layer 2 and black-holes all traffic to it. The entire subnet goes offline.

This is a pre-flight check, not a recovery. Before you deploy anything, iterate over every node and verify that neither the node IP nor the gateway falls within the MetalLB pool:

```python
_pool_lo <= _node_int <= _pool_hi  # FAIL
_pool_lo <= _gw_int <= _pool_hi    # FAIL
```

This simple integer-range comparison turns a silent multi-hour outage into a clear error before the cluster even starts.

## 5. Fix the AppArmor Containerd Profile

The default `cri-containerd.apparmor.d` profile runs in enforce mode. This blocks runc from sending SIGTERM and SIGKILL to container init processes. The symptom: pods stuck in Terminating forever. Zombie containers accumulate. kubelet runs out of resources, and the node goes dark.

Switch the profile to complain mode:

```bash
apparmor_parser -r /etc/apparmor.d/cri-containerd.apparmor.d
```

This allows signal delivery while still logging violations for audit. Apply it _before_ the reboot that follows container installation, so kubelet never runs under the restricted profile.

## 6. Disable the Broken IPVS Watchdog

Some Kubernetes setups install `ipvs-watchdog.timer`, a systemd timer that runs every 5 seconds to execute `/usr/local/sbin/ipvs-watchdog.sh`. The problem: the script is never installed. Every 5 seconds, the timer fails silently. Worse, if kube-proxy briefly enters IPVS mode during a restart, the `KUBE-IPVS-FILTER` chain can DROP **all** traffic — including SSH and the kube API.

Disable and remove it:

```bash
systemctl stop ipvs-watchdog.timer
systemctl disable ipvs-watchdog.timer
rm /etc/systemd/system/ipvs-watchdog.* /usr/local/sbin/ipvs-watchdog.sh
```

## 7. Deploy a NIC TX Hang Watchdog (Pi 5 Only)

The Raspberry Pi 5's macb/bcm Ethernet controller has a hardware bug: the TX ring can silently stop transmitting. No kernel message, no log entry, no watchdog timeout. The node simply stops talking.

Detection logic: if `/sys/class/net/eth0/statistics/tx_bytes` stops advancing for 60 seconds **and** the default gateway is unreachable, assume a TX hang. Recovery is three-tier:

1. `ip link set eth0 down/up` — resets TX queues (brief outage, usually works)
2. `systemctl restart systemd-networkd` — full netplan reapply
3. `systemctl reboot` — last resort

The dual condition (TX bytes stalled **and** gateway ping fails) avoids false positives on genuinely idle links.

## The Complete Playbook Order Matters

These fixes are not independent. The order in the provisioning playbook is deliberate:

```
1. Validate MetalLB pool        # fail fast before any damage
2. Sync time                    # TLS certs need accurate clocks
3. Pin static IPs                # before anything binds to a lease
4. Fix dracut DHCP fallback      # before first reboot
5. Install kernel metapackages   # before apt upgrade + reboot
6. Reboot                        # now safe to boot the new kernel
7. Install containerd + K8s      # creates AppArmor profile
8. Fix AppArmor profile          # before kubelet starts
9. Disable IPVS watchdog         # before kube-proxy starts
10. Deploy TX watchdog           # after eth0 has settled
```

Every step exists because a real outage hit a real cluster. Skipping any one of them guarantees you'll eventually SSH into a node that won't answer.
