---
title: "Setting up k8s on ubuntu 25.10 on Raspberry Pi 5"
date: 2026-01-27T11:16:00-06:00
draft: false
tags:
  - kubernetes
  - traefik
  - headlamp
  - ubuntu
---

## Installation of the OS
* Flash the NVME m2 SSD using [Raspberry Pi Imager](https://www.raspberrypi.com/software/), and install Ubuntu 25.10 OS on it.
* Ensure you specify the hostname, username, password, etc to reduce the steps post flashing the drive.
* Startup your Raspberry Pi, and configure the network and assign a static ip

  > sudo vi /etc/netplan/50-cloud-init.yaml

  ```
  network:
    version: 2
    ethernets:
      eth0:
        dhcp4: false
        addresses:
        - 192.168.0.xx/23
        nameservers:
          addresses:
          - 192.168.0.1
          search:
          - example.local
        routes:
        - to: default
          via: 192.168.0.1
        optional: true
  ```

  > sudo netplan apply

> **NOTE: I found these manual steps so error prone, that I created an ansible github repo to help with these.**

> **Checkout [Ansible k8s Operations](https://github.com/rahulkj/ansible-k8s-operations) github repo and hopefully this makes it easy for you to setup k8s quickly.**

## Prepping the OS for installation


* Disable ipv6, I ran into issues with the network just shutting off after certain time
    
  > sudo vi /etc/sysctl.d/99-disable-ipv6.conf

  ```
  net.ipv6.conf.all.disable_ipv6 = 1
  net.ipv6.conf.default.disable_ipv6 = 1
  net.ipv6.conf.lo.disable_ipv6 = 1
  ```

  > sudo sysctl --system

* Disable swap

  ```
  sudo swapoff -a
  sudo sed -i '/swap/ s/^\(.*\)$/#\1/g' /etc/fstab
  ```

* Enable cgroups
  ```
  sudo sed -i 's/$/ cgroup_enable=memory cgroup_memory=1/' /boot/firmware/current/cmdline.txt
  ```

## Install relevant packages on each of the Raspberry Pi's

* Update the apt repository and apply the updates
  ```
  sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
  ```

* Containerd configuration

  > sudo vi /etc/modules-load.d/containerd.conf

  ```
  overlay
  br_netfilter
  ```

* Modprobe

  ```
  sudo modprobe overlay
  sudo modprobe br_netfilter
  ```

* Add conf for containerd

  > sudo vi /etc/sysctl.d/99-kubernetes-cri.conf

  ```
  net.bridge.bridge-nf-call-iptables = 1
  net.ipv4.ip_forward = 1
  net.bridge.bridge-nf-call-ip6tables = 1
  ```

* Apply the system settings
  > sudo sysctl --system

* Install containerd network plugins and configure cgroups

  ```
  sudo apt-get update && sudo apt-get install -y containerd containernetworking-plugins
  
  sudo mkdir -p /etc/containerd
  
  containerd config default | sed "s/ShimCgroup = ''/ShimCgroup = ''\n            SystemdCgroup = true/" | 
  sudo tee /etc/containerd/config.toml
  
  sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

  if grep -q '^\[plugins."io.containerd.grpc.v1.cri".registry\]' /etc/containerd/config.toml; then
      # Insert Harbor config after the registry section
      sudo sed -i '/^\  [plugins."io.containerd.grpc.v1.cri".registry\]/a\
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]\
          [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.lab.int"]\
          endpoint = ["https://harbor.lab.int"]\
      [plugins."io.containerd.grpc.v1.cri".registry.configs]\
          [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.lab.int".tls]\
          insecure_skip_verify = true\
  ' /etc/containerd/config.toml
  else
      # Insert a new registry block after the [plugins] header
      sudo sed -i '/^\[plugins\]/a\
      [plugins."io.containerd.grpc.v1.cri".registry]\
      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]\
          [plugins."io.containerd.grpc.v1.cri".registry.mirrors."harbor.lab.int"]\
          endpoint = ["https://harbor.lab.int"]\
      [plugins."io.containerd.grpc.v1.cri".registry.configs]\
          [plugins."io.containerd.grpc.v1.cri".registry.configs."harbor.lab.int".tls]\
          insecure_skip_verify = true\
  ' /etc/containerd/config.toml
  fi

  sudo systemctl restart containerd
  ```

* Add conf for crictl

  > sudo vi /etc/crictl.yaml

  ```
  runtime-endpoint: unix:///run/containerd/containerd.sock
  image-endpoint: unix:///run/containerd/containerd.sock
  timeout: 2
  debug: true
  pull-image-on-create: false   
  ```

* Install and configure k8s dependencies

  ```
  sudo apt-get update && sudo apt-get install -y apt-transport-https curl ipvsadm
  
  curl -fsSL https://pkgs.k8s.io/core:/stable:/v{{k8s.version}}/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  
  sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
  ```

  ```
  sudo vi /etc/apt/sources.list.d/kubernetes.list

  deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v{{k8s.version}}/deb/ /
  ```

* Install the kubernetes cli;s

  ```
  sudo apt-get update
  sudo apt-mark unhold kubelet kubeadm kubectl
  sudo apt-get purge -y kubelet kubeadm kubectl
  sudo apt-get install -y kubelet kubeadm kubectl
  sudo apt-mark hold kubelet kubeadm kubectl    
  ```

* If planning to use nfs, install the package

  ```
  sudo apt-get update
  sudo apt-get install -y nfs-common    
  ```

* Reboot all the nodes

  > sudo reboot

## Initialize the control plane
* SSH to the machine which will be the first control plane node

* Generate the token, gather the local IP and initialize the cluster

  ```
  TOKEN=$(kubeadm token generate)

  IP_ADDR=$(hostname -I | awk '{print $1}')

  kubeadm init --pod-network-cidr=10.244.0.0/16 --token $TOKEN --control-plane-endpoint "$IP_ADDR:6443" --upload-certs
  ```

* Install the Pod Networking, I'm going with calico
    
  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf

  VERSION=$(curl https://api.github.com/repos/projectcalico/calico/releases | jq -r '.[0] | .tag_name')
      
  curl https://raw.githubusercontent.com/projectcalico/calico/$VERSION/manifests/canal.yaml -O
      kubectl apply -f canal.yaml
      rm canal.yaml
  ```

* Next lets get the join command to add worker nodes

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  kubeadm token create --print-join-command
  ```

## Join the worker nodes to the control plane

* SSH into the worker nodes one by one, and run the join command that you captured in the previous step

## Install the required packages to make the kubernetes cluster functional

Let's use the control plane node to run all these commands to install all the required packages. 

* SSH into the control plane node

### Load Balancer - METAL LB
* Let's install the load balancer. I prefer metallb as this is native and uses a set of IP's that we supply to create LoadBalancer endpoints for the services

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf

  kubectl get configmap kube-proxy -n kube-system -o yaml | \
  sed -e "s/strictARP: false/strictARP: true/" | \
  kubectl apply -f - -n kube-system

  VERSION=$(curl https://api.github.com/repos/metallb/metallb/releases | jq -r '.[0] | .tag_name')

  wget https://raw.githubusercontent.com/metallb/metallb/$VERSION/config/manifests/metallb-native.yaml -O metallb-native-$VERSION.yaml

  kubectl apply -f metallb-native-$VERSION.yaml

  kubectl -n metallb-system rollout status deployment/controller --watch=true

  rm metallb-native-$VERSION.yaml    
  ```

* Next create the following metal lb config file. Make sure you update the `$IP_POOL` with a range of IPs, like `192.168.0.50-192.168.0.60`. 

  > vi /tmp/metal-lb-config.yaml

  ```
  apiVersion: metallb.io/v1beta1
  kind: IPAddressPool
  metadata:
    name: pool
    namespace: metallb-system
  spec:
    addresses:
      - $IP_POOL
    autoAssign: true
  ---
  apiVersion: metallb.io/v1beta1
  kind: L2Advertisement
  metadata:
    name: l2-advertisement
    namespace: metallb-system
  spec:
    ipAddressPools:
      - pool
  ```

* Apply this config
  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  kubectl apply -f /tmp/metal-lb-config.yaml
  ```

### OPTIONAL - NFS Storage

I like having a storage class available for the cluster, and to enable this, I use the NFS storage class. To install this do the following

* Tweak the values for `$NFS_SERVER` and `$NFS_MOUNT`

  ```
  snap install helm --classic

  export KUBECONFIG=/etc/kubernetes/admin.conf
          
  helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner

  helm upgrade --install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
      --create-namespace \
      --namespace nfs-provisioner \
      --set nfs.server=$NFS_SERVER \
      --set nfs.path=$NFS_MOUNT \
      --set storageClass.name=nfs-storage \
      --set storageClass.defaultClass=true
  ```

### Install Ingress using  Traefik
Since nginx has been deprecated, I chose traefik, as its the next best option to expose the web endpoints, and it supports Gateway's too.

* To install using helm, we will first need the `traefik-values.yaml`. This configuration exposes the services on port 80, but if you need 443, then tweak this config

  > vi /tmp/traefik-values.yaml

  ```
  deployment:
    replicas: 1

  ingressClass:
    enabled: true
    isDefaultClass: true
    name: traefik

  providers:
    kubernetesIngress:
      enabled: true
    kubernetesCRD:
      enabled: true

  # ONLY HTTP ENTRYPOINT
  ports:
    web:
      port: 80
      exposedPort: 80
      protocol: TCP

  # IMPORTANT: do NOT define websecure at all

  service:
    type: LoadBalancer

  # No TLS, no ACME, no redirects
  additionalArguments:
    - "--log.level=INFO"

  experimental:
    http3:
      enabled: false

  ```

* Using helm install the package

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  helm upgrade --install traefik traefik/traefik \
      --create-namespace --namespace traefik \
      -f /tmp/traefik-values.yaml    
  ```

### Install Headlamp for viewing the kubernetes dashboard

Recently k8s dashboard has been depreacted due to lack of contributions, and the next best suggestion is Headlamp. So lets install this

* Apply the latest config 

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  
  kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/headlamp/main/kubernetes-headlamp.yaml
  ```

* Create a `headlamp-ingress.yaml` file with the following contents. Ensure you specify the correct value for the `Host`
    > vi /tmp/headlamp-ingress.yaml

  ```
  ---
  apiVersion: traefik.io/v1alpha1
  kind: IngressRoute
  metadata:
    name: headlamp
    namespace: kube-system
  spec:
    entryPoints:
      - web
    routes:
    - match: Host(`headlamp.example.local`)
      kind: Rule
      services:
      - name: headlamp
        port: 80
  ```

* Apply this to expose the headlamp using the IngressRoute via traefik

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  kubectl apply -f /tmp/headlamp-ingress.yaml
  ```

* Create service account user and assign permissions for accessing headlamp dashboard

  ```
  export KUBECONFIG=/etc/kubernetes/admin.conf
  
  kubectl -n kube-system create serviceaccount headlamp-admin
  
  kubectl create clusterrolebinding headlamp-admin \
      --serviceaccount=kube-system:headlamp-admin \
      --clusterrole=cluster-admin    
  ```

## Conclusion

And just like that you created your awesome functional kubernetes cluster.