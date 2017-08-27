## Part 2: Services

### Concept ###

As you have seen in the previous section, even though you have deployed a Pod, that doesn't mean you can access it easily.

Actually, there is a reason for that: Pods are ephemeral in nature. So you shouldn't try to access a pod through it's individual IP. That is when a Service object comes in handy.

A `Service` object has it's own IP, DNS and Port and they **never** change. It enables you to access pods through `selectors` and `labels`.

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-service.png?raw=true" width="700" height="500"/>

### Manifest ###

Attributes
Selectors => select pods by labels
IP => ip to be accessed within th cluster
Port => is the abstracted Service port, which can be any port other pods use to access the Service
targetPort => is the port the container accepts traffic on
nodePort => opens a port on the node, forward request to nodeIp:port to the clusterIp:port
By default the targetPort will be set to the same value as the port field.

ServiceTypes:
- ClusterIP
- NodePort
- LoadBalancer (cloud provider specific)

Exercise:
Deploy Pods
Deploy Service
Describe Service (attention: endpoints)

Spin up a busybox to access Service object by
  internal IP:port
  internal DNS:port
  external Node:Nodeport
  externalIP?

Traffic that ingresses into the cluster with the external IP (as destination IP), on the service port, will be routed to one of the service endpoints. 
