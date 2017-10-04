## Part 3: Services

### Concept: Allow access to Pods through Services ###

As you have seen in the previous section, even though you have deployed a Pod, that doesn't mean you can access it easily.

Actually, there is a reason for that: Pods are **ephemeral** in nature. For example, when whenever you deploy a new version, the old Pod gets deleted and the new one will get a new IP. So you shouldn't try to access a pod through it's individual IP. That is when a `Service` object comes in handy.

A `Service` object has it's own IP, DNS and Port and they **never** change. It enables you to access pods through the mechanism of `selectors` and `labels`.

<img src="https://github.com/actfong/k8s-workshop/blob/master/images/k8s-service.png?raw=true" width="800" height="700"/>

#### Service Types [ClusterIP, NodePort, LoadBalancer]
There are several [types of Services](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services---service-types). The default value is `ClusterIP`, which only exposes your Service internally.

A Service of type `NodePort` exposes the Service object on the Node's IP at a specific port, known as the `nodePort` (see manifest below). Reminder: A *Node* is a minion VM.

If you are running your cluster on the cloud, you could consider the type `LoadBalancer`, which exposes your Service on your cloud provider's load-balancer. However, how the type `LoadBalancer` is implemented varies with each cloud provider. 

If you look back at the example we did in [Part 1](https://actfong.github.io/k8s-workshop/Part-1-IntroWithGKE#deploy-our-app), we actually created a service of Type `LoadBalancer`

<img src="https://github.com/actfong/k8s-workshop/blob/master/images/k8s-service-types.png?raw=true" width="550" height="375"/>

### Manifest ###

For our example, we will continue by creating a NodePort Service. 

Once created, a `nodePort` will be automatically assgined to a port in the range of 30000-32767, unless we specify one ourselves.

A key element to pay attention to in our manifest is the `selector`: it has to match the key-value of our Pods in order to reach our Pods.

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
- `spec.ports.targetPort` allow us to select which Port to target within the Pods. In our example, since we know that our Sinatra-powered application runs on port 4567 within our Pod, we set the targetPort to 4567.
- `spec.ports.nodePort` opens a port on the `Node` and forward requests from {nodeIp}:{nodePort} to {service's ip}:{port}. The `nodePort` option is only available when we choose the type `NodePort`. And again, unless we specify a value ourselves, a  `nodePort` will automatically be assgined to a port in the range of 30000-32767,


**Note**: By default the `targetPort` will be set to the same value as the `port` field.

In our case, we make our Service accessible on port 8080 and forwards requests to Pod's 4567.

However, if we don't mind running our Service on port 4567, then we could just specify:
```
port: 4567
# no need to specify targetPort
```
And this will expose our service on 4567 and forward requests to Pod’s 4567 automatically.

With the manifest above, together with the Pod that you have previously spun up, you can now deploy your service object with:
```
kubectl apply -f {service-manifest.yml}
```
Once deployed, inspect the service object with `kubectl describe svc {svc-name}` and pay attention to the field `Endpoints`. Do you know which IPs it refers to?

#### Try it for yourself

As mentioned before, a NodePort exposes our Service on our Node's IP at a specific port (nodePort).

Could you lookup for your Node's IP and the nodePort of our Service? Once you have those values, try access your app. Any luck???

<details>
  <summary>Read me</summary>
  <div>
<br>
Theoretically, it should have worked... But it doesn't, why?<br>

If you were running on a "bare-metal" VM, it could have worked. However, due to GCE's firewall rules, these ports above 30000  are blocked by default.<br>

You could add a firewall rule with `gcloud compute firewall-rules create {name} `, but that's not ideal. 

Just think about the possibility that you might have to expose multiple services... And then for each one you would have to add a firewall rule!</br>

If we have gone for a Service of type `LoadBalancer`, our app was reachable already. No extra work needed<br/>

But instead of that, we could also create an `Ingress` on top of our existing Service. It would give us even more flexibility and everything will be documented in a declarative way.
  </div>
</details>

### Ingress 

An `Ingress` maps incoming requests to Services, based on rules that you set.

<img src="https://github.com/actfong/k8s-workshop/blob/master/images/k8s-ingress.png?raw=true" width="960" height="720"/>

These rules are based on info such as the *path* or *host* from the incoming traffic. Examples can be found [here](https://cloud.google.com/container-engine/docs/tutorials/http-balancer#step_6_optional_serving_multiple_applications_on_a_load_balancer) and [here](https://kubernetes.io/docs/concepts/services-networking/ingress/#updating-an-ingress).

For our purpose, we only need to forward our requests from our Ingress to our existing service called 'tasman'.

```
# ingress-example.yml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: tasman-ing
spec:
  backend:
    serviceName: tasman
    servicePort: 8080
```

And run:
```
kubectl apply -f {manifest-file}
```

Creating an Ingress could take a few minutes. You can see the progress on Google Cloud's `Container Engine -> Discovery & load balancing` or `watch -d "kubectl describe ingress tasman-ing"`

Once the Ingress is created, you can get the address from  `kubectl describe ingress {ingress-name}` and access your application.

---

### What you have learned in this section

Congrats! You have just learned how to make your Pods accessible, from outside and inside the cluster.

Because Pods are ephemeral, they should never be accessed directly through their IP. But rather, they should be accessed through a Service object.

There are different type of Services. We have created a NodePort and accompanied that with an Ingress.

An Ingress forwards (or terminates) incoming requests to services based on these rules. In our case, we didn't set any rules so it forwards all traffic to our Ingress' IP to the mapped Service.

Until now, we have only targeted one single pod with our Service object.

In the next chapter, you will find out how we can ensure that a number of pod replica's are always running with [ReplicationControllers / ReplicaSets](https://actfong.github.io/k8s-workshop/Part-4-RC-and-RS). And the good news: all these replica's will be targeted by our Service object, automagically!
