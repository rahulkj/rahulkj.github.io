---
layout: post
title: "Steps to install Confluent Platform manually"
date: 2025-09-06 19:20:00 +0530
categories: confluent platform, installation, ubuntu
---

## Install packages on all machines
```
sudo mkdir -p /etc/apt/keyrings
wget -qO - https://packages.confluent.io/deb/8.0/archive.key | gpg --dearmor | sudo tee /etc/apt/keyrings/confluent.gpg > /dev/null
CP_DIST=$(lsb_release -cs)
echo "Types: deb
URIs: https://packages.confluent.io/deb/8.0
Suites: stable
Components: main
Architectures: $(dpkg --print-architecture)
Signed-by: /etc/apt/keyrings/confluent.gpg

Types: deb
URIs: https://packages.confluent.io/clients/deb/
Suites: ${CP_DIST}
Components: main
Architectures: $(dpkg --print-architecture)
Signed-By: /etc/apt/keyrings/confluent.gpg" | sudo tee /etc/apt/sources.list.d/confluent-platform.sources > /dev/null

sudo apt-get update && sudo apt-get install -y confluent-platform openjdk-21-jdk
```


# KRaft installation

* Begin with editing the `controller.properties`
```
sudo vi /etc/kafka/controller.properties
```

* Edit the following fields and provide the right details. Node Id should correspond to the current host
```
node.id=1   # set it to 1/2/3
controller.quorum.voters=1@node-1:9093,2@node-2:9093,3@node-3:9093
#controller.quorum.bootstrap.servers=localhost:9093

log.dirs=/tmp/kraft-controller-logs # Update it based on where you want the files to be
```

* Generate a UUID that will be use for the installation. This is going to be the same for this cluster installation
```
kafka-storage random-uuid
```

* Generate the `logs.dir`
```
kafka-storage format -t kRJxsfCzQCSJn4If-AZsiA -c /etc/kafka/controller.properties
Formatting metadata directory /tmp/kraft-controller-logs with confluent.metadata.version 4.0-IV3A.
```

* Validate the contents in the `logs.dir`
```
ls /tmp/kraft-controller-logs/
bootstrap.checkpoint  __cluster_metadata-0  meta.properties
```

* Manually start the controller
```
sudo kafka-server-start /etc/kafka/controller.properties
```

* Repeat the steps above on all the controller nodes
* Check the quorum of the controllers
```
kafka-metadata-quorum --bootstrap-controller node-1:9093,node-2:9093,node-3:9093 describe --status

ClusterId:              kRJxsfCzQCSJn4If-AZsiA
LeaderId:               2
LeaderEpoch:            14
HighWatermark:          531
MaxFollowerLag:         0
MaxFollowerLagTimeMs:   248
CurrentVoters:          [{"id": 1, "directoryId": null, "endpoints": ["CONTROLLER://node-1:9093"]}, {"id": 2, "directoryId": null, "endpoints": ["CONTROLLER://node-2:9093"]}, {"id": 3, "directoryId": null, "endpoints": ["CONTROLLER://node-3:9093"]}]
CurrentObservers:       []
```

* Register the controller service
```
sudo chown -R cp-kafka:confluent /tmp/kraft-controller-logs
sudo chown -R cp-kafka:confluent /var/log/kafka /var/log/confluent

sudo vi /usr/lib/systemd/system/confluent-server.service
## EDIT THE ExecStart=/usr/bin/kafka-server-start /etc/kafka/controller.properties

sudo systemctl daemon-reload
sudo systemctl start confluent-server.service
sudo systemctl status confluent-server.service

journalctl -u confluent-server.service -f
```

* If all goes well, then register the service on each node
```
sudo systemctl enable confluent-server.service
```

## Broker installation

* Begin with editing the `broker.properties`

```
sudo vi /etc/kafka/broker.properties

# The node id associated with this instance's roles
node.id=4

# Information about the KRaft controller quorum.
# controller.quorum.bootstrap.servers=node-1:9093,node-2:9093,node-3:9093
controller.quorum.voters=1@node-1:9093,2@node-2:9093,3@node-3:9093

listeners=PLAINTEXT://:9092

advertised.listeners=PLAINTEXT://:9092
```

* Using the same UUID that was generated for controllers, generate the `logs.dir`
```
kafka-storage format -t kRJxsfCzQCSJn4If-AZsiA -c /etc/kafka/broker.properties
Formatting metadata directory /tmp/kraft-broker-logs with confluent.metadata.version 4.0-IV3A.
```

* Manually start the broker
```
sudo kafka-server-start /etc/kafka/broker.properties
```

* Register the broker service
```
sudo chown -R cp-kafka:confluent /tmp/kraft-controller-logs
sudo chown -R cp-kafka:confluent /var/log/kafka /var/log/confluent

sudo vi /usr/lib/systemd/system/confluent-server.service
## EDIT THE ExecStart=/usr/bin/kafka-server-start /etc/kafka/broker.properties

sudo systemctl daemon-reload
sudo systemctl start confluent-server.service
sudo systemctl status confluent-server.service

journalctl -u confluent-server.service -f
```

* If all goes well, then register the service on each node
```
sudo systemctl enable confluent-server.service
```

## Schema Registry

```
vi /etc/schema-registry/schema-registry.properties

# Specify the address the socket server listens on, e.g. listeners = PLAINTEXT://your.host.name:9092
listeners=http://0.0.0.0:8081

# The advertised host name. This must be specified if you are running Schema Registry
# with multiple nodes.
host.name=node-7

# List of Kafka brokers to connect to, e.g. PLAINTEXT://hostname:9092,SSL://hostname2:9092
kafkastore.bootstrap.servers=PLAINTEXT://node-4:9092,PLAINTEXT://node-5:9092,PLAINTEXT://node-6:9092
```

```
sudo systemctl start confluent-schema-registry

sudo systemctl status confluent-schema-registry
```

## Connect

```
sudo systemctl start confluent-kafka-connect

sudo systemctl status confluent-kafka-connect
```

```
sudo systemctl start confluent-kafka-rest

sudo systemctl status confluent-kafka-rest
```


## ksqlDB Installation

```
sudo vi /etc/ksqldb/ksql-server.properties

listeners=http://0.0.0.0:8088

advertised.listener=http://0.0.0.0:8088

bootstrap.servers=node-4:9092,node-5:9092,node-6:9092

ksql.schema.registry.url=http://node-7:8081
```

```
sudo systemctl start confluent-ksqldb

sudo systemctl status confluent-ksqldb
```

## Control Center Installation 

Follow the details listed in the [link](https://docs.confluent.io/control-center/current/installation/overview.html)
```
wget https://packages.confluent.io/confluent-control-center-next-gen/archive/confluent-control-center-next-gen-2.2.0.tar.gz

tar -xvf confluent-control-center-next-gen-2.2.0.tar.gz

cd confluent-control-center-next-gen-2.2.0

export C3_HOME=`pwd`
```