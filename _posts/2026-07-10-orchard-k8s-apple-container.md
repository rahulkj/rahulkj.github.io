---
title: "Orchard — Kubernetes clusters on Apple Silicon with apple/container"
date: 2026-07-10T20:11:00-05:00
description: "Run multi-node Kubernetes clusters on your Mac, powered by Apple's native Virtualization.framework"
featured: true
draft: false
toc: true
categories:
  - Kubernetes
tags:
  - kubernetes
  - apple-silicon
  - containers
  - tools
---

I've been wanting a lightweight way to run Kubernetes directly on my MacBook's Apple Silicon without firing up a whole VM farm. `minikube` doesn't have an `apple/container` driver, and Apple's own `container` CLI has an unmerged k8s proposal that never shipped. [Orchard](https://github.com/rahulkj/orchard) fills that gap — it drives `kubeadm` against node VMs booted with `apple/container`, no third-party shim in the loop.

---

## What Orchard is

A CLI tool (Go) that creates multi-node Kubernetes clusters where **every node is its own lightweight Linux VM** running on Apple's native [`container`](https://github.com/apple/container) CLI (Virtualization.framework). Each node boots from a `kindest/node` image (same image `kind` uses) with systemd, containerd, kubeadm, and kubelet pre-installed.

No Virtual Kubelet, no Docker Desktop, no Lima — just `kubeadm init` and `kubeadm join` running inside native Apple VMs.

---

## Prerequisites

- macOS on Apple Silicon (M-series)
- [`apple/container`](https://github.com/apple/container) installed
- `kubectl`

```bash
container system start
container system kernel set --recommended
```

---

## Install

```bash
brew install rahulkj/tap/orchard
orchard doctor   # verify everything is ready
```

Or build from source: `make build`.

---

## Quick start

```bash
orchard create --name dev --workers 2
```

Wait ~30 seconds, then:

```bash
kubectl get nodes -o wide
container list                    # same nodes, as VMs
```

You get:
- 1 control-plane VM + 2 worker VMs
- `kindnet` CNI (default, cross-node pod connectivity verified)
- metrics-server + local-path StorageClass
- `orchard-dev` context merged into `~/.kube/config`

---

## Anatomy of a cluster

| Component | Detail |
|-----------|--------|
| Node image | `kindest/node` (pinned by digest, currently `v1.36.1`) |
| VM hypervisor | Apple `container` (Virtualization.framework) |
| Kubernetes init | `kubeadm init` via `container exec` |
| CNI | kindnet (default), flannel (broken — missing `bridge` plugin), calico (untested) |
| Default sizing | 4 CPU / 4 GB control-plane, 4 CPU / 2 GB per worker |
| State | Saved locally; kubeconfig context `orchard-<name>` |

---

## Commands

```bash
orchard create --name dev --workers 2              # create
orchard scale --name dev --workers 4                # scale up/down
orchard stop --name dev                             # stop VMs (preserve state)
orchard start --name dev                            # resume VMs
orchard delete --name dev                           # teardown
orchard list                                        # list clusters
orchard doctor                                      # health check
orchard check-updates                               # check newer k8s versions
orchard upgrade --name dev                          # recreate on new image
orchard headlamp install --name dev                 # web UI
orchard headlamp token --name dev                   # access token
orchard self-update                                 # update orchard binary
```

### Create flags

| Flag | Default | Notes |
|------|---------|-------|
| `--workers` | `2` | `0` gives single untainted node |
| `--image` | pinned digest | override for different k8s version |
| `--cp-cpus` / `--cp-memory` | `4` / `4096M` | control-plane sizing |
| `--worker-cpus` / `--worker-memory` | `4` / `2048M` | per-worker sizing |
| `--cni` | `kindnet` | kindnet, flannel, or calico |
| `--no-metrics` | off | skip metrics-server |
| `--no-storage` | off | skip local-path StorageClass |
| `--headlamp` | off | install Headlamp web UI |
| `--proxy-forward` | off | corporate proxy / Zscaler support |

---

## The tricky parts Orchard handles

### Fresh DHCP lease on every `start`

Apple container VMs get a new IP every boot. `kindest/node` images aren't designed for that — `kind create` writes a `/kind/kubeadm.conf` that the image's entrypoint expects on IP change, and it never repoints `kube-proxy` or CoreDNS at a new control-plane address. Left stale, every ClusterIP silently breaks after a restart.

`orchard start` repairs all of this:
- Regenerates the API server serving cert for the new IP
- Repoints `admin.conf`, `super-admin.conf`, and every worker's `kubelet.conf`
- Repoints the cluster-wide `kube-proxy` ConfigMap and restarts kube-proxy
- Restarts CoreDNS (its in-process client latches onto the first connection and won't recover)

### No internet egress from VMs

`apple/container` pulls images host-side. Every addon URL (metrics-server, Headlamp, CNI manifests) is fetched on the host and piped into the guest:

```bash
container exec -i <node> kubectl apply -f -
```

### `kubeadm init` version pinning

Kubeadm's default "resolve latest stable from the internet" can return a version newer than what's baked into the image, causing image pull failures. Orchard reads `/kind/version` from inside the image and pins `--kubernetes-version` to it.

---

## Scaling

```bash
orchard scale --name dev --workers 4
```

Scaling up boots and joins new workers at the lowest free indices. Scaling down drains the highest-indexed workers first (`kubectl drain` → `kubectl delete node` → VM removal), keeping lower-numbered workers stable across resizes.

---

## Corporate proxy support

Macs behind Zscaler/Netskope-style TLS-intercepting proxies need the interception root CA trusted inside the guest VMs. `--proxy-forward` does two things:

1. Exports certs from `System.keychain` (where MDM/security-agent roots land) and installs them inside the guest via `update-ca-certificates`
2. Forwards `HTTP_PROXY`/`HTTPS_PROXY` env vars into the guest, rewriting `127.0.0.1`/`localhost` to the guest's default gateway

It deliberately skips `scutil --proxy`/PAC — on managed Macs, PAC scripts point at a loopback port bound by the local security agent that a guest VM can't reach.

---

## Headlamp web UI

```bash
orchard create --name dev --headlamp
# or install into existing cluster:
orchard headlamp install --name dev
orchard headlamp token --name dev

kubectl port-forward -n kube-system service/headlamp 8080:80
```

Headlamp is a solid Kubernetes web UI. The upstream manifest references a `headlamp-admin` ServiceAccount that doesn't exist — Orchard creates it, bound to `cluster-admin` (appropriate for a local dev cluster).

---

## Upgrade

```bash
orchard upgrade --name dev
```

This is a **destroy-and-recreate**, not an in-place kubeadm upgrade. The node images are immutable single-version builds with no internet egress, so there's no way to do an in-place version bump — `kind` has the same constraint. Orchard reads the cluster's saved settings, deletes it, and recreates on the new image. Workloads are not preserved.

---

## Why not alternatives?

| Tool | Gap |
|------|-----|
| `minikube` | No `apple/container` driver (open issue, unimplemented) |
| `apple/container k8s` plugin | Experimental proposal, never merged/shipped |
| `kiac` / `macvz` / `apple-container-kubelet` | Virtual Kubelet approach, not real multi-node kubeadm |
| `kind` / `k3d` | Runs containers inside Docker, not native VMs |

Orchard gives you real `kubeadm`-driven multi-node clusters running in native Apple VMs — no Docker dependency, no indirection layer, just `container run` → `kubeadm init`.

---

If you have an Apple Silicon Mac and want to poke at a real multi-node Kubernetes cluster without spinning up a homelab or paying for cloud VMs, give Orchard a shot.

```bash
brew install rahulkj/tap/orchard
orchard create --name dev --workers 2
kubectl get nodes
```

Enjoy!
