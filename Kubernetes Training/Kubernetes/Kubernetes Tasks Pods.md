Test your knowledge about pods with these tasks.

## Step 1: Preparation

Please run the following command to prepare your system accordingly:

curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.28.5+rke2r1 sh -
systemctl enable --now rke2-server.service 
ln -s /var/lib/rancher/rke2/bin/kubectl /usr/bin/kubectl
kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get all -n kube-system
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
echo "Install Finished. Please check cluster state by running kubectl get node. Have Fun!"

_Click to run on **node1**_

Please run the following command to authenticate against the cluster:

export KUBECONFIG=/etc/rancher/rke2/rke2.yaml

_Click to run on **node1**_

## Step 2: Task 1

TASK:

Check how many pods are running in all namespaces.

Open Solution

kubectl get pods -A

_Click to run on **node1**_

NAMESPACE     NAME                                                    READY   STATUS      RESTARTS   AGE
kube-system   cloud-controller-manager-rocky-4gb-hel1-2               1/1     Running     0          22h
kube-system   etcd-rocky-4gb-hel1-2                                   1/1     Running     0          22h
kube-system   helm-install-rke2-canal--1-lfm9s                        0/1     Completed   0          22h
kube-system   helm-install-rke2-coredns--1-g5z9d                      0/1     Completed   0          22h
kube-system   helm-install-rke2-ingress-nginx--1-gpg8r                0/1     Completed   0          22h
kube-system   helm-install-rke2-metrics-server--1-7ptwp               0/1     Completed   0          22h
kube-system   kube-apiserver-rocky-4gb-hel1-2                         1/1     Running     0          22h
kube-system   kube-controller-manager-rocky-4gb-hel1-2                1/1     Running     0          22h
kube-system   kube-proxy-rocky-4gb-hel1-2                             1/1     Running     0          22h
kube-system   kube-scheduler-rocky-4gb-hel1-2                         1/1     Running     0          22h
kube-system   rke2-canal-jxqw6                                        2/2     Running     0          22h
kube-system   rke2-coredns-rke2-coredns-5679c85bbb-5dsns              1/1     Running     0          22h
kube-system   rke2-coredns-rke2-coredns-autoscaler-6889866896-zff5x   1/1     Running     0          22h
kube-system   rke2-ingress-nginx-controller-667dz                     1/1     Running     0          22h
kube-system   rke2-metrics-server-8574659c85-5wkld                    1/1     Running     0          22h

## Step 3: Task 2

TASK:

Create a Pod with following specs:

- name: `my-first-pod`
- image: `nginx:1.21`
- Namespace: `default`

After the creation check if the pod is running.

Open Solution

INFO:

The pod shall be created in the default namespace. We do not need to provide it. Otherwise we would need the `-n <namespace>` flag.

1. Create the pod

kubectl run --image nginx:1.21 my-first-pod

_Click to run on **node1**_

2. Check if the pod is running

kubectl get pods 

_Click to run on **node1**_

## Step 4: Task 3

TASK:

Delete the pod created in the last step. Remember it's name was `my-first-pod` and it was created in the `default` namespace.

Open Solution

kubectl delete pod my-first-pod

_Click to run on **node1**_

kubectl get pods

_Click to run on **node1**_

## Step 5: Task 4

TASK:

Create a pod from a yaml file, to create the yaml file use the kubectl.

The Pod should have the following specs:

- name: `my-second-pod`
- image: `nginx:1.21`
- Namespace: `default` After the creation check if the pod is running.

Open Solution

1. Create yaml file and afterwards apply it to the cluster.

kubectl run --image nginx:1.21 my-second-pod --dry-run=client -o yaml > my-second-pod.yml
kubectl apply -f ./my-second-pod.yml

_Click to run on **node1**_

2. Check the status of the pod.

kubectl get pods 

_Click to run on **node1**_

## Step 6: Task 5

TASK:

Delete the created pod `my-second-pod` with the yaml file, and check if it is gone.

Open Solution

1. Delete the pod

kubectl delete -f ./my-second-pod.yml

_Click to run on **node1**_

2. Verify deletion.

kubectl get pods 

_Click to run on **node1**_

## Step 7: Task 6

TASK:

Create a pod with the following specs

