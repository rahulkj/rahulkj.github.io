---
layout: post
title: "Kubernetes Addons List"
date: 2024-07-7 17:10:00 +0530
categories: kubernetes, rpi-4
---

Recently I was setting up kubernetes on my RaspberryPi 4's + SSD storage and ran into strange issues, related to the storage going unresponsive after say 12 hours. It needed constant reboot. I purchased the PoE hat to power up my Pi's over PoE, and that didn't help either.

Finally, I looked at the back of my Pi's to realize all this while I was connecting my SSD on USB 2.0 and not USB 3.0. I switched the connections to my SSD over to USB 3.0 and since then I have had no issues with Pi's and k8s.

### Must try kubernetes addons
Along the lines of managing k8s on my Pi's, I ran a search on [reddit](https://www.reddit.com/r/kubernetes/comments/1849egq/must_have_kubernetes_cluster_addons/) (yes Reddit, and not ChatGPT) and some amazing folks posted these addon's. Here is the list:

- [Botkube](https://botkube.io/) - A monitoring add-on that sends alerts directly to chat messaging platforms with error metadata attached. Compatible with popular clients such as Discord or Slack.
- [Cert Manager](https://artifacthub.io/packages/helm/cert-manager/cert-manager) - Certificates
- [Chaos Mesh](https://chaos-mesh.org/) - Simulate hardware, network and other kind of failures, to check the robustness of your deployments.
- [Descheduler](https://github.com/kubernetes-sigs/descheduler) - Monitors if workloads are evenly distributed through nodes and cleans failed pods that remained as orphans/stuck. 
- [Eraser](https://github.com/azure/eraser) - A daemonset responsible for cleaning up outdated images stored in the cluster nodes. 
- [Falco](https://falco.org/) - A runtime controller that looks for unusual activity within the cluster and alerts of possible security threats. 
- [k8s-image-swapper](https://github.com/estahn/k8s-image-swapper) - Mirror images into your own registry and swap image references automatically. 
- [Kube-fledged](https://github.com/senthilrch/kube-fledged) - Allows for image caching on every node in the cluster, in order to speed up deployments. This can be used with Eraser, to define a few images that cannot be cleaned from the cluster. 
- [Kured](https://github.com/kubereboot/charts/tree/main/charts/kured) - All the cluster's nodes will be properly drained before rebooting cordoned back once they're online. 
- [node-problem-detector](https://github.com/kubernetes/node-problem-detector) - Detects if a node has been affected by an issue such as faulty hardware or kernel deadlocks, preventing scheduling. 
- [Reflector](https://github.com/emberstack/kubernetes-reflector) - Replicate a Secret or configMap between namespaces automatically. 
- [Reloader](https://github.com/stakater/Reloader) - Everytime a configMap or a Secret resource is created or changed, the pods that use them will be reloaded. 
- [Spegel](https://github.com/XenitAB/spegel) - Locally cache images from external registries with no explicit configuration.
- [Trivy operator](https://github.com/aquasecurity/trivy-operator) - Generates security reports automatically in response to workload and other changes to the cluster. 
- [Tailscale-operator](https://tailscale.com/kubernetes-operator/) - Provides a private load-balancer that generates entries to a zero-trust mesh VPN by annotating services or ingresses to use the operator. Think Ngrok plus all communication is encrypted (even non-SSL domain ingresses) but for free and easier to manage. 
- [Wavy](https://github.com/wavyland/wavy) - Patches Kubernetes resources with a VNC access using annotations to provide a GUI to any container. If you want to run for example, a containerized Skype client, you can access the application with a VNC using this add-on.

### For Baremetal setups
- [Democratic-CSI](https://github.com/democratic-csi/democratic-csi) - A CSI implementation for multiple ZFS-based network attached self-hosted storage systems. 

### Tool-based but still a few interesting add-on
- [kube-no-trouble](https://github.com/doitintl/kube-no-trouble) - To check if your current running version of Kubernetes and the resources that are a part of this cluster have been deprecated in future upgrades. 
- [krr](https://github.com/robusta-dev/krr) - Uses already existing Prometheus metrics stored to help on guiding the optimal usage of cluster resources. 
- [Prometheus Operator](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack) - Takes care about the grafana / prometheus setup 
- [CloudnativePG](https://artifacthub.io/packages/helm/cloudnative-pg/cloudnative-pg) - Best way to deploy postgres databases 
- [MariaDB Operator](https://artifacthub.io/packages/helm/mariadb-operator/mariadb-operator) - Manages your MariaDBs
- [Velero](https://artifacthub.io/packages/helm/vmware-tanzu/velero) - Takes care about your backups 
- [Glasskube](https://github.com/glasskube/operator) - Manages Open source tool installations (Gitlab, KeyCloak, Matomo Vault) 

