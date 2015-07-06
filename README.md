# Ansible ELK Example with Vagrant

This is a example of ELK stack deployment of ansible provisioned by Vagrant.

# Requirements

To bring up the Vagrant environment, it requires the followings to be install on your host machine:

1. Ansible (tested with 1.9.2)
2. sshpass (tested with 1.0.5)
3. Vagrant (tested with 1.7.2)

# ELK stack explanation

There are 6 virtual machines defined in this stack:

| Hostname | IP | CPU | Memory | Role |
| ---------- | ---------- | ---------- | ---------- | ---------- |
| elastic10  | 10.10.10.10  | 4 | 768 MB | elasticsearch + Marvel + Watcher + Packetbeat |
| elastic11  | 10.10.10.11  | 4 | 768 MB | elasticsearch + Marvel + Watcher + Packetbeat |
| logstash10  | 10.10.20.10  | 4 | 512 MB | logstash |
| logstash-forwarder10  | 10.10.30.10  | 4 | 256 MB | logstash-forwarder + Packetbeat |
| logstash-forwarder11  | 10.10.30.11  | 4 | 256 MB | logstash-forwarder + Packetbeat |
| kibana  | 10.10.40.10  | 4 | 512 MB | kibana + nginx + Packetbeat |

## Responsibility of roles

* The two `logstash-forwarder` nodes will collect their own `/var/log/syslog` and `var/log/auth.log` and send to `logstash` node.

    See configurations: `roles/logstash-forwarder/templates/logstash-forwarder.conf.j2`

* The `logstash` node will receive logs send by `logstash-forwarder` and process them. Then it will send processed log to `elasticsearch` cluster. (By default, logstash communicates with elasticsearch using 'node' protocol.)

    See configurations: `roles/logstash/templates/logstash.conf.j2`


* The `elasticsearch` cluster will store logs send by `logstash`.

    See configurations: `roles/elastic/templates/elasticsearch.yml.j2`


* The `kibana` node is a web dashboard of `elasticsearch` cluster. Nginx is installed as reverse proxy of kibana.

    See configurations: `roles/kibana/templates/kibana.yml.j2`

    See configurations: `roles/kibana/templates/nginx-sites-available-default.j2`

* The `packetbeat` will parse the network traffic (http, redis, etc.) and send to elasticsearch.

    See configurations: `roles/packetbeat/templates/packetbeat.yml.j2`

## Change the stack

If you want to change this artitecture, you may need to modify the 3 files:

1. `Vagrantfile` which defined the hardware details of machines.
2. `inventory.ini` which is the inventory of Ansible Playbook.
3. `main.yml` which is the playbook containing IP configs as well.

**And if you want to change the `logstash` node, you also need to replace the ssl certs in `files/certs`**

The certs is used for communication between `logstash-forwarder` and `logstash` and is configured with SAN: 10.10.20.10, therefore you must replace them if you want to change the IP of `logstash` node.

See [here](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-4-on-ubuntu-14-04#generate-ssl-certificates) for generating a new cert.

# Bring up the stack

Run the command:

```shell
$ vagrant up
```

When finished, you should be able access `kibana` from http://10.10.40.10 and see the logs which comes from `logstash-forwarder` nodes.

You can also access Marvel monitor of `elasticsearch` cluster from http://10.10.10.10:9200/_plugin/marvel or http://10.10.10.11:9200/_plugin/marvel

If you make changes in the playbook, run:

```
$ vagrant provision
```

# Knowing Issues

The elasticsearch 1.6.0 installed from the apt repo will not able to start after rebooting the machine. See https://github.com/elastic/elasticsearch/issues/11594



