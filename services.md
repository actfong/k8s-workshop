## Part 2: Services

### Concept ###

As you have seen in the previous section, even though you have deployed a Pod, that doesn't mean you can access it easily.

Actually, there is a reason for that: Pods are **ephemeral** in nature. So you shouldn't try to access a pod through it's individual IP. That is when a `Service` object comes in handy.

A `Service` object has it's own IP, DNS and Port and they **never** change. It enables you to access pods through the mechanism of `selectors` and `labels`.

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-service.png?raw=true" width="800" height="700"/>

#### Service Types ####
There are several [types of Services](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types). The default value is `clusterIP`, which only exposes your Service internally.

In our example, since we want to access our application from outside the cluster, we will go for the type `NodePort`. It exposes the Service object on the `Node`'s IP at a specific port. (Reminder: Node is a minion VM).

If you are running your cluster on the cloud, consider the type `LoadBalancer`, which exposes your Service on your cloud provider's load-balancer. However, how the type `LoadBalancer` works varies with each cloud provider.

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-service-types.png?raw=true" width="550" height="375"/>

### Manifest ###

Below is an example of a NodePort Service: 

It is made accessible on port 8080, forwards requests to Pod's 4567.

The `IP` of this Service object is auto-generated and `nodePort` will be automatically assgined to a port in the range of 30000-32767, unless we specify one ourselves.


```yml
# service-example.yml
apiVersion: v1
kind: Service
metadata:
  name: tasman                         # name of your Service
spec:                                  # defines what is in this resource
  type: NodePort                       # ClusterIP by default (which is only accessible internally)
  ports:
  - port: 8080                         # Allow Service access at this port
    targetPort: 4567                   # Let your Service target a specific port of your Pods
    # nodePort: 30001                  # Expose the Service object to the external world on a port of Node's IP (30000-32767)
    # protocol: TCP                    # TCP by default
  selector:
    app: tasman                        # targets pods with label app=tasman
```

A few attributes to pay special attention to:

- `spec.selector` allows you to target pods with specific `labels`.
- `spec.ports.Port` allows you to set the port number where your Service object will be running.
- `spec.ports.targetPort` allow us to select which Port to target within the Pods. In our example, since we know that our Sinatra-powered application runs on port 4567 within our Pod, we can set the targetPort to 4567.
- `spec.ports.nodePort` opens a port on the `Node` and forward requests from {nodeIp}:{nodePort} to {clusterIp}:{port}. The `nodePort` option is available as we have chosen the type `NodePort`.


**Note**: By default the `targetPort` will be set to the same value as the `port` field.

Example: If your application is running on port 4567, you could just specify:
```
port: 4567
# no need to specify targetPort
```
And this will expose your service on 4567 and forward requests to Pod’s 4567 automatically.


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
