RabbitMQ Cluster setup on RHEL 6.4
---

#### Steps to turn off iptables:

- `sudo service iptables save`
- `sudo service iptables stop`
- `sudo chkconfig iptables off`

#### Steps to setup a RabbitMQ cluster on RHEL 6.4 64-bit machine:

##### Installation on all boxes:

- `sudo rpm -ivh  erlang-R15B-02.1.el6.x86_64.rpm` --> VMWare erlang version
- `sudo rpm -ivh  rabbitmq-server-3.1.5-1.noarch.rpm` --> RabbitMQ version
`sudo rabbitmq-plugins enable rabbitmq_management` --> Enable RabbitMQ management plugin
- `sudo chkconfig rabbitmq-server on` --> "Start the daemon by default when the system boots"
- `sudo rabbitmq-server -detached`
- `sudo rabbitmqctl status`

##### Setup Erlang cookie:

- view the erlang cookie on Node1 --> `sudo vi /var/lib/rabbitmq/.erlang.cookie`
- on Node 2 follow the following steps:
  - `sudo chmod 755 /var/lib/rabbitmq/.erlang.cookie`
  - `sudo vi /var/lib/rabbitmq/.erlang.cookie`
  - delete value and replace it with the value from Node 1, exit and save the file
  - `sudo chmod 400 /var/lib/rabbitmq/.erlang.cookie`
  - `sudo reboot`

##### Now the actual stuff, adding nodes to cluster:

- Node 2 --> `sudo rabbitmqctl stop_app`
- `sudo rabbitmqctl join_cluster rabbit@node1`
- `sudo rabbitmqctl start_app`

Login to Node1 management console and on overview page, you should find Node 1 and Node 2 listed. Repeat the above steps to join more nodes to the cluster.

Hope this was helpful!
