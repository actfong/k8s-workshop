## Part 3: ReplicaSets


### Concept ###

As you have seen in the previous section, a `Pod` is pretty useless on its own. This is also why we would typically deploy Pods within a higher level construct (a "wrapper") when we deploy them.

A `ReplicationSet` is such a wrapper around the Pod. Its purpose is to ensure that a specified number of pod replicas are running. (Hence enforcing the desired state)

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-rs.png?raw=true" width="700" height="500"></img>

When composing a manifest for your `ReplicaSet`, you are required to supply the [template for your `Pod`](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#pod-template). You'd typically also define [the number of replicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#multiple-replicas) for your pods in the ReplicaSet's manifest. 

Once you feed the manifest to the API-server, apart from ensuring that the ReplicaSet and its Pods are deployed, it also starts a background loop, which continuously checks that the actual state matches your the desired.

### ReplicationController and ReplicaSets ###
You might have seen the term `ReplicationController` (a.k.a `rc`) and `ReplicaSet` (a.k.a. `rs` ) before and you might wonder: "*What's the difference?*" 

ReplicaSet is the new way of taking care of replicas, available from apiVersion `extensions/v1beta1`, while RC's are available in `v1`.
The main difference between them is how they support `selector` (more on that later).
