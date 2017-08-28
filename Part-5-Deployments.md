## Part 4: Deployments (Part 1)


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


### Manifest ###

Like the manifest for an RC, for a `Deployment` you are required to supply the template for your Pod.

Besides of that, you'd typically also define:
- the number of [replicas](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#replicas) (which will become an attribute of the wrapped `ReplicaSet`)
- the [strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) for deployment

Example:

```yml
# deployment-example.yml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tasman                        # name of your Deployment
spec:
  replicas: 4
  # Wait x amount of seconds after a new pod comes up, before you mark a pod as ready and move on
  minReadySeconds : 10
  strategy:
    type: RollingUpdate               # Default value is RollingUpdate
    rollingUpdate:
      maxUnavailable: 1               # 1 pod down at a time
      maxSurge: 1                     # Never have 1 pod more than the specifed replicas-amount

  template:                           # define your Pod here
    metadata:
      # the labels here are not only applied to the pods,
      # but also supplied to Deployment and RS as selectors
      labels:
        app: tasman
    spec:
      containers:
      - name: tasman                  # name of your container
        image: actfong/tasman:1.0
        ports:
          - containerPort: 4567
```

Now you can deploy with:
```
kubectl apply -f {manifest-file.yml}
```

Notice whereas in a manifest for a `RC`, where we need to ensure that RC's `selectors` match the `labels` within a Pod's template, with a `Deployment`'s manifest the `labels` within a Pod's template will be automatically applied to the `Deployment` and `RS` objects around it.

Now inspect your `Deployment`, `ReplicaSet` and `Pod` with `kubectl get` and `kubectl describe`.

Pay attention to the following

- Your `Deployment` object, will have `Selectors` which were supplied to your Pod template as `labels`
- Your RS's names are prefixed with the name of your Deployment. You will also see that the `Replicas`, `Labels` and `Selectors` are as specified in the Deployment's manifest

---
