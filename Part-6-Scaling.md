## Part 6: Scaling your application

### Concept

From the previous sections, we have seen that a Pod can be managed by a RC, RS or a Deployment.

`RC` and `RS` have the sole purpose to ensure that a number of Pods are up, while a `Deployment` wraps around `RC`'s/`RS`'s and takes care of rollouts and rollbacks.

However, when the demand for your application grows, these objects (RC, RS and Deployments) are not capable to scale up the number of pods dynamically. For that, you need a `HorizontalPodAutoscaler` (a.k. `hpa`).

### Create a HPA

If you have deployed with the manifests from the previous section, you should have 4 Pods by now. (You can check that by listing your pods with `kubectl get pods` or inspect your RS / Deployment with `kubectl describe`.)

To create a HPA to autoscale a Deployment:

```
# by default, target cpu-utilization is set to 80%
kubectl autoscale deploy/{deploy-name} --max={max-number-of-pods}
kubectl autoscale deploy/{deploy-name} --max={max-number-of-pods}  -min={min-number-of-pods}
kubectl autoscale deploy/{deploy-name} --max={max-number-of-pods} --cpu-percent={target-CPU%}
```

In the examples above, we use a Deployment object as a reference for our HPA. From that moment, the [`controller manager`](https://kubernetes.io/docs/admin/kube-controller-manager/) in the K8s Master will continously observe CPU usage of Pods managed by this Deployment and scales up/down, in order to match the CPU-target that is set in the `HPA` object.

So in case you are not using `Deployments` but rather `RCs`, you should refer to a RC with:

```
kubectl autoscale rc/{rc-name} 
```

After creating your HPA, you can see it in `kubectl get hpa` and find out more with `kubectl describe hpa {hpa-name}`.
When you inspect with `kubectl describe`, please pay attention to the attribtues `Reference` and `Metrics`.

Also, if the mininum number of Pods in our HPA is greater than the initial amount (as specified in our Deployment), you should see that the numbers of Pods now matches the minimum in our HPA.

At last, pay attention to the Replica's attribute in Deployment and RS. 
