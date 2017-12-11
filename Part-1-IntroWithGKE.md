# k8s-workshop

## GOALS:

At the end of the workshop, you should be able to:

- Understand what a **Pod** is and how to interact with it
- Ensure that a number of pods are running with **ReplicaSets / Replication Controllers**
- Perform rolling updates and rollbacks with **Deployments**
- Expose your application to the outside world with **Services**

## Prerequisites

#### 1 Watch this video

<div>
  <a href="https://youtu.be/PH-2FfFD2PU">
    <img src="https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQmRR8k2nuUFA25p5M0NIAGPzpt_yNSduQ5gf7y7MM1LKb7jIBXqw" alt="high level overview" >
  </a>
</div>

#### 2 Create a Cluster on Google Kubernetes Engine (GKE)
```
# Quick and easy
gcloud container clusters create {cluster-name} --project {google-cloud-project-id} --machine-type g1-small

# By default:
# --num-nodes "3"  
# --machine-type "g1-small"
# --machine-type "n1-standard-1"

# Wire you kubectl client into the cluster, by updating config with right credentials
gcloud container clusters get-credentials {cluster-name}
```
[More Info on GKE](https://cloud.google.com/container-engine/docs/clusters/operations)

## Part 1: Hello World
For our Hello World exercise, we will deploy an app containing a bunch of nginx containers.

GOALS:
- Able to create, inspect and delete K8s resources (Pods ReplicaSets Deployments Services)
- Verify that your application is running (index.html and 50x.html get served)

### Check your setup
If you followed the instructions correctly to create a container cluster, your `kubectl` should be wired into your GKE cluster by now. You can verify that with:
```
kubectl config current-context
```

The output should look something like this:
```
gke_{project_name}_{zone}_{cluster_name}
```

If not, run `gcloud container clusters get-credentials` with the correct arguments, which you can get from your Google Cloud Console.

Now that everything is setup, let's deploy! :)

### Deploy our app
```
# create a Deployment, containing a ReplicaSet (of 3 pods), each of them containing a nginx container
kubectl run --image=nginx hello-nginx --port=80 --labels="app=hello" --replicas=3

# expose your Pods within the hello-nginx Deployment outside the cluster.
kubectl expose deployment hello-nginx --port=80 --name=hello-http --type=LoadBalancer

# grab the "LoadBalancer Ingress"
kubectl describe svc hello-http

# or if you have "watch" installed
watch -d "kubectl describe svc hello-http"

# If you look at the events at the bottom, 
# one of the messages would be "Creating load balancer"
#
# Once you see the message "Created load balancer", 
# you can proceed by accessing your app through the LoadBalancer Ingress address above
```

Congrats! You just deployed an app on Kubernetes!

The approach that we took to deploy is known as the **imperative** way. (*imperative vs declarative*: more on that later)

By now, you should have created the following resources:
- 3 Pods (a.k.a. `po`)
- 1 ReplicaSet (a.k.a. `rs`)
- 1 Deployment (a.k.a. `deploy`)
- 1 Service (a.k.a. `svc`)

Please take a look at the resources (`deployment`, `replicasets`, `pods` and `services`) you just deployed with the following commands:

```
kubectl get {resource-type}                             # e.g. kubectl get deployment
kubectl describe {resource-type} {resource-name}        # e.g. kubectl describe deployment hello-nginx
```

Once you have familiarize with the attributes of the resources, please remove them all by:
```
kubectl delete deployment hello-nginx
kubectl delete svc hello-http
```

### Questions to ask yourself
In the *Deploy our app* section, we deployed an app and exposed it by running some commands (`run` and `expose`). And as mentioned, this way is known as the **imperative** way.

- Now imagine if you were to deploy with more configurations (such as more labels, mount volumes or even multiple pods etc), how would your command look like?

---

### What you have learned in this section

1. How to create a container-cluster on Google Kubernetes Engine (a.k.a. **GKE**)
2. Wire your `kubectl` client to the correct GKE cluster
3. Familiarize with the key resources within Kubernetes (Pods, ReplicaSets, Deployments, and Services)
4. Learned how to interact with K8s resources by kubectl `get`, `describe` and `delete`
5. Quickly deployed all these resources in one go with `kubectl run`, which is the `imperative way` of deployment.

This was a very high-level overview of Kubernetes. From here on, we will build your knowledge from the bottom up, starting by looking at [**Pods in the next section**](https://actfong.github.io/k8s-workshop/Part-2-Pods)
