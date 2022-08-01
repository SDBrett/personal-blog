---
author: Brett Johnson
categories:
- Kubernetes
- Kafka
date: "2022-08-01"
image: /assets/images/Apache_kafka_logo.svg
tags:
- Kafka
title: Kafka Listeners
versions:
- software: Kafka
  versions:
  - 3.2.0
draft: false
---

# Kafka Listeners

Kafka uses three settings to configure how client can connect to brokers within a cluster; `lister.security.protocol.map`, `listeners` and `advertised.listeners`. This page  demonstrate how to configure Kafka to for different client connectivity scenarios.

Before looking at different scenarios, let's go through how to configure Kafka listeners.

## Configuring Listeners
The `listener` configuration is a comma separated list that defines what interfaces, ports and associated security protocols Kafka will use to listen for client connections. A listener can be configured using the name of a security protocol or a listener can be named using the setting `lister.security.protocol.map`. Naming listeners is not required but can help make the intent of a specific listener clearer.

Note: Lister names and port numbers must be unique.

The `listener` setting format is `<LISTENERNAME or  PROTOCOL>://<HOSTNAME>:<PORT>`.
- A HOSTNAME value of '0.0.0.0' binds the lister to all interfaces.
- An HOSTNAME value binds the listener to the default interface.
- Set the HOSTNAME value to the IP address of an interface to bind the listener to a specific interface.

`listener: SSL://0.0.0.0:9091` - Use all interfaces to listen on port 9091 for incoming SSL connections
`listener: PLAINTEXT://192.168.1.43:9092` - Use the interface with IP address 192.168.1.43 to listen on port 9092 for incoming PLAINTEXT connections
`listener: EXTERNAL://172.20.20.32:9093` - Use the interface with IP address 172.20.20.32 to listen on port 9093 for incoming connections. The lister name and protocol must be configured using `lister.security.protocol.map` for this to work.

Multiple listeners could be configured like this: `listener: SSL://192.168.1.43:9091,PLAINTEXT://192.168.1.43:9092,EXTERNAL://172.20.20.32:9093` which may suit a multi network environment.

## Named Listeners

Named listeners are defined by using the `lister.security.protocol.map` setting to create a map between a listener name and security protocol.

The configuration `lister.security.protocol.map: PUBLIC:PLAINTEXT` defines a single listener named 'PUBLIC' with the 'PLAINTEXT' security protocol. 
The configuration `lister.security.protocol.map: PUBLIC:PLAINTEXT, EXTERNAL:SSL` defines an additional listener called 'EXTERNAL' with the 'SSL'' security protocol

## Advertised Listeners

In a multi-broker cluster scenarios a front end load balancer would be used to forward a clients initial connection to one of the brokers. The broker provides a new address which is specific to that broker, to the client. The returned address is based on the listener (interface and port) which the broker received initial connection on. This address is the 'advertised listener address' and is what the client will use for future connections to the cluster.

Advertised listeners are unique to each broker, the cluster will not start if multiple brokers have the same advertised listener addresses set.

The above scenario demonstrates how advertised listeners allow balancing of initial connections while providing persistence for ongoing connectivity.

![Advertised listener client connection](/assets/images/kafka-advertised-listener-uml.svg)

## Additional Configurations

You can specify the listener for inter-broker communicate using the setting `inter.broker.listener.name`. If you're not using named listeners you can use `security.inter.broker.protocol` to set inter-broker communication by lister security protocol.  Note that Kafka will not store if both `inter.broker.listener.name` and `security.inter.broker.protocol` are configured.


## Scenario 1: Simple Topology

Our first scenario is a simple one network topology with a load balancer in front of the Kafka brokers. The load balancer is used to ensure clients connect to a healthy broker when initially attempting connection.

- The client first connects to the load balancer which forwards the connection to one of the brokers. 
- The broker is receiving an SSL connection on port 9091 which the SSL listener is bound to. 
- The broker provides the client with its advertised address for the SSL listener
- The client will now connect to the broker using the address provided.

