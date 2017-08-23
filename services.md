
Concept

Service => Object, with its own unique IP, allows you to access its pods through selectors/labels from anywhere in the cluster
IP, DNS, Port => all reliable on a Service (non-ephemeral)

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
