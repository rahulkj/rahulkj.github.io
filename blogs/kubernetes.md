Kubernetes
---

### Master node

ETCD - Backed database
default port 2379

Kube Scheduler - Deploys containers on the nodes

Controller Manager
- Node Controller (Health and placement of the containers)
- Replication Controller (desired state of the containers)

Kube API Server - Orchestrator


### Worker Node/s
Kubelet - run on each node (takes instructions from api server)
Kube-proxy - layer to allow containers to communicate with each other
