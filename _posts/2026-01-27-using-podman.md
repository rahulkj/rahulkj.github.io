---

title: "🐳 Podman to Generate Images"
date: 2026-01-27T12:32:00-06:00
draft: false
tags:
  - oci
  - kubernetes
  - containers
categories:
  - Kubernetes
  - Containers

---

## 🐧 Using Podman Desktop Instead of Docker
I've used Docker in the past, and offlate I found it super annoying for the following reasons:
* Pricing
* Complexity with experimental features to use insecure registries
* Battery hogger

So I moved to using podman. 

## 📥 Installation on Mac
There are 2 options to [install podman](https://podman.io/docs/installation#macos) on the mac. One is to use the installer and 2 is using `brew`.

> brew install podman

## 🏗️ Generating Images Using Dockerfile
Migration from docker to podman was simple. I had existing Dockerfiles in my app repo, and to generate OCI images, all I needed to do was to run `podman` cli and use the `Dockerfile` to build the image

* To generate the OCI images for amd64 and arm64, I used the following commands

```
podman build --platform linux/arm64 -t harbor.example.local/library/myapp:latest-arm64 .
    
podman push harbor.example.local/library/myapp:latest-arm64

podman build --platform linux/amd64 -t harbor.example.local/library/myapp:latest-amd64 .

podman push harbor.example.local/library/myapp:latest-amd64
```

* My k8s deployment file looks like this for deploying to Raspberry Pi

```
...

spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-site
  strategy: {}
  template:
    metadata:
      labels:
        app: app-site
    spec:
      containers:
      - image: harbor.example.local/library/myapp:latest-arm64
        imagePullPolicy: Always
...

```

and for linux based vms

```
...

spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-site
  strategy: {}
  template:
    metadata:
      labels:
        app: app-site
    spec:
      containers:
      - image: harbor.example.local/library/myapp:latest-amd64
        imagePullPolicy: Always
...

```

* Challenge with this approach is, when I try deploying my application into 2 different OS Arch's, I need to specify the correct image, else things will just fail. To do this, I modified my approach to create the images, load it up in the manifest and push it to harbor.

```
podman build --platform linux/amd64,linux/arm64 \
  --manifest harbor.example.local/library/myapp:latest .
    
podman manifest push --all harbor.example.local/library/myapp:latest \
  docker://harbor.example.local/library/myapp:latest
```

* Now my k8s deployment file looks simple, and when I deploy my application to any of the OS Arch's, k8s pulls the right image and runs my app.

Now my manifest looks like:
```
...

spec:
  replicas: 1
  selector:
    matchLabels:
      app: app-site
  strategy: {}
  template:
    metadata:
      labels:
        app: app-site
    spec:
      containers:
      - image: harbor.example.local/library/myapp:latest
        imagePullPolicy: Always
...

```

So learning is to always generate and package the OCI images along with the manifest.

Enjoy learning and sharing.