# AMQ Broker v7 and Interconnect v1 OpenShift Demo

This repository includes demonstrations of various AMQP messaging scenarios and capabilities implemented with open source technologies including Apache ActiveMQ Artemis, Apache Qpid Dispatch Router, cert-manager, Prometheus, and of course Kubernetes (this demo targets OpenShift v4.x).

Capabilities demonstrated include:

- Operator-managed AMQ Broker and Interconnect router instances
- TLS Certificate management using cert-manager
- Link routing client connections to single-broker instances
- Message routing using Auto Links for broker sharding
- Message producer and consumer configuration
- Broker and router metrics monitoring using Prometheus and charting using Grafana
- Custom init image for advanced broker configuration

## Messaging Patterns and Platform Topologies

This repository contains several top-level directories which correspond to namespaces in an OpenShift cluster. Nearly all files contained in these directories are a mix of native OpenShift/Kubernetes object instances or `CustomResource` instances. File names indicate the object type and instance name. This repository structure could provide a starting point for GitOps-oriented management of a messaging environment.

The following describes each individual namespace and its purpose.

### Simple Broker Deployment

The `amq-broker-simple` example deployment demonstrates a single-instance broker. When message order must be perfectly preserved (strictly FIFO), a single-broker instance may be a good choice. Durable topic subscriptions are not compatible with standard Artemis clustering in Kubernetes, so single-instance broker deployments may be useful when clients need durable subscriptions.

This deployment exposes secure (SSL/TLS) broker acceptors and leverages cert-manager to request certificates.

### Sharded Broker Deployment

The `amq-broker-sharded` example deployment demonstrates 3 identical, but independent broker instances. While these broker instances share the same configuration, they are not networked (clustered).

This broker deployment scheme would require significant client coordination by itself, but with the help of an AMQP router, clients can produce and consume from a single endpoint while messages are load-balanced across broker instances.

This deployment exposes secure (SSL/TLS) broker acceptors and leverages cert-manager to request certificates.

### AMQP Router Deployment

The `amq-router` example deployment shows an Interconnect router that connects to the first two example brokers `amq-broker-simple` as well as `amq-broker-sharded`. The router makes two different types of links to the brokers.

The router's connection to the simple broker is a `LinkRoute`. Any client that produces to or consumes from an address starting with the prefix `simple.` is directly connected to the simple broker. This direct connection between the client and broker can support transactions and durable topic subscriptions.

The router's connection to the sharded broker is accomplished with `Auto Links` and a `waypoint`. When clients produce messages to the address `sharded.test` the messages will be balanced across the 3 broker instances and stored in their respective journals. When clients consume messages from `sharded.test` they will receive an aggregated stream of messages sourced from all 3 brokers. This routing strategy can support a fast and horizontally scalable messaging platform, but it cannot support strict in-order message delivery (FIFO).

### Custom Broker Deployment

The `amq-broker-custom` example demonstrates the technique for directly configuring native ActiveMQ Artemis configuration files. The standard configurations supported by AMQ Broker CustomResources may not be sufficient for some edge cases. It is possible to build a custom init image to provide any of the following configuration files:

- `artemis-roles.properties`
- `artemis.profile`
- `broker.xml`
- `jolokia-access.xml`
- `login.config`
- `artemis-users.properties`
- `bootstrap.xml`
- `jgroups-ping.xml`
- `logging.properties`
- `management.xml`

Only `logging.properties` is customized in this example, but the pattern applies to any of the above configuration files.

This example does not demonstrate secure (SSL/TLS) acceptors.

### Client Application Deployment

The `amq-client` deployment example demostrates a simple Spring Boot service that uses the Camel framework to produce and consumer messages over AMQP. The producer and consumer are configurable with a ConfigMap that contains the application properties. By default, the application produces messages once per second and consumes the same messages.

Note that a certicate is requested for the client application, however the application only makes use of the `truststore.jks` data so that it can verify router and broker certificates.  

## Provisioning Instructions

Use the following guide to provision the example scenarios.

### Provision Global Operators

This set of configuration examples makes use of two operators installed with cluster-wide scope.
- Cert-manager: for issuing TLS certificates and keystores
- AMQ Broker: for managing Artemis broker instances

Install both operators into the `openshift-operators` namespace via OLM, or apply the following commands:

```
oc apply -f openshift-operators/operator -n openshift-operators
```

### Provision a ClusterIssuer to provide AMQ Certificates

Create a namespace for certificate management.

The `openshift-operators` namespace will contain the root CA certificate for the messaging environment. A `ClusterIssuer` will use that root CA to sign certificates issued for the messaging platform.

```
oc apply -f openshift-operators/ -n openshift-operators
```

### Provision Broker Example

The following example commands use `amq-broker-simple` configuration.

```
oc new-project amq-broker-simple
oc apply -f amq-broker-simple/ -n amq-broker-simple
```

Repeat this procedure to provision other broker examples.

### Provision Router

Provision the Interconnect operator and example router deployment.

```
oc new-project amq-router
oc apply -f amq-router/operator/ -n amq-router
oc apply -f amq-router/ -n amq-router
```

### Provision Fuse Client Application

Provision the example AMQP producer and consumer application.

```
oc apply -f amq-certificates/client/ -n amq-certificates
oc new-project amq-client
oc apply -f amq-client/ -n amq-client
```

The included BuildConfig will start its first build as soon as it is provisioned. The DeploymentConfig will trigger its first rollout as soon as the build is finished publishing the resulting image.

View the application logs of the example application to observe the producer and consumer perform a round-trip brokered message exchange. Adjust the application properties ConfigMap to configure the producer and consumer and exercise the various broker and router configurations and topologies.

### Provision Monitoring Components

Provision the Prometheus monitoring environment.

```
oc new-project amq-monitor
oc apply -f amq-monitor/operator/ -n amq-monitor
oc apply -f amq-monitor/ -n amq-monitor
```
