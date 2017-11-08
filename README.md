# Docker meetup k8s topic

this repository contains the resources used for the K8S demo at the Docker meetup.

## The demo

* Schedule a demo deployment
* Use an `HorintalPodAutoscaler`
* Job scheduling with a `Job` resource

### the original scenario

During this demo, I needed:
* 2 kubernetes clusters (cluster1 and cluster2), version 1.8.1
* An nginx load-balancer in front of cluster2.

On cluster2 is running the `demo` deployment. This runs 2 replicas of a dump "demo" 
container:

```
package main

import (
        "io"
        "net/http"
        "os"
)

func helloHandler(w http.ResponseWriter, r *http.Request) {
  hostname, err := os.Hostname()
        if err != nil {
                panic(err)
        }

        io.WriteString(w, hostname)
}

func main() {
        http.HandleFunc("/", helloHandler)
        http.ListenAndServe(":8080", nil)
}
```

On cluster2, we add an HPA that will scale the demo's pods when it's CPU consumption rises above 50%.
For the HPA to work, the heapster addon must be running.

Once cluster2's setup is done and demo's deployment is up and running, We execute the `vegeta` `Job`.
This job will running vegeta containers. Those containers will send some HTTP workload to the 
demo deployment throught the nginx load-balancer. This is supposed to trigger the HPA mechanisms
and a scaling of the demo pods.

The k8s resources in the repository can be scheduled on a single k8s cluster. The frontal
nginx loadbalancer is not required.

### Ingress resources

I decided to use an ingress controller with a nginx instance acting as a load balancer in front
of my kubernetes cluster.

* Create the ingress controller required resources
```
kubectl create -f ingress-controller/
```

* Edit the `ingress.yml` with a FQDN of your own then create the resource
```
kubectl create -f ingress.yml
```

### The demo deployment

```
kubectl create -f demo.yml
kubectl create -f hpa.yml
```

### Schedule the job

```
kubectl create -f job-vegeta.yml
```
