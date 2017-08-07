## Part 2: Pods

### Concept ###
In Docker, the **atomic unit** of scheduling is a *container*. In K8s, the atomic unit of scheduling is a **pod**.

A Pod is actually an abstraction that contains one or more containers. Within a Pod, containers share the same network and storage resources.


<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-pod.png" width="600" height="500"></img>


The containers on a pod are **co-located** and **co-scheduled**

*Co-located*: If a Pod contains multiple containers, all its containers will run on the same Node (VM that acts as a minion) instead of spread over multiple Nodes.
        
*Co-scheduled*: A Pod's status is only `running` when all its containers are up and running. 


Like all resources in K8s, a Pod can be defined in a manifest file

### Manifest ###

As mentioned in part 1, the way we deployed by running `kubectl run` is known as the *imperative way*.
The imperative way has a few drawbacks:

- If you want to provide more customizations, the command becomes too long
- You can't keep track of changes with version control.
- The imperative way doesn't express the relationship between Pods / Replicas / Deployments 

Therefore, in general, the *declarative way* is the preferred way to work with K8s.

Example:

```yml
apiVersion: v1
kind: Pod                                   # RC, Services and Deployments etc
metadata:                                   
  name: sinatra-skeleton
  labels:                                   # labels; to be used by "selectors"
    zone: staging
    version: v1
spec:                                       # defines what is in the resource
  containers:
  - name: sinatra-skeleton
    image: actfong/sinatra-skeleton:0.1
    ports:
      - containerPort: 4567
```

