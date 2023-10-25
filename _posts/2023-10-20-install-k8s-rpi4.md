---
layout: post
title:  "Part 2 - Install kubernetes on Raspberry Pi 4 - 8GB"
date:   2023-10-20 10:23:00 +0530
categories: kubernetes, raspberry pi, flannel, containerd
---

## Disable swap memory on all Raspberry pi's
* Begin with disabling swap memory
```
sudo swapoff -a
```

* Verify swap is disabled, run `free -m`, and validate the values for Swap
```
               total        used        free      shared  buff/cache   available
Mem:            7809         942        5566           4        1567        6866
Swap:              0           0           0
```

* Now lets save the cgroups and swap setting in our bootup scripts

```
echo " cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1" | sudo tee -a  /boot/cmdline.txt
```

```
echo " console=serial0,115200 console=tty1 root=PARTUUID=58b06195-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory swapaccount=1" | sudo tee -a /boot/firmware/cmdline.txt
```

* Repeat this on all the nodes

## Installing containerd and network plugins on all Raspberry pi's

* To begin with, we will need to install the required packaged for container runtime, networking and vxlan for flannel to work. To do this, run the following command:
```
sudo apt update &&  sudo apt install -y containerd containernetworking-plugins linux-modules-extra-raspi
```

* Once done, lets configure containerd to leverage cgroups. Do this, lets generate a default config
```
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

* Now lets tweak the configuration to use cgroups
```
sudo vi /etc/containerd/config.toml
```

* Change `SystemdCgroup = false` to `SystemdCgroup = true` under `plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options`

* Repeat this on all the nodes

## Allow Iptables to see bridged traffic on all Raspberry pi's
According to the documentation, Kubernetes needs iptables to be configured to see bridged network traffic.

```
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF
```

* Enable net.bridge.bridge-nf-call-iptables and -iptables6

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf 
net.bridge.bridge-nf-call-ip6tables = 1 
net.bridge.bridge-nf-call-iptables = 1 
EOF
```

* Lets validate the configurations
```
sudo sysctl --system
```

* Repeat this on all the nodes

## Install kubernetes cli's on all Raspberry pi's

* Lets install the required cli's to setup kubernetes
```
sudo apt install -y kubelet kubeadm kubectl
```

* Lets pin these versions, so you can control the version updates when needed
```
sudo apt-mark hold kubelet kubeadm kubectl
```

* Repeat this on all the nodes


## Setting up the control-plane node

Let us begin with bootstrapping the control-plane (master) node. 

We will be using **flannel** for our CNI, and hence using the pod-network-cidr as `10.244.0.0/16`

Replace `control-plane-endpoint` with the IP of your master node Pi's IP

```
TOKEN=$(sudo kubeadm token generate)
sudo kubeadm init --token=${TOKEN} --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint "<CONTROL-PLANE-IP>:6443" --upload-certs
```

Once this completes you will see the output similar to the following, we will run these on the Pi's to set them as worker nodes.
Copy this value into another terminal, as we will need this later

```
kubeadm join <CONTROL-PLANE-IP>:6443 --token <TOKEN> \
	--discovery-token-ca-cert-hash sha256:<token-cert-sha>
```

### Install the CNI
* We will need the CNI installed, and for this setup, we will setup flannel

```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```

* Note once this is done, check the pods and their status, they should all be in `Running` state, especially `coredns` pod

* If the flannel pod is crashing, its likely that you missed to enable VXLAN or installing the `containernetworking-plugins` plugins. Please fix this, and check the status again.

## Joining other nodes to the control-plane node
* This is the easy part, you need to run the command

```
kubeadm join <CONTROL-PLANE-IP>:6443 --token <TOKEN> \
	--discovery-token-ca-cert-hash sha256:<token-cert-sha>
```

One this is executed, you can very your cluster status by running 

```
kubectl get nodes -o wide
```

