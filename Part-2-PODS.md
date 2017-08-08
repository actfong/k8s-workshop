## Part 2: Pods

### Concept ###
In Docker, the **atomic unit** of scheduling is a *container*. In K8s, the atomic unit of scheduling is a *pod*.

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
kind: Pod                                   # K8s ResourceType
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

Once you have saved the config above into a file, you can run

```
kubectl create -f {your-file.yml}               # creates a kubectl resource
# or 
kubectl apply -f {your-file.yml}                # update/create a kubectl resource
# where -f indicates the path to the manifest file
```

In our example above, since the `kind` defined was a *Pod*, the `kubectl create` command will create a *Pod*. 
If the `kind` was, let's say, a `ReplicationController`, then a `ReplicationController` will be created by `kubectl create`.

To prove that you have indeed created a pod, you can inspect it by:

```
kubectl get pods                                # should return the name, as provided in manifest's metadata/name
kubectl describe pod {name-of-pod}
```

Once you look into the pod with `describe`, you should see the values provided under the "spec" sections (port and image).

Could you pay special attention to the value in `Controllers`? We will come back to this in the next section.



### I just deployed a Pod, then what? ###

Now that you have deployed a pod, you probably want to do "fun stuff" with it, e.g. hit your application with HTTP requests, scale it up and down , maybe do a rolling update etc.

So, knowing that your application runs on port 4567, how would you access it? And how could you scale it up?

Maybe using your Node's IP and attach `:4567` to it? And maybe we can use `kubectl scale` and pass a pod to it? How about you give it a try and come back in a few minutes?

Have a poke with `kubectl`. See which commands you can run. And you can always rely on `kubectl describe pod {pod-name}` to check the state of your pod. 

<details>
<br/>
<summary>Possible Solution</summary>
Well, here is fun fact for you. You CAN'T do any of the above! :) That is also why <i>no one would deploy a Pod on its own</i>.

<br/><br/>
<img src="https://cdn.meme.am/cache/instances/folder755/59003755.jpg"/>
<br/><br/>

<p>
A <i>Pod</i> is actually completely useless on its own. That is also why a Pod is meant to be deployed with higher level constructs, such as <b>ReplicationController</b> or <b>Deployment</b>. (see the next sections)
</p>

<p>
I might have wasted a few minutes of your time, letting you type a manifest etc.... But at least, for the rest of your life, you will never deploy your application as a stand-alone </i>Pod</i>
</p>
</details>
