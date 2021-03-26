# AMQ OpenShift Configuration

## Install Certificate Manager Operator

In Operators|operatorHub, install 'cert-manager' community edition(not marketplace) Cert-manager may take a while to install. Wait for the operator to show in Operator|Installed Operators before proceeding.

## Install Resource Locker Operator

In Operators|operatorHub, install 'Resource Locker Operator'. Resource Locker installs in all namespace and may take a while to install. Wait for the operator to show in Operator|Installed Operators before proceeding.

## Provision AMQ Certificate Management

Create Certificates
```
oc new-project amq-certificates
oc apply -f amq-certificates/ -n amq-certificates
oc apply -f amq-certificates/customization/ -n amq-certificates
```

## Provision Brokers

This example uses `amq-broker-simple` configuration.

```
oc new-project amq-broker-simple
oc apply -f amq-broker-simple/ -n amq-broker-simple
oc apply -f amq-certificates/amq-broker-simple/ -n amq-certificates
oc apply -f amq-broker-simple/customization/ -n amq-broker-simple
```

## Provision Routers

This example uses `amq-router1` configuration.

```
oc new-project amq-router1
oc apply -f amq-router1/ -n amq-router1
oc apply -f amq-certificates/router1/ -n amq-certificates
```

## Provision Monitoring Components

In Operators|operatorHub, install 'Prometheus Operator' community edition(not marketplace) 
Prometheus Operator may take a while to install. Wait for the operator to show in Operator|Installed Operators before proceeding.

In Operators|operatorHub, install 'Grafana Operator' community edition(not marketplace) 
Grafana Operator may take a while to install. Wait for the operator to show in Operator|Installed Operators before proceeding.

```
oc new-project amq-monitor
oc apply -f amq-monitor/ -n amq-monitor
```
