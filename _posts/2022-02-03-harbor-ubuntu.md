---
layout: post
title:  "Harbor Installation on Ubuntu"
date:   2022-02-03 12:35:00 -0530
categories: harbor, vm, ubuntu
---

## Pre-reqs for VM
* 2 vCPU
* 4GB RAM
* 200GB HDD
* Routable IP

## Install Docker compose on the Ubuntu VM

Before you begin the harbor installation, you need to install docker compose on your machine. To do this, get the latest release from [releases page](https://github.com/docker/compose/releases).

* Now run the following command

    > sudo curl -L "https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

* Next, give the executable permissions to the binary

    > sudo chmod +x /usr/local/bin/docker-compose

* Lastly, validate the version of the binary

    > docker-compose --version

## Install Harbor (Offline)

Once you have installed docker-compose, get the latest harbor version from [Harbor Release page](https://github.com/goharbor/harbor/releases). 

* Next download the latest release using the command below:

    > wget https://github.com/goharbor/harbor/releases/download/v1.10.10/harbor-offline-installer-v1.10.10.tgz

* Now, lets unpack the tgz

    > tar xzvf harbor-offline-installer-v1.10.10.tgz

* To use the HTTP or HTTPS ports, update the harbor.yml file located in `./harbor` directory

* Update the hostname in the harbor.yml file. Register this in your DNS Server

* Set the admin password to something complex. To update this, modify the value for the key `harbor_admin_password` in the `harbor.yml`

* Now, we can finally kick off the installation using the script under `./harbor` directory
    > sudo ./install.sh

    ``` 
    SAMPLE OUTPUT

    .....
    .....
    .....

    [Step 5]: starting Harbor ...
    [+] Running 10/10
    ⠿ Network harbor_harbor        Created                                                                                                                                         0.1s
    ⠿ Container harbor-log         Started                                                                                                                                         0.7s
    ⠿ Container registry           Started                                                                                                                                         2.9s
    ⠿ Container registryctl        Started                                                                                                                                         2.1s
    ⠿ Container harbor-db          Started                                                                                                                                         3.0s
    ⠿ Container redis              Started                                                                                                                                         2.8s
    ⠿ Container harbor-portal      Started                                                                                                                                         2.9s
    ⠿ Container harbor-core        Started                                                                                                                                         4.0s
    ⠿ Container harbor-jobservice  Started                                                                                                                                         5.3s
    ⠿ Container nginx              Started                                                                                                                                         5.5s
    ✔ ----Harbor has been installed and started successfully.----
    ```

* Once done, you can now access the harbor instance using the provided hostname in the `harbor.yml`

You now have a local harbor registry available to host your golden application images.