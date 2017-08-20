## Part 3: ReplicationControllers and ReplicaSets


### Concept ###

As you have seen in the previous section, a `Pod` is pretty useless on its own. This is also why we would typically deploy Pods within a higher level construct (a "wrapper") when we deploy them.

`ReplicaSets` and `ReplicationControllers` are such a wrapper around the Pod. Its purpose is to ensure that a specified number of pod replicas are running. (Hence enforcing the desired state)

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-rs.png?raw=true" width="700" height="500"></img>


### ReplicationController and ReplicaSet ###

You might wonder: "*What's the difference between `ReplicationControllers` (a.k.a `rc`) and `ReplicaSets` (a.k.a. `rs` )*"

ReplicaSet is the new way of taking care of replicas, available from apiVersion `extensions/v1beta1`, while RC's are available in `v1`.
The main difference between them is how they support `selectors` (more on that later). Also, a `ReplicaSet` is meant to be wrapped within a *Deployment* (also more on that later).


### Manifest ###

When composing a manifest for your `ReplicaSet` or `ReplicationController`, you are required to supply the [template for your Pod](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#pod-template). You'd typically also define [the number of replicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#multiple-replicas) for your pods in the ReplicaSet's manifest.

Once you feed the manifest to the API-server, apart from ensuring that the ReplicaSet and its Pods are deployed, it also starts a background loop, which continuously checks that the actual state matches your the desired.

Example:

```yml
# rc-example.yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: tasman                    # name of your ReplicationController
spec:                             # define what is in the RC resource
  replicas: 5                     # number of replicas. The sole purpose of an RC
  selector:
    app: tasman                   # manages Pods with app=tasman as label

  template:                       # define your Pod here
    metadata:
      labels:                     # labels of your Pod and RC's selector must match
        app: tasman
    spec:
      containers:
      - name: tasman              # name of your container
        image: actfong/tasman:1.0
        ports:
        - containerPort: 4567

```

One thing that you should pay attention to: `spec.selector` and `spec.template.metadata.labels` **have to match**. If they don't, K8s will raise an error.

From K8s documentation: "*A ReplicationController manages all the pods with labels that match the selector. It does not distinguish between pods that it created or deleted and pods that another person or process created or deleted.*"

Another key element to pay attention to is `spec.template`, where you define your Pod. If you compare to our Pod manifest in the previous section, you will see that the only things omitted here in `spec.template` are `apiVersion` and `kind` (which will always be Pod ofcourse, as this is the sole purpose of a `spec.template`).

Now you can deploy your manifest with `kubectl apply -f {manifest-file.yml}`

### Check your resources ###

Once deployed, you can list and inspect your RC and Pod resources.

Inspect your RC:
```
kubectl get rc
kubectl inspect rc {rc-name}
```
Pay attention the the `Replicas` (where it shows whether the current state matches the desired state) and [`Pods Status`](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)

Inspect your Pod (again)
```
kubectl get pods              # you will see that the Pods are prefixed with the name of your RC
kubectl inspect pod {pod-name}
```

Do you remember looking at the `Controllers` field in the previous section? When as deploy the Pod as a standalone resource, this field was empty. Since you deploy your Pod wrapped in a RC, this field would have your "wrapper" as a value.

What does this mean? Well, if your Pod is controller by another resource (e.g. RC), once you delete the Controller, you will also delete the Pod.

### Try deleting your pods... You will fail! ###

Do you have the utility `watch` installed? If so, watch your ReplicationController doing its job.
If not, install with `brew install watch` (or apt-get, yum whatever)

In one of your Terminal tabs, list the pods
```
watch -d "kubectl get pods"
```

Then in another tab, delete one of the pods created by your rc with
```
kubectl delete pod {pod-name}
```
Switch back to the first tab..... You should see a new pod being created with a new name, prefixed with the RC's name. Also see the age-difference.

Conclusion: ReplicationControllers (and ReplicaSet) are reponsible for ensuring that current the number of running replicas matches the one you desired.

### Mini Challenge ###

In the previous section, we deleted our pods manually. So RC created new ones to match our desired state. But what happens when it's the opposite: we have too many pods.

As stated previously, A ReplicationController manages all the pods with labels that match the selector. It does not distinguish between pods that it created or deleted and pods that another person or process created or deleted.

Could you create extra pods (stand-alone) that matches the label of the RC? And prove to yourself that the RC will kill pods when there are too many?

<details>
<summary>Possible Solution:</summary>

<br/>
To simulate such situation, the key is to make sure that the `labels` of your Pod matches the `selectors` of the ReplicationController. So based on the previous rc-example.yml, you might want to have a Pod like this:

<pre>
# foobar-pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: foobar
  labels:
    app: tasman
spec:
  containers:
  - name: tasman
    image: actfong/tasman:1.0
    ports:
      - containerPort: 4567
</pre>

Then deploy by:
```
kubectl create -f foobar-pod.yml
```

To watch ReplicationController evict pods to keep the desired state, you could again use `watch`
```
watch -d "kubectl get pods"
```

Also, use
```
kubectl describe rc {rc-name}
```
to see the events within your RC. You should see at the bottom that it has evicted a Pod with a specific name.
</details>

---

While RC and RS are great for ensuring that a number of replicas are running, they aren't really meant for rolling-update and rollbacks.

For that, we need `Deployments`, which we will look at in the next section.