```
NAME   STATUS   ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE       KERNEL-VERSION     CONTAINER-RUNTIME
pi-1   Ready    control-plane   39h   v1.28.2   10.0.10.16   <none>        Ubuntu 23.04   6.2.0-1014-raspi   containerd://1.7.2
pi-2   Ready    <none>          39h   v1.28.2   10.0.10.17   <none>        Ubuntu 23.04   6.2.0-1014-raspi   containerd://1.7.2
pi-3   Ready    <none>          39h   v1.28.2   10.0.10.18   <none>        Ubuntu 23.04   6.2.0-1014-raspi   containerd://1.7.2
pi-4   Ready    <none>          39h   v1.28.2   10.0.10.19   <none>        Ubuntu 23.04   6.2.0-1014-raspi   containerd://1.7.2
```

## Deploy nginx ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.3/deploy/static/provider/baremetal/deploy.yaml
```

* We need to change the service from ClusterIP to NodePort for our service, so update the service type by running
```
kubectl edit svc -n ingress-nginx ingress-nginx-controller
```

* Finally validate the service, and it should show as `NodePort`
```
kubectl describe svc -n ingress-nginx ingress-nginx-controller
```

```
NAME                                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             NodePort    10.98.225.179   <none>        80:30230/TCP,443:32484/TCP   2m49s
ingress-nginx-controller-admission   ClusterIP   10.110.94.219   <none>        443/TCP                      2m49s
```

## Now lets deploy the kubernetes dashboard

Follow the documentation [here](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

* Deploying the dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

* Change the service type from ClusterIP to NodePort
```
kubectl -n kubernetes-dashboard edit service kubernetes-dashboard
```

```
NAME                   TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
kubernetes-dashboard   NodePort   10.111.90.128   <none>        443:32438/TCP   18h
```

Access the Web UI, and you will need to generate a token to login to the dashboard. Follow the instructions [here](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md) to create a user and obtain a token.

## Creating a Service Account

We are creating Service Account with the name `admin-user` in namespace `kubernetes-dashboard` first.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

## Creating a ClusterRoleBinding

In most cases after provisioning the cluster using `kops`, `kubeadm` or any other popular tool, the `ClusterRole` `cluster-admin` already exists in the cluster. We can use it and create only a `ClusterRoleBinding` for our `ServiceAccount`.
If it does not exist then you need to create this role first and grant required privileges manually.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

## Getting a long-lived Bearer Token for ServiceAccount 

We can also create a token with the secret which bound the service account and the token will be saved in the Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
  annotations:
    kubernetes.io/service-account.name: "admin-user"   
type: kubernetes.io/service-account-token  
```

After Secret is created, we can execute the following command to get the token which saved in the Secret:

```shell
kubectl get secret admin-user -n kubernetes-dashboard -o jsonpath={".data.token"} | base64 -d
```
## Exposing the kubernetes dashboard via ingress

Now that you have verified the kubernetes dashboard, lets expose this using the ingress controller

* Create the yaml file as follows:

```
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kubernetes-dashboard-ingress
  namespace: kubernetes-dashboard
  annotations:
    nginx.org/ssl-services: kubernetes-dashboard
    nginx.ingress.kubernetes.io/ssl-passthrough: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
      - kubernetes-dashboard.k8s.int
      secretName: kubernetes-dashboard-certs
  rules:
  - host: kubernetes-dashboard.k8s.int
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: kubernetes-dashboard
            port: 
              number: 443
```

Note, the `tls` setting here exposes the service on port `443`, also we are doing passthrough to the underlying service, so the annotations are important here.

* Also create a DNS record for `*.k8s.int` that maps to one of your kubernetes nodes IP.

Finally, you should be able to access `https://kubernetes-dashboard.k8s.int:<NODE PORT>` from your browser

If you are reading this, they yay, you managed to setup your k8s cluster on your Raspberry Pi's successfully.

Enjoy!

