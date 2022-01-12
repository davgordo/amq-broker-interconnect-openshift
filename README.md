# AMQ OpenShift Configuration

## Demonstrated Capabilities

This repository includes demonstrations of various AMQP messaging scenarios and capabilities implemented with open source technologies.

- Operator-managed AMQ Broker and Interconnect router instances
- TLS Certificate management using cert-manager
- Link routing client connections to single-broker instances
- Message routing using Auto Links for broker sharding
- Message producer and consumer configuration
- Broker and router metrics monitoring using Prometheus and charting using Grafana
- Custom init image for advanced broker configuration

## Provisioning Instructions

Use the following guide to provision the example scenarios.

### Provision Global Operators

This set of configuration examples makes use of two operators installed with cluster-wide scope.
- Cert-manager: for issuing TLS certificates and keystores
- Resource Locker: for copying certificate secrets and translating secret keys when necessary

Install both operators into the `openshift-operators` namespace via OLM, or apply the following commands:

```
oc apply -f openshift-operators/operator -n openshift-operators
```

### Provision Certificate Management

Create a namespace for certificate management.

The `amq-certificates` namespace will contain the root CA certificate for the messaging environment. This is also where certificates will be issued by the cert-manager operator. Another operator, Resource Locker, is responsible for copying certificate secrets into the target namespaces and translating the keys to match the naming that the consuming workloads expect.

```
oc new-project amq-certificates
oc apply -f amq-certificates/ -n amq-certificates
```

### Provision Broker Example

The following example commands use `amq-broker-simple` configuration. If the example requires certificates, apply the corresponding sub-directory of resources under the `amq-certificates` directory.

```
oc apply -f amq-certificates/broker-simple/ -n amq-certificates
oc new-project amq-broker-simple
oc apply -f amq-broker-simple/operator/ -n amq-broker-simple
oc apply -f amq-broker-simple/ -n amq-broker-simple
```

### Provision Router

Provision the Interconnect operator and example router deployment.

```
oc apply -f amq-certificates/router/ -n amq-certificates
oc new-project amq-router
oc apply -f amq-router/operator/ -n amq-router
oc apply -f amq-router/ -n amq-router
```

### Provision Monitoring Components

Provision the Prometheus monitoring environment.

```
oc new-project amq-monitor
oc apply -f amq-monitor/operator/ -n amq-monitor
oc apply -f amq-monitor/ -n amq-monitor
```

When monitoring cert-manager, add a role binding to grant permissions to the service monitor.

```
oc apply -f openshift-operators/ -n openshift-operators
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