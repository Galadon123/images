# Taints and Tolerations in Kubernetes

Taints and tolerations in Kubernetes are a way to ensure that certain nodes in a cluster only run certain pods.

**Taints**:A taint is a property you can apply to a node, marking it as "tainted". This means that no pod will be scheduled to run on it, unless the pod has a matching toleration.
In Kubernetes, taints can be assigned one of three effects: NoSchedule, PreferNoSchedule, and NoExecute. 

- **NoSchedule**: This is the hardest restriction. It means that no pod will be scheduled on the node unless it has a matching toleration.

- **PreferNoSchedule**: This is a softer version of NoSchedule. The system will try to avoid placing a pod that doesn't have a matching toleration on the node, but it's not a hard requirement.

- **NoExecute**: This means that pods without a matching toleration will be evicted from the node if they are already running. If a pod has a matching toleration, it can continue running.

**Toleration**:A toleration is a property you can apply to a pod, allowing it to be scheduled on nodes with matching taints.

## Task:

 let's consider a scenario where we have a Kubernetes cluster with multiple nodes and we want to ensure that a particular set of pods (let's say, res-intensive pods) only run on a specific node.

<div style="text-align:center"><img src="./images/task.png" width="800"></div>

### Create A deployment(deployment.yaml) with 3 replicas of pods with toleration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-with-toleration
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app-1
  template:
    metadata:
      labels:
        app: my-app-1
    spec:
      containers:
      - name: pod-container-1
        image: nginx
      tolerations:
      - key: "app"
        operator: "Equal"
        value: "res-intensive"
        effect: "NoSchedule"
```

This YAML defines a `Deployment` with the specified container and toleration settings, ensuring that three replicas of the `pod` are created

### Create A deployment(deployment2.yaml) with 3 replicas of pods without toleration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-without-toleration
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app-2
  template:
    metadata:
      labels:
        app: my-app-2
    spec:
      containers:
      - name: pod-container-2
        image: nginx
```


### Applying Taints on node

##### Identifying the `node`to taints:

```
kubectl get nodes
```
this will provide the names of `nodes` in the `cluster`.

Assuming `node1`= desired `<Node-name>`

##### Apply the taint to the node (`node1`):

```
kubectl taint nodes node1 app=res-intensive:NoSchedule
```

This command applies a taint with the key `app`, value `res-intensive`, and effect `NoSchedule` to node1.

We can also apply taint with a specific effect (Optional):

```
kubectl taint nodes node1 key=value:PreferNoSchedule
kubectl taint nodes node1 key=value:NoExecute
```

### Applying the Deployments

```
kubectl apply -f deployment1.yaml
kubectl apply -f deployment2.yaml
```

### Verify the pods 

```
kubectl get pods -o wide
```
This will display more information about each pod, including the `node` each pod is running on

<div style="text-align:center"><img src="./images/Screenshot 2024-05-16 163034.png" width="700"></div>

### Shortcomings in Taints & Toleration

Pods that have toleration can be ended up with nodes which is not desired.

<div style="text-align:center"><img src="./images/toleration-demo.png" width="700"></div>

 