### Configuration

**Broker1**
```
listeners: SSL://0.0.0.0:9091
advertised.listeners: SSL://broker1.kafka.example.com:9091
```
**Broker2**
```
listeners: SSL://0.0.0.0:9091
advertised.listeners: SSL://broker2.kafka.example.com:9091
```
**Broker3**
```
listeners: SSL://0.0.0.0:9091
advertised.listeners: SSL://broker3.kafka.example.com:9091
```
### Topology
![Advertised listener scenario1](/assets/images/kafka-listeners-scenario1.svg)


### Client Connection Process
![Advertised listener client connection](/assets/images/kafka-listeners-uml-scenario1.svg)


## Scenario 2: Private Broker Network

This scenario builds from the first scenario by adding an private network for inter-broker communications. The listeners are now named to make their purpose clear.

Both the CLIENT and BROKER listeners are listening for SSL connections on port 9091 but they are bound to dedicated interfaces.

### Configuration

**Broker1**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.11:9091, BROKER://172.20.10.11:9091
advertised.listeners: CLIENT://broker1.kafka.example.com:9091 BROKER://172.20.10.11:9091
```
**Broker2**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.12:9091, BROKER://172.20.10.12:9091
advertised.listeners: CLIENT://broker2.kafka.example.com:9091, BROKER://172.20.10.12:9091
```
**Broker3**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.13:9091, BROKER://172.20.10.13:9091
advertised.listeners: CLIENT://broker3.kafka.example.com:9091, BROKER://172.20.10.13:9091
```

### Topology
![Advertised listener scenario2](/assets/images/kafka-listeners-scenario2.svg)


## Scenario 3: External Clients

Building on from scenario 2 this scenario adds clients in a third network segment which adds a few constraints into our topology design.
- The clients cannot use the domain `kafka.example.com` to reach the cluster. 
- The clients will use the domain `kafka.external.com`, which cannot be resolved by the Kafka brokers 
- The clients are not allowed to directly communicate with the brokers, security devices must be in line.

The topology diagram does not show specific security devices for this scenario as doing so just made everything messy.

The external load balancer has 4 virtual servers, VS 1, VS 2 and VS 4; each of these virtual servers is configured to forward traffic to a specific broker. Virtual server VS 3 is used to perform the initial connection from the client to the broker.

The client uses the address of VS3 for the initial connection; afterwards the client will use the advertised address provided by the broker. The advertised address will direct client traffic to the virtual server for that broker.


### Configuration

**Broker1**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.11:9091, BROKER://172.20.10.11:9091, EXTERNAL://192.168.1.11:9092
advertised.listeners: CLIENT://broker1.kafka.example.com:9091 BROKER://172.20.10.11:9091, EXTERNAL://broker1.kafka.external.com
```
**Broker2**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.12:9091, BROKER://172.20.10.12:9091, EXTERNAL://192.168.1.12:9092
advertised.listeners: CLIENT://broker2.kafka.example.com:9091, BROKER://172.20.10.12:9091, EXTERNAL://broker2.kafka.external.com
```
**Broker3**
```
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://192.168.1.13:9091, BROKER://172.20.10.13:9091, EXTERNAL://192.168.1.13:9092
advertised.listeners: CLIENT://broker3.kafka.example.com:9091, BROKER://172.20.10.13:9091, EXTERNAL://broker3.kafka.external.com
```

### Topology
![Advertised listener scenario2](/assets/images/kafka-listeners-scenario3.svg)

### External Client Connection
![Advertised listener client connection](/assets/images/kafka-listeners-uml-scenario3.svg)

## Scenario 4: Kubernetes Internal Clients

This scenario looks client communication where Kafka and the client are both running on the same Kubernetes cluster.

The Kubernetes service resource is of clusterIP type and `.spec.clusterIP` has no value supplied making this a [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services).

The brokers are deployed to a Kubernetes namespace called `kafka` and the cluster domain is `cluster.local`

### Configuration
**Service**
```
# Kubernetes Settings
Service Name: kafka
```
**Broker1**
```
# Kubernetes Settings
Pod name: kafka-broker-0
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092
advertised.listeners: CLIENT://kafka-broker-0.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-0:9092
```
**Broker2**
```
# Kubernetes Settings
Pod name: kafka-broker-1
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092 
advertised.listeners: CLIENT://kafka-broker-1.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-1:9092
```
**Broker3**
```
# Kubernetes Settings
Pod name: kafka-broker-2
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092 
advertised.listeners: CLIENT://kafka-broker-2.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-2:9092
```

Using the services FQDN ensures that pods from all namespaces can resolve the service hostname. The pod running the client receives the address `kafka-broker-0.kafka.svc.kafka.cluster.local` which resolves to the broker pods IP address.

### Topology
![Advertised listener scenario2](/assets/images/kafka-listeners-scenario3.svg)

## Scenario 5: Kubernetes External Clients Using Routes

This scenario looks the listener configuration for using `route.openshift.io/v1` resources to allow external clients to reach the Kafka cluster running on Kubernetes.

The advertised port address for the EXTERNAL listener is different to the port which the listener is bound to. The advertised listener address, including port is relevant only to the client and translation may occur allow the data path between the client and listener.

Info about the route resource can be found [here](https://docs.openshift.com/container-platform/4.10/rest_api/network_apis/route-route-openshift-io-v1.html).

We are going to configure one service for initial load balancing and then one service and one route per broker.

### Configuration
**Initial balancing Service and Route**
```
Service Name: kafka-svc
Route Name: kafka-route
```
**Broker1**
```
# Kubernetes Settings
Pod name: kafka-broker-0
Service name: kafka-broker-0-svc
Route name: kafka-broker-0-route
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092, EXTERNAL://:9093 
advertised.listeners: CLIENT://kafka-broker-0.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-0:9092, EXTERNAL://kafka-broker-0.apps.ocp4.example.com:443
```
**Broker2**
```
# Kubernetes Settings
Pod name: kafka-broker-1
Service name: kafka-broker-1-svc
Route name: kafka-broker-1-route
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092, EXTERNAL://:9093
advertised.listeners: CLIENT://kafka-broker-1.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-1:9092, EXTERNAL://kafka-broker-1.apps.ocp4.example.com:443
```
**Broker3**
```
# Kubernetes Settings
Pod name: kafka-broker-2
Service name: kafka-broker-2-svc
Route name: kafka-broker-2-route
# Kafka settings
lister.security.protocol.map: CLIENT:SSL, BROKER:SSL, EXTERNAL:SSL
inter.broker.listener.name: BROKER
listeners: CLIENT://:9091, BROKER://:9092, EXTERNAL://:9093
advertised.listeners: CLIENT://kafka-broker-2.kafka.svc.kafka.cluster.local:9091 BROKER://kafka-broker-2:9092, EXTERNAL://kafka-broker-2.apps.ocp4.example.com:443
```

Using the services FQDN ensures that pods from all namespaces can resolve the service hostname. The pod running the client receives the address `kafka-broker-0.kafka.svc.kafka.cluster.local` which resolves to the broker pods IP address.

### Topology
![Advertised listener scenario2](/assets/images/kafka-listeners-scenario5.svg)

### External Client Connection
![Advertised listener client connection](/assets/images/kafka-listeners-uml-scenario5.svg)

## Summary
Configuring listeners and advertised listeners can take a bit of time to get your head around, particularly the difference between the settings. Gaining an understanding of these configuration options will help simplify designing a Kafka architecture for client connectivity.

Page logo image source: https://commons.wikimedia.org/wiki/File:Apache_kafka.svg