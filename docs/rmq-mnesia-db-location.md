Changing RabbitMQ mnesia database location
---

Recently, I had a requirement to install and setup a RabbitMQ cluster on RHEL. (Refer to my post regarding Installing RabbitMQ on RHEL).

The foot print of this machine was too low. The base hard disk was just 4GB.
The administrator provisioned 40GB and mounted `/opt` directory to it.

Now at this moment I had to update my current cluster nodes to use the `/opt` directory as opposed to `/var/lib/rabbitmq` directory.

Following are the steps I performed to ensure I don't loose any data, and at the same time, update my mnesia database location from `/var/lib/rabbitmq/mnesia` to `/opt/mnesia`


#### Step 1: Reset one of the node on the cluster by running the following commands
```
sudo /usr/sbin/rabbitmqctl cluster_status
sudo /usr/sbin/rabbitmqctl status
sudo /usr/sbin/rabbitmqctl stop_app
sudo /usr/sbin/rabbitmqctl reset
```

#### Step 2: Create and move the mnesia folder using the following commands
```
sudo mkdir /opt/mnesia
sudo cp -r /var/lib/rabbitmq/mnesia/* /opt/mnesia/
sudo chown -R rabbitmq:rabbitmq /opt/mnesia
sudo chmod 766 /opt/mnesia
```

#### Step 3: Update the RabbitMQ Environment to set the `RABBITMQ_MNESIA_BASE` to the new folder that we created
```
touch /tmp/rabbitmq-env.conf
cat <> /tmp/rabbitmq-env.conf
CONFIG_FILE=/tmp/rabbitmq
RABBITMQ_MNESIA_BASE=/opt/mnesia

sudo mv /tmp/rabbitmq-env.conf /etc/rabbitmq/
```

#### Step 4: Reboot the system and now join this node to the existing cluster nodes
```
sudo /usr/sbin/rabbitmqctl status
sudo /usr/sbin/rabbitmqctl stop_app
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl join_cluster
sudo /usr/sbin/rabbitmqctl start_app
```

At this moment, login to the management console, to see the extended disk space on the overview page.
All the exchanges/queues and messages should be intact.

That's it!! Perform the Steps 1 to 4 on other nodes, one at a time.
Finally moved away from `4GB` disk to `40GB`.

**NOTE:**
If you have a cluster of 3 nodes, and in an event of network partition, the following setting comes very handy.

```
touch /tmp/rabbitmq.config
cat <> /tmp/rabbitmq.config
[{rabbit, [
{ cluster_partition_handling, [pause_minority] }
]}
].
End

sudo mv /tmp/rabbitmq.config /etc/rabbitmq/
```
