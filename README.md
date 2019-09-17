# a very quick intro to prometheus

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [intro to the intro](#intro-to-the-intro)
- [what's this thing?](#whats-this-thing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## intro to the intro

Some time ago, we stood up `hush-house`, an environment where we run Concourse
on top of a Kubernetes cluster so that 

- we could see where our Helm chart would break, and
- empathize with Alanas moving to k8s-environments

Naturally, that meant we'd need to have a way of observing the system somehow,
in a way that we could:

- know when things were broken and how it evolved to that state
- whether action would be needed soon


```


                  .--> how can I tell this is ok?
                 /
                 /
                 /
   k8s cluster................................
   .             /
   .             /
   .   concourse deployment --------------
   .   |
   .   |   web
   .   |     
   .   |
   .   |   workers
   .   |
   .   |
   .   |   postgres
   .   |


```



Traditionally, we've been using a combination of InfluxDB as our timeseries
database of choice, paired with Grafana, for displaying the metrics sent to
Influx on dashboards.

While that works just fine, it turns out that many people in the community use
Prometheus rather than InfluxDB when it comes to Kubernetes deployments.

Fortuantelly, Concourse has already been exposing metrics in the Prometheus
exposition format:


```

    Metric Emitter (Datadog):
          --datadog-agent-host=              
          ...

    Metric Emitter (InfluxDB):
          --influxdb-url=                    
          ...

    Metric Emitter (NewRelic):
          --newrelic-account-id=             
          ...

    Metric Emitter (Prometheus):      <<<<<<<
          --prometheus-bind-ip=              
          ...

    Metric Emitter (Riemann):
          --riemann-host=                    
          ...

```


As we were all about empathizing with the Kubernetes ecosystem, it made a lot of
sense to then leverage that stack.


## what's this thing?

Prometheus is a 


