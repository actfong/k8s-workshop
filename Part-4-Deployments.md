## Part 4: Deployments


### Concept ###

While ReplicationControllers are great for taking care of the desired state for the number of replica's, they aren't really great at performing rolling-updates or rollbacks.

As you can read from the [this section from the K8s documentation](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#rolling-updates), with RC's, in order to deploy a new version of your app as a rolling update, you would have to:

1. create a new RC 
2. set the initial state of the new RC to 1 replica
3. then gradually scale the new RC up by 1 and scale the old RC down by 1. Until there are no more pods left in the old RC...

To avoid all this manual work on the client side, K8s now comes with `Deployments` (still beta), to take care of rolling-updates and rollbacks.

`Deployment` is a wrapper around a `ReplicaSets`, which on its turn wraps around `Pods`. 

<img src="https://github.com/actfong/k8s-workshop/blob/master/k8s-deployment.png?raw=true" width="900" height="500"/>

The `Deployment` object enforces the desired state and makes sure that the change towards this desired state happens at a controlled rate.
Example: 

```
My application currently has 5 Pod replica's. 
Update my application's image from foobar:1.0 to foobar:1.1. 
While replacing old pods during a rolling-update, 
ensure that the number of pods does not exceed / fall below the specified replica's amount by 1.
(In other words, there will be no more than 6 nor less than 4 at any given time)
And ensure there is a period of 3 second between rolling out a new pod.
```


