This scenario provides some tasks for you to try using kubernetes imperative commands to create pods, deployments, services and namespaces.

## Step 1: Preparation

Please run the following command to prepare your system accordingly:

curl -sfL https://get.rke2.io | INSTALL_RKE2_VERSION=v1.28.5+rke2r1 sh -
systemctl enable --now rke2-server.service 
ln -s /var/lib/rancher/rke2/bin/kubectl /usr/bin/kubectl
kubectl --kubeconfig /etc/rancher/rke2/rke2.yaml get all -n kube-system
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml
echo "Install Finished. Please check cluster state by running kubectl get node. Have Fun!"
echo "Please run the following command for connecting to the cluster:"
echo "export KUBECONFIG=/etc/rancher/rke2/rke2.yaml"

_Click to run on **node1**_

## Step 2: Task 1

Let's try using imperative commands.

TASK:

Deploy a pod with the following specifications by only using the cli commands provided by kubectl

- Name: `nginx-pod`
- Image: `nginx:1.21.0`
- Namespace: `default`

### Verification

kubectl get pod -o=custom-columns='NAME:.metadata.name,IMAGE:.spec.containers[0].image,NAMESPACE:.metadata.namespace'

_Click to run on **node1**_

The output provided should exactly match the following:

NAME        IMAGE          NAMESPACE
nginx-pod   nginx:1.21.0   default

Open Solution

kubectl run nginx-pod --image nginx:1.21.0

_Click to run on **node1**_

## Step 3: Task 2

You can provide even more information to `kubectl run`

TASK:

Now create a pod with the following specs:

- Name: `redis`
- Image: `redis:alpine`
- Label: `app=db`
- Namespace: `my-namespace`
- Ports: `8080`

Create namespace:

Create the namespace before applying the pod.

kubectl run flags:

Use `kubectl run --help` for more help about the flags

Open Solution

kubectl create namespace my-namespace

_Click to run on **node1**_

kubectl run redis --image redis:alpine --labels="app=db" -n my-namespace --port=8080

_Click to run on **node1**_

## Step 4: Task 3

TASK:

Create a deployment without utilizing definition files.

- Name: `web`
- Image: `jcdemo/flaskapp`
- Replicas: `4`

Open Solution

kubectl create deployment web --image jcdemo/flaskapp --replicas=4

_Click to run on **node1**_

## Step 5: Task 4

Now let's use everything we have done before.

TASK:

Create a deployment with the following specs:

- Name: `redis-deployment`
- Image: `redis`
- Namespace: `labs`
- Replicas: `2`

Open Solution

kubectl create namespace labs

_Click to run on **node1**_

kubectl create deployment redis-deployment --image=redis --replicas=2 --namespace=labs

_Click to run on **node1**_

## Step 6: Task 6

Create an `httpd` pod with image `httpd:alpine` in `external-labs` Namespace and expose the pod by using a service with port `80` and type `ClusterIP`.

Open Solution

Create the Namespace first, afterwards create a pod named `httpd` with the provided image. The `--expose` flag will create service with the `ClusterIP` type for us.

kubectl create namespace external-labs
kubectl run httpd --image httpd:alpine --expose=true --port=80 -n external-labs

_Click to run on **node1**_