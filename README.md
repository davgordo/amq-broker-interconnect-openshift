# AMQ OpenShift Configuration

## Provision Global Operators

```
oc apply -f openshift-operators/ -n openshift-operators
```

## Provision Certificate Management

```
oc new-project amq-certificates
oc apply -f amq-certificates/operator/ -n amq-certificates
oc apply -f amq-certificates/ -n amq-certificates
oc apply -f amq-certificates/customization/ -n amq-certificates
```

## Provision Brokers

This example uses `amq-broker-simple` configuration.

```
oc apply -f amq-certificates/broker-simple/ -n amq-certificates
oc new-project amq-broker-simple
oc apply -f amq-broker-simple/operator/ -n amq-broker-simple
oc apply -f amq-broker-simple/ -n amq-broker-simple
oc apply -f amq-broker-simple/customization/ -n amq-broker-simple
```

## Provision Router

```
oc apply -f amq-certificates/router/ -n amq-certificates
oc new-project amq-router
oc apply -f amq-router/operator/ -n amq-router
oc apply -f amq-router/ -n amq-router
```

## Provision Monitoring Components

```
oc new-project amq-monitor
oc apply -f amq-monitor/operator/ -n amq-monitor
oc apply -f amq-monitor/ -n amq-monitor
```

## Provision Fuse Client Application

```
oc apply -f amq-certificates/client/ -n amq-certificates
oc new-project amq-client
oc apply -f amq-client/ -n amq-client
```
