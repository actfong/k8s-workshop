## Part 3: Services

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


With the manifest above, together with the Pod that you have previously spun up, you can now deploy your service object with:
```
kubectl apply -f {service-manifest.yml}
```
Once deployed, inspect the service object with `kubectl describe svc {svc-name}` and pay attention to the field `Endpoints`. Do you know which IPs it refers to?

### Mini Challenge 1 - Access your Service from outside the cluster ###
Since the Service was of the type `NodePort`, you should be able to access your app through your browser (outside of the cluster). Do you know which IP and Port to hit?

<details>
<summary>Possible Solution:</summary>
<br/>
<p>
If you are on Minikube, you would have only 1 Node and it's IP is the same as returned by the command <pre>$(minikube ip)</pre>
The port can be obtained from <i>nodePort</i> as shown in <i>kubectl describe svc {svc-name}</i>.
<br/>
If you are on GKE, you can get Node IP's from your Cloud Console.
</p>
</details>

<br/>

### Mini Challenge 2 - Access your Service from inside the cluster ###
Apart from `IP`, your Service object can also be reached with its DNS from within the cluster.
The hostname for your Service object is the same as the name you have provided in the manifest.

Could you spin up a Busybox to
- Access the Service by internal-IP:port
- Access the Service by DNS:port

<details>
<summary>Possible Solution:</summary>
<br/>
First, get the <i>Name</i>, <i>IP</i> and <i>Port</i> of your Service through <i>kubectl describe svc {svc-name}</i>.<br/> Then:
<pre>
kubectl run -it busybox --image=busybox --restart=Never -- /bin/sh
wget {Service's IP}:{Service's Port}
wget {Service's Name}:{Service's Port}
</pre>
</details>

<br/>

### Mini Challenge 3 - Access your Service with your ClusterIP ###
The Service manifest allows you to specify [externalIPs](https://kubernetes.io/docs/concepts/services-networking/service/#external-ips).

As described by K8s documentation:

```
If there are external IPs that route to one or more cluster nodes,
Kubernetes services can be exposed on those externalIPs.
Traffic that ingresses into the cluster with the external IP (as destination IP),
on the service port, will be routed to one of the service endpoints.
```

If you are running on minikube, this externalIP is actually your `minikube ip`.

Could you make the required adjustments to the manifest, to allow you to access the application with only the IP address from your browser (no port required)?

<details>

<summary>Possible Solution:</summary>
</br>
Obtain the ip through.
<pre>minikube ip</pre>
</br>
The key is now to add this IP as <i>externalIP</i> and make your service accessible on <i>port</i> 80 (so that clients don't have to specify the port)
</br>

<pre>
apiVersion: v1
kind: Service
metadata:
  name: tasman
spec:
  type: NodePort
  ports:
  - port: 80                           # <= expose service to port 80.
    targetPort: 4567
  selector:
    app: tasman
  externalIPs:
    - 192.168.99.100                    # <= my minikube-ip
</pre>

</br>
Apply the change with:
<pre>
kubectl apply -f {service-manifest.yml}
</pre>

</br>
Then access your application the specified externalIP from your browser.
</details>

---

### What you have learned in this section

Congrats! You have just learned how to make your Pods accessible, from outside and inside the cluster.

Because Pods are ephemeral, they should never be accessed directly through their IP. But rather, they should be accessed through a Service object.

Until now, we have only targeted one single pod with our Service object.

In the next chapter, you will find out how we can ensure that a number of pod replica's are always running with [ReplicationControllers / ReplicaSets](https://actfong.github.io/k8s-workshop/Part-4-RC-and-RS). And the good news: all these replica's will be targeted by our Service object, automagically!