- name: `uptime-pod`
- image: `busybox:1.34`
- Namespace: `default`
- command: `sleep inf`

After the creation check if the pod is running.

Open Solution

1. Create a pod and provide the image aswell as the command.

kubectl run uptime-pod --image busybox -- sleep inf

_Click to run on **node1**_

2. Verify the pod is running

kubectl get pods 

_Click to run on **node1**_

3. You can exec into the pod as it is running

kubectl exec -it uptime-pod -- /bin/sh 

_Click to run on **node1**_

4. Delete the pod (you need to execute this on the host, not inside the container)

kubectl delete pod uptime-pod 

_Click to run on **node1**_

## Step 8: Task 7

TASK:

Create Pod with a true loop and run the `date` command Create a pod with the following specs:

- name: `date-pod`
- image: `busybox:1.34`
- Namespace: `default`
- command: `/bin/sh -c 'while true; do date; done'`

After the creation check if the pod is running and check the logs.

Open Solution

1. Create the pod

kubectl run date-pod --image busybox -- /bin/sh -c 'while true; do date; done'

_Click to run on **node1**_

2. Verify that it is in running state

kubectl get pods

_Click to run on **node1**_

3. Retreive the logs

kubectl logs date-pod

_Click to run on **node1**_

4. Clean up the pod.

kubectl delete pod date-pod

_Click to run on **node1**_

## Step 9: Task 8

TASK:

Create a pod with environment variables, the spec should be the following

- name: `env-pod`
- image: `busybox:1.34`
- Namespace: `default`
- command: `/bin/sh -c 'while true; do export; done'`
- environment: `testvar: "This is a test var"`

Open Solution

1. Create env-pod.yaml

env-pod.yaml 

`apiVersion: v1 kind: Pod metadata:   namespace: default   name: env-pod spec:   containers:   - image: nginx     name: env-pod     args:     - /bin/sh     - -c     - while true; do export; done     env:     - name: testvar       value: "This is a test a environment"`

_Click to create env-pod.yaml on **node1**_

kubectl apply -f ./env-pod.yaml

_Click to run on **node1**_

### Cleanup resources

Delete your created resources

Open Solution

kubectl delete -f ./env-pod.yaml

_Click to run on **node1**_

## Step 10: Task 9

### Create a pod with a configmap as environment

Open Solution

1. Create `env-cm-pod.yaml`

env-cm.yaml 

`apiVersion: v1 kind: ConfigMap metadata:   name: environment-cm data:     configmap-env: "This is a configmap!"`

_Click to create env-cm.yaml on **node1**_

env-pod.yaml 

`apiVersion: v1 kind: Pod metadata:   namespace: default   name: env-cm-pod spec:   containers:   - image: nginx     name: env-pod     args:     - /bin/sh     - -c     - while true; do export; done     env:     - name: TESTENV       valueFrom:         configMapKeyRef:           name: environment-cm                      key: configmap-env`

_Click to create env-pod.yaml on **node1**_

kubectl apply -f ./env-cm.yaml
kubectl apply -f ./env-pod.yaml

_Click to run on **node1**_

### Cleanup resources

Delete your created resources

Open Solution

kubectl delete -f ./env-cm-pod.yaml

_Click to run on **node1**_

## Step 11: Task 10

### Create a pod with a secret as environment

Open Solution

1. Create `env-secret-pod.yaml`

env-secret.yaml 

`apiVersion: v1 metadata:   name: environment-secret type: Opaque data:   secret-env: VGhpcyBpcyBhIFNlY3JldCEK kind: Secret`

_Click to create env-secret.yaml on **node1**_

env-secret-pod.yaml 

`apiVersion: v1 kind: Pod metadata:   namespace: default   name: env-secret-pod spec:   containers:   - image: nginx     name: env-pod     args:     - /bin/sh     - -c     - while true; do export; done     env:     - name: TESTENV       valueFrom:           secretKeyRef:             name: environment-secret             key: secret-env`

_Click to create env-secret-pod.yaml on **node1**_

kubectl apply -f ./env-secret.yaml
kubectl apply -f ./env-secret-pod.yaml

_Click to run on **node1**_

### Cleanup resources

Delete your created resources

Open Solution

kubectl delete -f ./env-secret-pod.yaml

_Click to run on **node1**_