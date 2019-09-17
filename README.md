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

<br />
<br />



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

<br />
<br />


As we were all about empathizing with the Kubernetes ecosystem, it made a lot of
sense to then leverage that stack.


## what's this thing?

Prometheus is a whole stack for monitoring and alerting based around metrics
(timeseries data), aiming at supporting you with tools for making your system
better to observe:

- getting metrics out of the things we care about
- collecting them
- querying


```

                  .----------> expose % of time spent in kernel mode
                  |
                  |       .---> expose number of requests to /api/v1/containers
                  |       |
    LINUX MACHINE.|.......|..............
    |             |       |
    |    NODE_EXPORTER    |
    |                     |
    |                     |
    |    MY SERVICE.......|..............
    |    |                |
    |    |     import "github.com/prometheus/client_golang/prometheus"
    |    |     ...
    |    |
    |    |               


```


<br />
<br />


In practical terms, that means:

- providing you with client libraries that let you instrument your code in a way
  that allows you to easily expose metrics that you care about


```go
// import the library
//
import (
	"github.com/prometheus/client_golang/prometheus"
)



// define the metric you care about
//
var DatabaseQueries = prometheus.NewCounter(
  prometheus.CounterOpts{
    Name: "concourse_db_queries_total",
    Help: "Number of queries performed",
  },
)


// in a function that executes queries ...
//
func (e *countingConn) Query(query string, args ...interface{}) (*sql.Rows, error) {

  // increment the counter
  //
	DatabaseQueries.Inc()

	return e.Conn.Query(query, args...)
}
```


<br />
<br />



- providing a server that ingests those metrics and allows you to query them
  in various forms



```

    prometheus server instance
      
      .
      |
      |   ingestion of timeseries samples 
      |
      .

    service


```

<br />
<br />


- defining standards on how to make those queries, and how to expose those
  metrics (so that the system can be extended).



```

      grafana
       |
       |
       |  queries   (what's per-second rate of db queries being performed
       |               by our concourse web nodes?)
       |
       |
       *-->  prometheus server instance


```

<br />
<br />



---

Something that I really enjoy about it is how it nicely fits around dynamic
environments, where the things you want to observe come and go


