# k8s-workshop


## Prerequisites
1. Watch this video:

[![Watch this video](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQmRR8k2nuUFA25p5M0NIAGPzpt_yNSduQ5gf7y7MM1LKb7jIBXqw)](https://youtu.be/PH-2FfFD2PU)

2. [Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

## Part 1: Hello World

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
