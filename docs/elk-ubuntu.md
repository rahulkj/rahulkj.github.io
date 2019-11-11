Steps for setting up Elasticsearch, logstash and Kibana on Ubuntu 15.04
---

Here's a dump of all the steps you need to perform to setup a ELK instance on your VM

`sudo apt-get update && sudo apt-get install openssh-server && sudo service ssh restart`

## SSH to the vm and execute the following steps:

```
sudo ufw disable #Administrators will not like this :-), but this is a development setup

sudo add-apt-repository -y ppa:webupd8team/java

wget -O - http://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -

echo 'deb http://packages.elasticsearch.org/elasticsearch/1.6/debian stable main' | sudo tee /etc/apt/sources.list.d/elasticsearch.list
echo 'deb http://packages.elasticsearch.org/logstash/1.5/debian stable main' | sudo tee /etc/apt/sources.list.d/logstash.list

sudo apt-get update
sudo apt-get -y install oracle-java8-installer

sudo apt-get -y install elasticsearch=1.6.0
sudo vi /etc/elasticsearch/elasticsearch.yml #Set network.host: localhost
sudo service elasticsearch restart
sudo update-rc.d elasticsearch defaults enable 95 10

cd ~; wget https://download.elasticsearch.org/kibana/kibana/kibana-4.1.0-linux-x64.tar.gz
tar xvf kibana-*.tar.gz
vi ~/kibana-4*/config/kibana.yml #Set host: ""
sudo mkdir -p /opt/kibana
sudo cp -R ~/kibana-4*/* /opt/kibana/
cd /etc/init.d && sudo wget https://gist.githubusercontent.com/thisismitch/8b15ac909aed214ad04a/raw/bce61d85643c2dcdfbc2728c55a41dab444dca20/kibana4
sudo chmod +x /etc/init.d/kibana4
sudo update-rc.d kibana4 defaults enable 96 9
sudo service kibana4 start

sudo apt-get install logstash
sudo vi /etc/logstash/conf.d/logstash-filter.conf
sudo update-rc.d logstash defaults enable 97 8
sudo service logstash start
```

## Sample filter configuration:
```
input {
  tcp {
    port => 5000
    type => syslog
  }
  udp {
    port => 5000
    type => syslog
  }
}

filter {
  if [type] == “syslog” {
    grok {
      match => { “message” => “%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}” }
      add_field => [ “received_at”, “%{@timestamp}” ]
      add_field => [ “received_from”, “%{host}” ]
    }
    syslog_pri { }
    date {
      match => [ “syslog_timestamp”, “MMM d HH:mm:ss”, “MMM dd HH:mm:ss” ]
    }
  }
}

output {
  elasticsearch { host => localhost }
  stdout { codec => rubydebug }
}
```
## Credits to:
How To Install Elasticsearch, Logstash, and Kibana 4 on Ubuntu 14.04 | [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-4-on-ubuntu-14-04)
Logstash Configuration [Examples](https://www.elastic.co/guide/en/logstash/current/config-examples.html)
