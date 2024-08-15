Basic Helm features: Releases, Templating and Rollbacks.

## Step 1: Intro

This tutorial shows you how to use [helm](https://helm.sh) to deploy applications to your kubernetes cluster.

### Objectives

- Install helm
- How to manage repositories
- How to manage applications
    - Install applications
    - About helm releases
    - upgrading values
    - uninstalling applications

## Step 2: Setup cluster

To install [k3s](https://k3s.io) we need to start the installer script:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -
mkdir -p ~/.kube
ln -sf /etc/rancher/k3s/k3s.yaml ~/.kube/config

_Click to run on **node1**_

To verify our installation we can run our first `kubectl` commands:

kubectl get nodes

_Click to run on **node1**_

Repeat the `kubectl get nodes` command until your node is Ready:

NAME                        STATUS   ROLES                  AGE   VERSION
dynamic-d611d165-8781479c   Ready    control-plane,master   12s   v1.20.6+k3s1

### Install tree

In this lab we will display folder contents using tree.

apt update && apt install tree -y

_Click to run on **node1**_

## Step 3: Install Helm

Since helm v3 the architecture of helm changed to be client side only. That means that we need to install the helm cli tool on our server.

![](https://developer.ibm.com/developer/default/blogs/kubernetes-helm-3/images/helm3-arch.png)

There are different ways to install helm which are described [here](https://helm.sh/docs/intro/install/). In our case we will just use the installer script with a specific version of helm.

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | DESIRED_VERSION=v3.13.3 bash

_Click to run on **node1**_

After a successful installation of Helm, we should check our installation to ensure that we are ready to work with helm.

helm version --client

_Click to run on **node1**_

## Step 4: Apps without Helm

In this task we will take a look at working without Helm.

Before doing so, we need to clone our examples repository.

git clone https://github.com/svalabs/k8s-labs-helm
cd k8s-labs-helm
git checkout -f step0

_Click to run on **node1**_

### Inspect the yaml files in the blue and red directories.

We will have two examples. One is a blue and the other a red page.

tree .

_Click to run on **node1**_

Please have a closer look at the spec files to identify their purpose.

### Create the blue app

kubectl create -f blue

_Click to run on **node1**_

Afterwards you can visit the app [here](http://$%7Bvminfo:node1:public_ip%7D/blue).

### Create the red app

Lets say we need just another app that does basically the same but is red instead of blue.

kubectl create -f red

_Click to run on **node1**_

Afterwards you can visit the app [here](http://$%7Bvminfo:node1:public_ip%7D/red).

### New App in green

Lets say we just want to create another app but this time it should be green.

Try to figure out what would be needed to create the green app and deploy it to your cluster.

Help

To create your green app you would first create a copy of the existing ones.

This can be done like this:

cp -rf blue green

_Click to run on **node1**_

Now lets change the names of the files so that they match the color of our folder:

cd green
for file in *.yaml ; do mv $file ${file//blue/green} ; done
tree .

_Click to run on **node1**_

To complete the step we also need to change a few things in our spec files. This could be done with sed:

sed -i -- 's/blue/green/g' *

_Click to run on **node1**_

Now that we changed all the files, lets deploy our green app

kubectl create -f .

_Click to run on **node1**_

If everything works fine you should be able to see your app [here](http://$%7Bvminfo:node1:public_ip%7D/green).

### Cleanup

Make sure to delete all your ressources from your cluster.

cd ~/k8s-labs-helm/
kubectl delete -f blue,red,green
git clean -fd

_Click to run on **node1**_

## Step 5: Apps with Helm

In this step we will use Helm to install the app.

Lets check out the step1 branch and list the files in it.

git checkout -f step1
tree .

_Click to run on **node1**_

### Release the red application

helm install red ./color-viewer --wait

_Click to run on **node1**_

### Verify your app is running

helm ls
kubectl get all

_Click to run on **node1**_

You can visit the red application [here](http://$%7Bvminfo:node1:public_ip%7D/red).

### Doing a customized release

With helm you can also change specific values for a deployment. Helm charts do come with defaults that will be defined in the values.yaml. In our case the `values.yaml` looks like this:

values.yaml 

`color: red`

This means that we can use this variable in our templates. For example this is our service template:

yaml 

`apiVersion: v1 kind: Service metadata:   name: my-{{ .Values.color }}-service spec:   selector:     app: my-{{ .Values.color }}-deployment   ports:     - port: 80`

`{{ .Values.color }}` refers to the `color` field in the `values.yaml` file.

As a helm user we have different ways of overriding those values.

### Using inline Values

You can specify values using the command-line with the `--set` flag. This makes it very easy to override some of the values.

helm install green --set color=green ./color-viewer --wait

_Click to run on **node1**_

You can visit the green application [here](http://$%7Bvminfo:node1:public_ip%7D/green).

### Use a custom values file

Instead of passing the `--set` flag it is also possible to define those values in a separate yaml file. To specify a yaml file one would use the `-f` flag which references a yaml file containing the values.

In the following case we will use a predefined `my-values.yaml` that looks like this:

my-values.yaml 

`color: magenta`

Lets now create this magenta application

helm install magenta ./color-viewer -f my-values.yaml --wait

_Click to run on **node1**_

You can visit the magenta application [here](http://$%7Bvminfo:node1:public_ip%7D/magenta).

### Verify the installed components

helm list
kubectl get all

_Click to run on **node1**_

### Create your own application

TASK:

Now create a new application called black.

Help

To deploy a new black application you would have to override the color value to black.

This can be done using a new values.yaml like this:

values.yaml 

`color: black`

and installing the helm release as previous mentioned with `helm install black -f values.yaml`

OR by just setting the color value like this:

helm install black --set color=black ./color-viewer --wait

_Click to run on **node1**_

You can visit the black application [here](http://$%7Bvminfo:node1:public_ip%7D/black).

### Cleanup

helm uninstall red green magenta black

_Click to run on **node1**_

## Step 6: Rolling back a release

In this step we will learn how to rollback a release.

### Release the red app

helm install my-app ./color-viewer --wait

_Click to run on **node1**_

You can visit the red application [here](http://$%7Bvminfo:node1:public_ip%7D/red).

### Upgrade the red app

It might be the case that we would like to upgrade our application some time. In this case we can upgrade an existing release and also change the values.

helm upgrade my-app --set color=blue ./color-viewer --wait

_Click to run on **node1**_

You can visit the app [here](http://$%7Bvminfo:node1:public_ip%7D/blue).

Take a look at the Helm releases

helm list

_Click to run on **node1**_

### Rollback to the previous version of my-app

Take a look at the history of the `my-app` release.

helm history my-app

_Click to run on **node1**_

Finally rollback `my-app` to revision `1`.

helm rollback my-app 1 --wait

_Click to run on **node1**_

You can visit the app [here](http://$%7Bvminfo:node1:public_ip%7D/red).