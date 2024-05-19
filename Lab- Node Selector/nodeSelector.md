# NodeSelector in Kubernetes for Pod Scheduling

`nodeSelector` is a feature in the PodSpec that enables the assignment of a Pod to specific nodes in the cluster. It operates by defining a set of key-value pairs, where each pair corresponds to a label on a node. A Pod can only be scheduled on a node if the node's labels match all the key-value pairs specified in the nodeSelector. This mechanism ensures that Pods are placed on nodes that meet their specific requirements.

## Task

The task is to schedule a pod that requires high processing power on a node that has high processing capabilities.

<div style="text-align:center"><img src="./images/Screenshot 2024-05-17 171915.png" width="700"></div>

### Create A deployment(deployment.yaml) with 3 replicas of pods

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: label-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: label-app
  template:
    metadata:
      labels:
        app: label-app
    spec:
      containers:
      - name: label-container
        image: nginx
      nodeSelector:
        processing: high
```

In this Deployment, three replicas of the Pod will be created. The `nodeSelector` field ensures that these Pods are scheduled on nodes labeled with `processing: high`.


### Identifying the `node`to add labels:

Before we can use nodeSelector, we need to label the nodes where we want our Pod to be scheduled. We can get the names of the nodes in our cluster by running the command ..

```
kubectl get nodes
```

this will provide the names of `nodes` in the `cluster`.

Assuming `node1`= desired `<Node-name>`

### Label the Node

```
kubectl label nodes node1 processing=high
```

### Applying the Deployment

```
kubectl apply -f deployment.yaml
```

### Verify the pods 

We can verify the pods  using the following command:

```
kubectl get pods -o wide
```

This command will display a list of all pods along with the node each pod is running on.

### Expected Output:

All the pods will be in one node which was labeled.

<div style="text-align:center"><img src="./images/Screenshot 2024-05-17 170943.png" width="700"></div>

### Shortcomings of `NodeSelector`

In our scenario, we use nodeSelector to schedule Pods on nodes labeled `processing: high`. But, if we have multiple such nodes, nodeSelector doesn't prioritize. It might schedule Pods on a nearly full node instead of a less utilized one. For better control, we can use Node Affinity to set more specific scheduling rules.



