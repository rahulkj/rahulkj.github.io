---
layout: post
title:  "Kubernetes"
date:   2019-12-18 09:35:00 -0600
categories: kubernetes
---

## Architecture

Master consists of:
* API Server - Frontend to k8s to communicate with the cluster
* ETCD - Distributed key-value store to persist data on the cluster. Implements locks
* Scheduler - Distributes the workload across workers
* Controller - Monitor pods

Node consists of:
* Kubelet - agent on each node that is responsible to monitor the containers are running as expected
* Container Runtime - provide runtime to run the containers

Pod is the smallest object that exists in k8s. It encapsulate the container and provides storage, networking, etc to the container

Single pod can have multiple containers (of different type). These additional containers are called helper containers.

## apiVersion Reference

| kind | version |
| - | - |
| Pod | v1 |
| Service | v1 |
| ReplicaSet | apps/v1 |
| Deployment | apps/v1 |
| Ingress | extensions/v1beta1 |
| Job | batch/v1 |
| CronJob | batch/v1beta1 |
| ServiceAccount | v1 |
| NetworkPolicy | networking.k8s.io/v1 |

## kubectl basic commands

| Operation | Command |
| - | - |
| Get All namespaces | `kubectl get all --all-namespaces` |
| Get Cluster Info | `kubectl cluster-info` |
| Get any object | `kubectl get nodes/pods/services/etc` |

## Namespaces

Workspaces that are constrained to resources. Pre-provisioned namespaces are:
- kube-system
- kube-public
- default

```
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

| Operation | Command |
| - | - |
| Create | kubectl create namespace ingress-space |
| Delete | kubectl delete namespace ingress-space |
| Get Objects | kubectl get pods --namespace=space-name |
| Set context | kubectl config set-context $(kubectl config current-context) --namespace=some-space |

Referencing to services running in different pods:

`redis-service.dev.svc.cluster.local`

domain: `cluster.local`
Service: `svc`
Namespace: `dev`
Service name: `redis-service`

## ConfigMaps

| Operation | Command |
| - | - |
| Create | kubectl create configmap nginx-configuration --namespace some-space |

## Service Accounts

| Operation | Command |
| - | - |
| Create | kubectl create serviceaccount some-serviceaccount --namespace some-space |

## Roles, RoleBindings

| Operation | Command |
| - | - |
| List all | `kubectl get roles,rolebindings --all-namespaces` |
| List for a space | `kubectl get roles,rolebindings --namespace ingress-space` |

### Pods
```
apiVersion: v1                                  <-- apiVersion for Pod
kind: Pod
metadata:
  labels:
    name: webapp                                <-- Define label/s
  name: something
  namespace: default
spec:
  containers:
  - image: docker-repo/image-name:version       <-- Container image
    name: webapp
    ports:
    - containerPort: 8080                       <-- Any port bindings
      protocol: TCP
    resources: {}
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token
      readOnly: true
  securityContext: {}
  serviceAccount: default                       <-- Service Account details
  serviceAccountName: default
  tolerations:                                  <-- if the nodes are tainted
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
```

| Operation | Command |
| - | - |
| Run/Create | kubectl run name --image=some-image |
| Create | kubectl create -f pod-definition.yml |
| Get | kubectl get pods |
| Update | kubectl apply -f pod-definition.yml |
| Update a paramater | kubectl set image deployment/deployment-name name=image |
| Status | kubectl rollout status deployment/deployment-name |
| History | kubectl rollout history deployment/deployment-name |
| Rollback | kubectl rollout undo deployment/deployment-name |
| Delete | kubectl delete pod pod-name |
| Get Details | kubectl describe pod pod-name |

Generate the pod manifest yml:
`kubectl run --generator=run-pod/v1 nginx --image=nginx --dry-run -o yaml`

## ReplicaSet

Ensures the given number of containers are running. IF we edit the replicaset definition, and run the `replace` command, it will not update the pods, with the new configuration. One has to delete the pods manually, so the new pods will get the latest configuration.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    name: webapp
  name: frontend
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
      - image: docker-repo/image-name:version       <-- Container image
        name: webapp
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
```

| Operation | Command |
| - | - |
| Create | kubectl create -f rs-definition.yml |
| Get | kubectl get replicaset |
| Replace | kubectl replace -f rs-definition.yml |
| Scale | kubectl scale --replicas=x -f rs-definition.yml |
| Delete | kubectl delete replicaset rs-name |

## Deployments

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    name: webapp
  name: frontend
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate                             <-- Recreate or RollingUpdate
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
      - image: docker-repo/image-name:version       <-- Container image
        name: webapp
        ports:
        - containerPort: 8080
          protocol: TCP
        resources: {}
