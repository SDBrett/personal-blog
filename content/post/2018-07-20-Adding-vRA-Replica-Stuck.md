---
author: Brett Johnson
categories:
- vRA
date: "2018-07-20T00:00:00Z"
image: /assets/images/sdbrett-logo.svg
summary: RabbitMQ causes join cluster process to hang when adding new vRA nodes.
tags:
- vRA
- VMware
title: Adding vRA Replica Stuck
---

Recently when attempting to add additional vRA appliances to a 7.4 cluster, I came across an issue where the process became stuck.

After clicking `Join Cluster` the on screen status became stuck at 48% with the message `Finishing rabbitmq-server`. The node was left alone for a couple of hours, but there was no progress.

The node was not dead as `/var/log/messages` was still receiving updates, including the join progression status. At this time the only error in the logs was from the postgres DB, which was fine due to the current circumstance, so I started to look at what RabbitMQ was doing.

The command service `rabbitmq-server status` showed some information about the current state.
```
# systemctl rabbitmq-server status

DIAGNOSTICS
===========

attempted to contact: ['rabbit@vra03']

rabbit@rabbit@vra03
  * connected to epmd (port 4379) on vra03
  * epmd reports node 'rabbit' running on port 25682
  * TCP connection succeeded but Erlang distribution failed
  * suggestion: hostname mismatch?
  * suggestion: is the cookie set correctly?
  * suggestion: is the Erlang distribution using TLS?

current node details:
- node name: 'rabbitmq-cli-1045@vra03'
- home dir: /var/lib/rabbitmq
- cookie hash: ***

```


Attempts to the stop the server were OK, but starting the service would just hang.

A few moments (hours) of troubleshooting (swearing) later, I came across this [article](https://docs.vmware.com/en/vRealize-Automation/7.4/com.vmware.vra.install.upgrade.doc/GUID-C8973C96-A82F-4C78-A51C-AE50142E73AB.html) on RabbitMQ hostnames. It was a simple thing to test, so I followed the steps. 

NOTE: At this point, DNS and network connectivity between the vRA nodes had been verified.

Comparing the file `/etc/rabbitmq/rabbitmq-env.conf` between the new and existing nodes showed a difference.

Existing node

```
NODENAME=rabbit@vra01.corp.local
USE_LONGNAME=true
```

New Node

```
NODENAME=rabbit@vra03
USE_LONGNAME=true
```

At this point, I should state that the new nodes were deployed with the node hostname / IP address as the FQDN.

I updated The `NODENAME` setting on the new nodes to be the FQDN instead of shortname and restarted the `rabbitmq-server` service. This produced a great success, my first error message was gone. But I had a new, enthralling error message.

```
attempted to contact: [rabbit@vra03.corp.local]

rabbit@vra03:
  * connected to epmd (port 4369) on vra03
  * epmd reports node 'rabbit' running on port 25682
  * TCP connection succeeded but Erlang distribution failed

Hostname mismatch: node "rabbit@vra03.corp.local" believes its host is different. Please ensure that hostnames resolve the same way locally and on "rabbit@vra03.corp.local"

current node details:
- node name: 'rabbitmq-cli-30@vra03.corp.local'
- home dir: /var/lib/rabbitmq
- cookie hash: 
```

After troubleshooting until my voice was horse, I ran `ps ax | grep rabbit` and found that even though the `rabbitmq-server` service was stopped, there were still RabbitMQ processes running. Most of the processes appeared to have been started by VAMI as part of the cluster join process.

I killed all the PIDs and restarted `rabbitmq-server`, this time it started without issue. The information returned by `service rabbitmq-server status` was correct. 

The next attempt to join the cluster worked successfully.

On the other node I was attempting to add to the cluster, instead of stopping the services, I rebooed the node and successfully joined the cluster.

If you come across this issue and attempt to resolve it yourself, make sure that your vRA environment is backed up and take take snapshots before attempting anything.

TL:DR - Before adding vRA 7.4 appliances to a cluster, check the `rabbitmq-env.conf` file to validate the settings.