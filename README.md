# k8s-workshop

## GOALS:

At the end of the workshop, you should be able to:

- Understand what a **Pod** is and how to interact with it
- Ensure that a number of pods are running with **ReplicaSets / Replication Controllers**
- Perform rolling updates and rollbacks with **Deployments**
- Expose your application to the outside world with **Services**

## Prerequisites
1. Watch this video:

[![Watch this video](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQmRR8k2nuUFA25p5M0NIAGPzpt_yNSduQ5gf7y7MM1LKb7jIBXqw)](https://youtu.be/PH-2FfFD2PU)

2. [Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

## Part 1: Hello World
For our Hello World exercise, we will deploy an app containing a bunch of nginx containers.

GOALS:
- Able to create, inspect and delete K8s resources (Pods ReplicaSets Deployments Services)
- Verify that your application is running (index.html and 50x.html get served)

### Check your setup
Before we start, ensure that your `minikube` is running:
```
minikube status
```

If it isn't started yet, run `minikube start` to start a VM containing your K8s-cluster

Then let's verified that `kubectl` is wired to minikube.
```
kubectl config current-context
```

If not, run:

```
kubectl config use-context minikube
# This is comparable to docker-machine's `eval "$(docker-machine env {machine-name})"`, 
# ensuring that your client is wired to the right environment.
```

Now that everything is setup, let's deploy! :)

### Deploy our app 
```
# create a Deployment, containing a ReplicaSet (of 3 pods), each of them containing a nginx container
kubectl run --image=nginx hello-nginx --port=80 --labels="app=hello" --replicas=3

# expose your Pods within the hello-nginx Deployment outside the cluster.
kubectl expose deployment hello-nginx --port=80 --name=hello-http --type=NodePort

# grab the NodePort number (30000-32767)
kubectl describe svc hello-http 

# Access Pods through your cluster's ip (which is minikube)
wget $(minikube ip):{nodePort}
```

Congrats! You just deployed an app on Kubernetes!

The approach that we took to deploy is known as the **imperative** way. (*imperative vs declarative*: more on that later)

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

### What you have learned in this section
- Wire your `kubectl` client to the correct Kubernetes cluster
- Inspect and delete various Kubernetes resources (`deployment`, `replicasets`, `pods` and `services`) with `kubectl get` and `kubectl describe`

### Questions to ask yourself
- In the *Deploy our app* section, we deployed an app by running and exposed it by some commands (`run` and `expose`). And as mentioned, this way is known as the **imperative** way. 

Now imagine if you were to deploy with more configurations (such as replica numbers, multiple pods etc), how would your command look like? 

---

### What you have learned in this section

1. Familiarize yourself with **minikube**
2. Familiarize with the key resources within Kubernetes (Pods, ReplicaSets, Deployments, and Services)
3. Learned how to interact with K8s resources by kubectl `get`, `describe` and `delete`
4. Quickly deployed all these resources in one go with `kubectl run`, which is the `imperative way` of deployment.

This was a very high-level overview of Kubernetes. From here on, we will build your knowledge from the bottom up, starting by looking at [**Pods in the next section**](https://actfong.github.io/k8s-workshop/Part-2-PODS)