```

| Operation | Command |
| - | - |
| Create | kubectl create -f deployment-definition.yml |
|  | kubectl run --generator=deployment/v1beta1 nginx --image=nginx |
| Get | kubectl get deployments |
| Update | kubectl apply -f deployment-definition.yml |
| Update a paramater | kubectl set image deployment/deployment-name name=image |
| Status | kubectl rollout status deployment/deployment-name |
| History | kubectl rollout history deployment/deployment-name |
| Rollback | kubectl rollout undo deployment/deployment-name |
| Generate | kubectl create deployment --image=nginx nginx --dry-run -o yaml |

## Services

NodePort - Service to allow app to be accessible on the port exposed by the node. Map port on the node to the port on the pod

```
apiVersion: v1                        <-- Service api version
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  type: NodePort
  ports:
  - nodePort: 30080                   <-- Port on the node (30000 - 32767)
    port: 8080                        <-- Port on the service (mandatory)
    protocol: TCP
    targetPort: 8080                  <-- Port of the app
  selector:
    name: webapp                      <-- binding service to the pod's that have the label webapp
```

ClusterIP - Virtual IP created inside the cluster to enable communication between services
```
apiVersion: v1                        <-- Service api version
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  type: ClusterIP                     <-- Default type
  ports:
  - port: 8080                        <-- Port on the service (mandatory)
    protocol: TCP
    targetPort: 8080                  <-- Port of the app
  selector:
    name: webapp                      <-- binding service to the pod's that have the label webapp
```

Load Balancer - Provisions loadBalancer on supported IaaS
```
apiVersion: v1                        <-- Service api version
kind: Service
metadata:
  name: webapp-service
  namespace: default
spec:
  type: NodePort
  ports:
  - nodePort: 30080                   <-- Port on the node (30000 - 32767)
    port: 8080                        <-- Port on the service (mandatory)
    protocol: TCP
    targetPort: 8080                  <-- Port of the app
  selector:
    name: webapp                      <-- binding service to the pod's that have the label webapp
```

| Operation | Command |
| - | - |
| Create | kubectl create -f service-definition.yml |
| Get | kubectl get services |
| Update | kubectl apply -f service-definition.yml |
| Delete | kubectl delete service service-name |

Ingress Networking
- SSL Security
- Layer 7 routing
- Route based routing

Ingress Controller (nginx/proxy/traefik/contour/istio)
- nothing provided by default
- GCP comes with LB and/or Nginx

Commands:
`kubectl get ingress --all-namespaces`

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: ingress-wear-watch
  namespace: app-space
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: wear-service
          servicePort: 8080
        path: /wear
      - backend:
          serviceName: video-service
          servicePort: 8080
        path: /stream
status:
  loadBalancer:
    ingress:
    - {}
```

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

```
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: ingress-controller
  namespace: ingress-space
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nginx-ingress
  template:
    metadata:
      labels:
        name: nginx-ingress
    spec:
      serviceAccountName: ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.21.0
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --default-backend-service=app-space/default-http-backend
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: POD_NAMESPACE
              valueFrom:
                fieldRef:
                  fieldPath: metadata.namespace
          ports:
            - name: http
              containerPort: 80
            - name: https
              containerPort: 443
```

Ingress resources (pods/deployments/services)

## Jobs
```
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job-1
  labels:
    name: something
  namespace: default
spec:
  completions: 3                                  <-- successful completion
  parallelism: 3                                  <-- parallel execution, instead of sequential
  template:
    metadata:
      labels:
        name: some-job
    spec:
      containers:
      - image: docker-repo/image-name:version       <-- Container image
        name: some-job
        command: ['echo', 'Hello World']
      restartPolicy: Never
```

| Operation | Command |
| - | - |
| Create | kubectl create -f job-definition.yml |
| Get | kubectl get jobs |
| Logs | kubectl logs pod-name |
| Delete | kubectl delete job job-name |


Cron Jobs:
```
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: batch-job-1
  labels:
    name: something
  namespace: default
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      completions: 3                                  <-- successful completion
      parallelism: 3                                  <-- parallel execution, instead of sequential
      template:
        metadata:
          labels:
            name: some-job
        spec:
          containers:
          - image: docker-repo/image-name:version       <-- Container image
            name: some-job
            command: ['echo', 'Hello World']
          restartPolicy: Never
```

| Operation | Command |
| - | - |
| Create | kubectl create -f job-definition.yml |
| Get | kubectl get cronjobs |
| Logs | kubectl logs pod-name |
| Delete | kubectl delete cronjob job-name |


## Network Policies
Define ingress and egress rules, for container to container access. Default is allow all access.

kube-router
calico
romana
weave-net

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: db-pod
    ports:
    - protocol: TCP
    port: 3306
```

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: app-policy
spec:
  podSelector:
    matchLabels:
      role: app
  policyTypes:
  - Egress
  egress:
  - to:
    - podSelector:
        matchLabels:
          name: db-pod
    ports:
    - protocol: TCP
    port: 3306
```

| Operation | Command |
| - | - |
| Get | kubectl get networkpolicies |
| Describe | kubectl describe networkpolicy payroll-policy |
| Create | kubectl create -f networkpolicy-definition.yml |
| Delete | kubectl delete networkpolicy policy-name |


## Volumes
Data is destroyed along with containers. So if you want to retain the data, you need to attach persistent disk to the containers