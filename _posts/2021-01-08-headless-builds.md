---
layout: post
title:  "Headless builds for container images"
date:   2021-01-08 16:00:00 -0600
categories: k8s, buildah, skopeo, containerd
---

I was looking for options on how I could build my container images, without install docker on my linux machines. I was introduced to the https://github.com/containers repository that is a holy grail of awesome tools for dealing with containers.

Once such tool that can be used to build images is [buildah](https://github.com/containers/buildah). Easy to install and use.

Let's see this into action.

I have a [Dockerfile](https://github.com/rahulkj/sample-k8s-app/Dockerfile). Traditionally I used to install docker engine on my operating system, and then executed the following commands:

```
# Build the image using the Dockerfile
docker build . -t rjain/sample-k8s-app

# Push the docker file up to dockerhub
docker push rjain/sample-k8s-app
```

Now issue with this approach is, I have to install docker engine, which itself is a pain. So lets take a look at `buildah`

First we install this cli `buildah` using the [documentation](https://github.com/containers/buildah/blob/master/install.md)

Once we have the cli on the machine, we'll create the above container image, but using the `buildah` cli

```
buildah bud -t rjain/sample-k8s-app .

STEP 1: FROM openjdk:8-jdk-alpine
STEP 2: ARG JAR_FILE=target/*.jar
STEP 3: COPY ${JAR_FILE} app.jar
STEP 4: ENTRYPOINT ["java","-jar","/app.jar"]
STEP 5: COMMIT rjain/sample-k8s-app
Getting image source signatures
Copying blob f1b5933fe4b5 skipped: already exists
Copying blob 9b9b7f3d56a0 skipped: already exists
Copying blob ceaf9e1ebef5 skipped: already exists
Copying blob 09a656726106 done
Copying config 14509f1550 done
Writing manifest to image destination
Storing signatures
--> 14509f1550e
14509f1550e95c3fb5c7dfda02422b5e58a83d7b81f012f7be232f8aff0a2b01
```

Let's validate the images on the local machine

```
buildah images

REPOSITORY                       TAG            IMAGE ID       CREATED          SIZE
localhost/rjain/sample-k8s-app   latest         14509f1550e9   34 minutes ago   124 MB
<none>                           <none>         47822cf9918f   10 days ago      123 MB
docker.io/library/openjdk        8-jdk-alpine   a3562aa0b991   20 months ago    106 MB
```

Next, let's push this image to dockerhub

```
buildah push --creds=rjain:xxx-xxx-xxx-xx-xxxxx localhost/rjain/sample-k8s-app rjain/sample-k8s-app

Getting image source signatures
Copying blob 09a656726106 done
Copying blob 9b9b7f3d56a0 skipped: already exists
Copying blob ceaf9e1ebef5 skipped: already exists
Copying blob f1b5933fe4b5 skipped: already exists
Copying config 14509f1550 done
Writing manifest to image destination
Storing signatures
```

Pay attention to the source and destinations.

Now that we have the image pushed, we can use another tool to inspect the conatiner images. The tool that we would install and use is [skopeo](https://github.com/containers/skopeo).

The usage is very simple:

```
skopeo inspect docker://rjain/sample-k8s-app

{
    "Name": "docker.io/rjain/sample-k8s-app",
    "Digest": "sha256:4cdafa91be0383c0d3f7b13a6d772c2c8dace93a4b2f667fdb54064f9d0586c7",
    "RepoTags": [
        "latest",
        "v1"
    ],
    "Created": "2021-01-08T16:40:17.426637258Z",
    "DockerVersion": "",
    "Labels": null,
    "Architecture": "amd64",
    "Os": "linux",
    "Layers": [
        "sha256:e7c96db7181be991f19a9fb6975cdbbd73c65f4a2681348e63a141a2192a5f10",
        "sha256:f910a506b6cb1dbec766725d70356f695ae2bf2bea6224dbe8c7c6ad4f3664a2",
        "sha256:c2274a1a0e2786ee9101b08f76111f9ab8019e368dce1e325d3c284a0ca33397",
        "sha256:c344f1d964bb4b6192419c84f5b4da83ca1d07e3a8f81ed1ae18c587d2f852c8"
    ],
    "Env": [
        "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/jvm/java-1.8-openjdk/jre/bin:/usr/lib/jvm/java-1.8-openjdk/bin",
        "LANG=C.UTF-8",
        "JAVA_HOME=/usr/lib/jvm/java-1.8-openjdk",
        "JAVA_VERSION=8u212",
        "JAVA_ALPINE_VERSION=8.212.04-r0"
    ]
}
```

The cli informs us all about the image. `skopeo` can also be used to pull down images from dockerhub, and host them on an internal container registry. For ex: if we want to pull down the windows pause container image and have it on our local machine, we can run the following command:

```
skopeo copy --override-os windows --override-arch multiarch docker://mcr.microsoft.com/oss/kubernetes/pause:1.2.0-windows-1809-amd64 docker-archive:/tmp/windows.tar

Getting image source signatures
Copying blob cf10787548c4 done
Copying blob 5e66bc671ba5 done
Copying blob 065af5c9a486 done
Copying config fbe28212db done
Writing manifest to image destination
Storing signatures
```

Hope this simplifies your CI/CD of container images, and avoids the pain of installing docker engine.

Cheers and Happy New Year!! God bless all of us!