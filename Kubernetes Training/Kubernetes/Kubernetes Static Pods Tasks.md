Learn what static pods are and how you can create one.

## Step 1: Preparation

Please run the following command to prepare your system accordingly:

curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.28.5+rke2r1 sh -
systemctl enable --now rke2-server.service 
ln -s /var/lib/rancher/rke2/bin/kubectl /usr/bin/kubectl
kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get all -n kube-system

_Click to run on **node1**_

Please run the following command to authenticate against the cluster:

export KUBECONFIG=/etc/rancher/rke2/rke2.yaml

_Click to run on **node1**_

## Step 2: Task 1

TASK:

Identify, how many Static Pods are actually available in your Kubernetes Cluster. (in all namespaces)

Open Solution

Search for all Pods with `-controlplane` in the name

kubectl get pods -A

_Click to run on **node1**_

## Step 3: Task 2

TASK:

Search for the Location of static pods manifest files. Create a new static pod by creating the appropriate file in the located directory.

- Name: `static-busybox`
- Namespace: `default`
- image: `busybox`
- command: `sleep 500`

Open Solution

Create a pod yaml manifest in this in Path: `/var/lib/rancher/rke2/server/manifests/`

static-busybox.yaml 

`apiVersion: v1 kind: Pod metadata:   name: static-busybox   namespace: default spec:   containers:   - command:     - sh      - -c     - --     args:       - sleep 1000     image: busybox     imagePullPolicy: IfNotPresent     name: static-busybox`

_Click to create /var/lib/rancher/rke2/server/manifests/static-busybox.yaml on **node1**_