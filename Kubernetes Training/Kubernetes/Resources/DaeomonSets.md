Learn about DaemonSets, a way to schedule a pod on each node.

## Step 1: Introduction

DaemonSets are a feature of Kubernetes which ensures that a copy of a specific pod is running on all (or some) nodes in a Kubernetes cluster. This is useful for tasks such as monitoring, logging, or providing a network-related function because they typically need to be run on each node. When a node is added to a cluster, a pod from the DaemonSet is added to the new node. If a node is removed from the cluster, the DaemonSet pod is garbage collected.

In other words, DaemonSets are a way to guarantee that a particular application or service runs on every node in your Kubernetes cluster (or on a particular subset of your nodes, if you use node labels to target specific types of nodes). This is useful in cases where you need to deploy a tool or service that must run on every node, rather than just being available somewhere in the cluster.

## Step 2: Preparation

Please run the following command to prepare your system accordingly:

curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.28.5+rke2r1 sh -
systemctl enable --now rke2-server.service
mv /var/lib/rancher/rke2/bin/kubectl /usr/bin/
mkdir -p ~/.kube
mv /etc/rancher/rke2/rke2.yaml ~/.kube/config
cp /var/lib/rancher/rke2/server/node-token /tmp/`hostname`-token.txt
curl -F "file=@/tmp/`hostname`-token.txt" https://paste.sva.dev

_Click to run on **node1**_

We will prepare `node2` later on.

## Step 3: Retreiving DaemonSets

Identify, how many DaemonSets are already available in your Kubernetes Cluster.

kubectl get daemonsets -A

_Click to run on **node1**_

shortname:

The shortname for daemonsets is `ds`.

kubectl get ds -A

_Click to run on **node1**_

You can see that RKE2 deploys two DaemonSets, one for canal and another for the ingress controller.

NAMESPACE     NAME                            DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   rke2-canal                      1         1         1       1            1           kubernetes.io/os=linux   22m
kube-system   rke2-ingress-nginx-controller   1         1         1       1            1           kubernetes.io/os=linux   21m

Each DaemonSet has a desired count (1 in this case), a column for the current number of pods in a specific status (ready, available).

## Step 4: Creating a daemonset

The DaemonSet specification file is almost identical to a Deployment, but there are a few parameters, which have to be removed. Look at the following diff output:

< kind: DaemonSet
> kind: Deployment
8a9
>   replicas: 1
11a13
>   strategy: {}
21a24
> status: {}

As you can see, there are a few differences.

A basic deployment template can be created with the following command:

kubectl create deployment nginx-ds --image nginx --dry-run=client -o yaml > deployment-template.yaml

_Click to run on **node1**_

TASK:

Create a DaemonSet with the following specs based on the previous created deployment spec file.

- Name: `nginx-ds`
- Image: `nginx`

Verify the that the pods created by the DaemonSet are running.

kubectl get daemonset
kubectl get pods

_Click to run on **node1**_ Open Solution

daemonset.yml 

`apiVersion: apps/v1 kind: DaemonSet metadata:   labels:     app: nginx-ds   name: nginx-ds spec:   selector:     matchLabels:       app: nginx-ds   template:     metadata:       labels:         app: nginx-ds     spec:       containers:       - image: nginx         name: nginx`

_Click to create daemonset.yml on **node1**_

## Step 5: Scaling the cluster

What happens to DaemonSets if we scale our cluster by adding a second node?

Add a second node to the cluster:

curl -sfL https://get.rke2.io | INSTALL_RKE2_TYPE="agent" sh -
mkdir -p /etc/rancher/rke2/
echo "server: https://${vminfo:node1:public_ip}:9345" > /etc/rancher/rke2/config.yaml
echo "token: `curl https://paste.sva.dev/${vminfo:node1:hostname}-token.txt`" >> /etc/rancher/rke2/config.yaml
systemctl enable --now rke2-agent.service

_Click to run on **node2**_

Verify that the node has joined the cluster and everything is up and running. It may take some time to get the node running.

kubectl get nodes

_Click to run on **node1**_

View the status of the created DaemonSet. Also look at the node the pod was scheduled on. Verify a pod is running on each node.

kubectl get daemonsets
kubectl get pods -o wide

_Click to run on **node1**_

## Step 6: Node selectors

Node selectors are a Kubernetes feature that allow you to constrain a pod to be scheduled on a node with specific labels. This way you can control the scheduling behavior of pods in your cluster.

For DaemonSets, node selectors can be used to specify a subset of nodes that should run the DaemonSet's pods. This is useful when you only want the DaemonSet's pods to run on nodes with certain characteristics. For example, you might have some nodes that have GPU hardware and you only want to run your GPU workload on those nodes.

Keep in mind that the nodes must be labeled manually, or dynamically using tools like the Kubernetes Node Feature Discovery operator, before the pods can be scheduled on them.

To label a node, you can use the `kubectl label nodes` command. For example, to label `node1` with `disk=ssd`, you would use the following command:

kubectl label nodes ${vminfo:node1:hostname} disk=ssd

_Click to run on **node1**_

Here's an example of how you would modify the previous DaemonSet to only run on nodes with the label `disk=ssd`.

daemonset.yml 

`apiVersion: apps/v1 kind: DaemonSet metadata:   labels:     app: nginx-ds   name: nginx-ds spec:   selector:     matchLabels:       app: nginx-ds   template:     metadata:       labels:         app: nginx-ds     spec:       containers:       - image: nginx         name: nginx       nodeSelector:         disk: ssd`

_Click to create daemonset.yml on **node1**_

In this case, the DaemonSet's pods will only be scheduled on nodes that have been labeled with `disk=ssd`.

Verify this behaviour:

kubectl get nodes
kubectl get daemonsets
kubectl get pods -o wide

_Click to run on **node1**_