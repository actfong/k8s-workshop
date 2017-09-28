## Part 5: Deployments


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

Congratulations, you have just deployed with Kubernetes! 

Since we already setup a `Service` object with selectors that match the labels of our Pods, you should be able to access the application from your browser. 

On the main page, you should see that *Abel is travelling from Batavia to Mauritius* and there is a link for you to check the current version of the app.

A note about the manifest file: Within a `Deployment`'s manifest, the `labels` within a Pod's template will be automatically applied to its surrounding `Deployment` and `RS` as `selectors`. Whereas in a manifest for a `RC`, where we need to ensure that RC's `selectors` match the `labels` within a Pod's template. 

Now inspect your `Deployment`, `ReplicaSet` and `Pod` with `kubectl get` and `kubectl describe` and pay attention to the following:

- Your `Deployment` object, will have `Selectors` which were supplied to your Pod template as `labels`
- Your RS's names are prefixed with the name of your Deployment. You will also see that within your RS, the values for `Replicas`, `Labels` and `Selectors` are same as specified in the Deployment's manifest.


### Rolling Update ###

Now you have seen how to deploy, how about updating your app?

When we deploy a newer version of our app, that means we need to deploy a newer version of our application's image.
So, this is simply a matter of you changing the image within the specs of your container.

In our case, let's change it to `image: actfong/tasman:2.0` in our deployment's manifest.


But before you roll out a new version, I want you to pay attention to a few more things.

When you deploy for the first time, K8s created a `ReplicaSet` for you to hold your pods.
When you deploy an update, it will use (or create) another `ReplicaSet` to control the new pods. Then, while it tears down the old Pods in the first ReplicaSet, it spins up new pods in the new ReplicaSet

The following section in the manifest is related to this mechanism:

```
  minReadySeconds : 10                # wait 10 sec after spinning up before moving on to next pods
  strategy:
    type: RollingUpdate               # Default value is RollingUpdate
    rollingUpdate:
      maxUnavailable: 1               # 1 pod down at a time
      maxSurge: 1                     # Never have 1 pod more than the specifed replicas-amount
```

The `RollingUpdate` [strategy](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy) means that the old pods get replaced gradually by the new pods. 

The `maxUnavailable` and `maxSurge` parameters define how much below or above the specified replicas number we can go during the deploy. With `minReadySeconds`, we specify how long a Pod should have been running before we move on with spinning up the next one.

The `kubectl apply` command can be used to update K8s resources, including `Deployment`. But this time, when you deploy this update, please add the `--record` flag. I will explain it to you later.

```
kubectl apply -f {manifest-file.yml} --record
```

If you want to know the status during the rollout, you can check with:
```
kubectl rollout status deployment {name-deployment}
```

To verify that you have indeed deployed the updated version: check the app through your browser again: you should see that where Abel is travelling from/to has changed (from Mauritius to Van Diemens's Land). Same goes for the link showing you the current version (2). 

Also, could you list and inspect the ReplicaSets with `kubectl get` and `kubectl describe`? 

You should now see two RS's and that the old RS has no pods within it. When you inspect the new pods, you should also see that its container has the new image.


### Rollbacks and History ###

K8s keeps track of revisions of your `Deployment` objects, which you can see with:
```
kubectl rollout history deployment/{deploy-name}
```

In our case, we should see 2 revisions. You should also see that the second revision has a value for the `CHANGE-CAUSE`. This is because with your last deployment, the `--record` flag was added.

These revision also shows us the attributes of our deployment, such as which image was deployed. You can inspect that with:

```
kubectl rollout history deployment/{deploy-name} --revision={revision-number}
```

These revisions in K8s allow us to perform a rollback easily. Not only to previous version, but also further in the past.

Rollback can be performed with

```
kubectl rollout undo deployment/{deploy-name}                               # rollback by 1 revision
kubectl rollout undo deployment/{deploy-name} --revision={revision-number}  # rollback to a specific revision
```


---
