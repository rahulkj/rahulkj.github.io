---
layout: post
title: "Ansible playbook for managing k8s ops"
date: 2025-02-05 12:40:00 +0530
categories: ansible, kubernetes, raspberry-pi, ubuntu
---

If you are looking to bootstrap or destroy the k8s clusters that are provisioned on you raspberry Pi's or vms using kubeadm, then you are in the right place. This blog will guide you on how to install the full blown k8s components, and not k3s or microk8s.

### Pre-requsities
* Install all the required cli's on your mac
  * ansible
  * direnv
* Clone the [repo](https://github.com/rahulkj/ansible-k8s-operations) `https://github.com/rahulkj/ansible-k8s-operations`
* Update the [.envrc](https://github.com/rahulkj/ansible-k8s-operations/.envrc) file with your hosts names, username and passwords
* Generate the inventory yaml using the script `https://github.com/rahulkj/ansible-k8s-operations/generate-k8s-inventory.sh`, that will generate the inventory files in the [config](https://github.com/rahulkj/ansible-k8s-operations/config/) folder
* Validate and update the inventory [yamls](https://github.com/rahulkj/ansible-k8s-operations/config) if setting up on vms or k8s

### Bootstrap k8s on pi/vms
To do this, its simple, just run the script `https://github.com/rahulkj/ansible-k8s-operations/run-anisble.sh` with the appropriate options

```
./run-ansible.sh k8s-bootstrap
Usage: ./run-ansible.sh k8s-bootstrap <OPTION>
pi: bootstrap k8s environment on pi
vms: bootstrap k8s environment on vms
```

Once done, you can ssh into the control_plane vm/s, and run the following commands:

```
ok: [k8s-node-1] => {
    "msg": [
        "SSH to the host, and run the commands",
        "mkdir -p $HOME/.kube",
        "sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config",
        "sudo chown $(id -u):$(id -g) $HOME/.kube/config"
    ]
}
```

The following are provisioned:
* Kubernetes cluster
  * CNI - canal
  * containerd
* metallb loadbalancer that uses the IP Pool's from your .envrc
* kubernetes dashboard
* nfs storage provider using your nfs configuration from your .envrc
* nginx ingress controller

### Destroy k8s on pi/vms
Its similar to bootstrapping, but the command and options are different

```
./run-ansible.sh k8s-destroy
Usage: ./run-ansible.sh k8s-destroy <OPTION>
pi: destroy k8s environment on pi
vms: destroy k8s environment on vms
```

Enjoy!