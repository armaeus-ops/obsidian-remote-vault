Learn the basics about k9s, a terminal based UI for kubernetes.

## Step 1: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running.

kubectl get nodes

_Click to run on **node1**_

## Step 2: Install and configure k9s

There are multiple ways to install k9s. All of them are listed here [https://github.com/derailed/k9s#installation](https://github.com/derailed/k9s#installation)

We choose to install k9s via [Webi](https://webinstall.dev/):

curl -sS https://webinstall.dev/k9s | bash

_Click to run on **node1**_

The installer will tell you to append k9s bin folder to the PATH environment variable. Do so by executing:

export PATH="/root/.local/bin:$PATH"

_Click to run on **node1**_

Now you can run k9s by executing `k9s` However it does not show any cluster by default. This is because we are missing some context information.

Close k9s again (`ctrl + c`) and try running k9s together with the kubeconfig file located in `/etc/rancher/k3s/k3s.yaml`, which was generated during the installation of k3s.

k9s --kubeconfig /etc/rancher/k3s/k3s.yaml

_Click to run on **node1**_

This does work but we do not want to always provide the kubeconfig file when running k9s.

By simply adding the path of the config file to the `KUBECONFIG` environment variable we can solve this issue.

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

_Click to run on **node1**_

Now we can just run

k9s

_Click to run on **node1**_

without needing to provide the kubeconfig file. Great!

## Step 3: Basic navigation

k9s provides a nice UI to look into our cluster. On the first start we will see all the Pods located in the default Namespace. On a clean installation of k3s there are no pods located in the default Namespace. When pressing `0` we can switch to see pods from all namespaces.

You can navigate lists in k9s using the `up` and `down arrow keys` on your keyboard. By using the `left` and `right arrow keys` you can scroll horizontally to see more information like on which node the pod is scheduled or the age of the pod.

Press `enter` on a pod to see which containers run inside the pod. Press `enter` again to see the logs provided by the selected container. You may have to press `5` to see logs from the last hour.

Using the `Escape` key on your keyboard you can go back to see the containers, press Escape once again to exit to the list overview of the pods. Escape will back out of view, command or filter modes.

You can describe using `d`, edit using `e`, kill a pod using `ctrl-k` or display logs using `l`. Also you can quickly open a shell inside a pod using `s`

Exiting k9s can be either done with `ctrl + c` or typing `:q`

## Step 4: Navigating resources

k9s can not only display the pods located in a cluster but all other resources in the cluster

You can navigate by typing `:` followed by the resource name or a unique short-handle.

Type

- `:namespace`, `:namespaces` or `:ns` to see all namespaces
- `:deployment`, `:deployments` or `:deploy` to see all deployments
- `:ingress`, `:ingresses` or `:ing` to see all Ingresses
- `:service`, `:services` or `:svc` to see all Services

and so on.

## Step 5: There is more

### Filter lists

1. Filter lists by typing `/<filter>`
2. You can also use a regex filter `/!<filter>`
3. or filter by labels `/-l <filter`>

Where `<filter>` is a string, the regex or a label-selector. Filter all pods including `dns` in their name by typing `/dns`

### How to ... ?

Press `?` and you will see a nice display of all commands.