# Ansible ELK Example with Vagrant

This is a example of ELK stack deployment of ansible provisioned by Vagrant.

# Requirements

To bring up the Vagrant environment, it requires the followings to be install on your host machine:

1. Ansible (tested with 1.9.2)
2. sshpass (tested with 1.0.5)
3. Vagrant (tested with 1.7.2)

# ELK stack explanation

There are 10 virtual machines (5.75GB of RAM) defined in this stack:

| Hostname | IP | CPU | Memory | Role |
| ---------- | ---------- | ---------- | ---------- | ---------- |
| 10.elastic  | 10.10.10.10  | 4 | 768 MB | elasticsearch + Topbeat |
| 11.elastic  | 10.10.10.11  | 4 | 768 MB | elasticsearch + Topbeat |
| 12.elastic  | 10.10.10.12  | 4 | 768 MB | elasticsearch + kibana + Topbeat |
| 10.logstash  | 10.10.20.10  | 4 | 512 MB | logstash + Topbeat |
| 11.logstash  | 10.10.20.11  | 4 | 512 MB | logstash + Topbeat |
| 12.logstash  | 10.10.20.12  | 4 | 512 MB | logstash + redis + Topbeat |
| 13.logstash  | 10.10.20.13  | 4 | 512 MB | logstash + redis |
| 14.logstash  | 10.10.20.14  | 4 | 512 MB | logstash + redis |
| 10.postgres  | 10.10.30.10  | 4 | 512 MB | postgres + Packetbeat + Topbeat |
| 10.nginx  | 10.10.40.10  | 4 | 512 MB | fake-app + logstash-forwarder + Packetbeat + Topbeat |

## Responsibility of nodes

* The two nodes `10.elastic` and `11.elastic` are the elasticsearch cluster collecting logs from three logstash instances `12.logstash`, `13.logstash` and `14.logstash`.

* The node `11.elastic` is the monitor node of the elasticsearch cluster. There is also a elasticsearch instance on it, but it comes with the watcher plugin for querying `10.elastic` and `11.elastic` cluster. The kibana instance is also installed on this node for querying `10.elastic` and `11.elastic` cluster.

* The two nodes `10.logstash` and `11.logstash` are the log shippers receiving logs sent by logstash-forwarder from the `10.nginx` node and send them to the redis instance on the node `12.logstash`.

* The node `12.logstash` is the logstash indexer, which reads and processes logs from the local redis, and then index them into `10.elastic` and `11.elastic` cluster.

* The node `13.logstash` is the logstash indexer for the topbeats, which reads and processes logs from the local redis, and then index them into `10.elastic` and `11.elastic` cluster.

* The node `14.logstash` is the logstash indexer for the packetbeats, which reads and processes logs from the local redis, and then index them into `10.elastic` and `11.elastic` cluster.

* The node `10.postgres` is the postgresql database instance for the fake app on the node `10.nginx`.

* The node `10.nginx` is the application node which installed with a fake nodejs app I wrote. The purpose of the fake app is to generate PostgreSQL traffic, so that it can be visualized on the PostgreSQL dashboard of Kibana. Please see here for how to use it: https://github.com/rueian/fake-app. The logstash-forwarder instance on the node is responsible for sending log from local files `/var/log/syslog` and `var/log/auth.log`.

* All the `topbeat` instances are responsible for collecting system information like cpu usage and then send to the node `13.logstash`.

* All the `packetbeat` instances are responsible for collecting all the http and postgresql traffic and send processed packets to the node `14.logstash`.

## Change the stack

If you want to change this artitecture, you may need to modify the 3 files:

1. `Vagrantfile` which defined the hardware details of machines.
2. `inventory.ini` which is the inventory of Ansible Playbook.
3. `main.yml` which is the playbook containing IP configs as well.

**And if you want to change the `logstash` node, you also need to replace the ssl certs in `files/certs`**

The certs is used for communication between `logstash-forwarder` and `logstash` and is configured with CN: *.logstash, therefore you must replace them if you want to change the hostname of `logstash` node.

See [here](https://github.com/elastic/logstash-forwarder#important-tlsssl-certificate-notes) for generating a new cert.

# Bring up the stack

Run the command:

```shell
$ vagrant up
```

When finished, you should be able access `kibana` from http://10.10.10.12 and see the logs.

If you make changes in the playbook, run:

```
$ vagrant provision
```

# Knowing Issues

* Topbeat 1.0.0-beta3 can't generate proc field with elasticsearch 2.0. But it fixed in beta4.

* Kibana 4.2.0-beta2 can't use the index pattern `[index-]YYYY.MM.DD`, therefore if you wants to use the sample dashboards, you need to export the dashboard json and change all the `[packetbeat-]YYYY.MM.DD` to `packetbeat-*` ans change all the `[topbeat-]YYYY.MM.DD` to `topbeat-*` and then import the json back before using it.
