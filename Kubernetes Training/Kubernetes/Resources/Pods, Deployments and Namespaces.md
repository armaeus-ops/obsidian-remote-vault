Learn about the basic kubernetes commands using kubectl

## Step 1: Intro

This tutorial shows you how to interact with a kubernetes cluster. We will go through the basic first commands, deploy some pods and get an overview of the resources.

### Objectives

- What is a kubeconfig?
- Basics about Kubernetes Objects
- Create our first [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)
- Create a first [pod](https://kubernetes.io/de/docs/concepts/workloads/pods/)
- Get [logs](https://kubernetes.io/docs/concepts/cluster-administration/logging/) of a pod
- [Execute commands](https://kubernetes.io/docs/tasks/debug-application-cluster/get-shell-running-container/) inside of pods

## Step 2: Install K3S

In this lab we will use [k3s](https://k3s.io) as Kubernetes distribution.

Usually a Kubernetes cluster looks like this:

![](https://media.sva.dev/kubernetes-work.svg)

Which means that there are many pods already running to fulfill different jobs of the kubernetes cluster itself. In a fully fledged kubernetes distribution this also means support for cloud providers and different storage and network integrations. In general a fully fledged Kubernetes cluster is not the lightest installation to manage only a few containers.

This is where k3s comes in. it has a much lighter footprint and is basically made for IoT and edge computing scenarios. For the reason it is much more lighweight and selfcontained.

![](https://k3s.io/images/how-it-works-k3s.svg)

It only needs 512MB RAM and comes as a single self contained binary that just needs around 100MB of storage. Even the usual cluster database etcd is replaced with sqlite which of course makes the footprint a lot lighter then a full Kubernetes installation.

Eventhough it is a very minimalized kubernetes distribution it is fully API compatible what makes it so good for reproducing things locally or to do trainings with it like we do in this lab. Because of the light footprint and the IoT use-cases K3s even runs on a Raspberry Pi.

To install [k3s](https://k3s.io) we only need to start the installer script:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

To verify our installation we can run our first `kubectl` commands:

kubectl get nodes

_Click to run on **node1**_

Repeat the `kubectl get nodes` command until your node is Ready:

NAME                        STATUS   ROLES                  AGE   VERSION
dynamic-d611d165-8781479c   Ready    control-plane,master   12s   v1.20.6+k3s1

## Step 3: Kubeconfig

Kubernetes has a built in user and permissions management. All Users need to authenticate against a kubernetes cluster. To make this work every user gets its own API token or a client certificate to authenticate against the api server of a cluster. To make sure credentials are getting passed privately, the connecting is encrypted using SSL. There is no unauthenticated or unencrypted traffic for the kubernetes components to ensure a maximum of security.

Usually a cluster administrator creates new users, tokens and generates so called kubeconfig files.

A kubeconfig file includes everything needed to connect to the cluster:

- the adress of the api server
- username and token or a client certificate
- the public ca key of our api server to validate the ssl connection

Now that we installed out kubernetes cluster and also used kubectl to verify that our node is ready, lets have a look at our kubeconfig file.

cat /etc/rancher/k3s/k3s.yaml

_Click to run on **node1**_

The output should look like this:

k3s.yaml 

`apiVersion: v1 clusters: - cluster:     certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkakNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdGMyVnkKZG1WeUxXTmhRREUyTWpreU5qVTVNekl3SGhjTk1qRXdPREU0TURVMU1qRXlXaGNOTXpFd09ERTJNRFUxTWpFeQpXakFqTVNFd0h3WURWUVFEREJock0zTXRjMlZ5ZG1WeUxXTmhRREUyTWpreU5qVTVNekl3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFTNlZDV0h5aHU5aUdOanRyWGNGZlJsRmZwTm1qWURwK0p5Q01ldkVVSUEKQk1IRXk0S1FpNDFWMUNBRGVjTVBCQ3hoSTZ6K3lpSUsyVGdaNzc3Rm1wNjZvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVUJtNnZocVVrQ1MxWnZ5a2xPNWF4ClR2d0VEZ293Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnSjRtTk5VTmRULzFuKzJqUjhFeFVxWW1KazJMS2c4TE0KazBJT3FleXhHcTBDSUNyeGdjOStVK1lWbi9jbUtRVlZNbTZ3aG4wZi90M1JHR2VyMjRHaENxWlQKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=     server: https://127.0.0.1:6443   name: default contexts: - context:     cluster: default     user: default   name: default current-context: default kind: Config preferences: {} users: - name: default   user:     client-certificate-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJrVENDQVRlZ0F3SUJBZ0lJT3U0UkpySWxsZzh3Q2dZSUtvWkl6ajBFQXdJd0l6RWhNQjhHQTFVRUF3d1kKYXpOekxXTnNhV1Z1ZEMxallVQXhOakk1TWpZMU9UTXlNQjRYRFRJeE1EZ3hPREExTlRJeE1sb1hEVEl5TURneApPREExTlRJeE1sb3dNREVYTUJVR0ExVUVDaE1PYzNsemRHVnRPbTFoYzNSbGNuTXhGVEFUQmdOVkJBTVRESE41CmMzUmxiVHBoWkcxcGJqQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxR1NNNDlBd0VIQTBJQUJNczJ3eFBucWZSaVFVMmUKQ0czMWVLQ2d3enk0ZFZQeTNHRWVKdUZ4SXhyRnBMR1JRc1VUaHRpemhxS0c1SFVGNi9HanlVTjZ3S21ER0xKRgpyUXF0UE9LalNEQkdNQTRHQTFVZER3RUIvd1FFQXdJRm9EQVRCZ05WSFNVRUREQUtCZ2dyQmdFRkJRY0RBakFmCkJnTlZIU01FR0RBV2dCVHAzMmt6dzlBK1FVYVVBS3N3cmJ2dVNSbzVsakFLQmdncWhrak9QUVFEQWdOSUFEQkYKQWlBdGNQRk5JVTk3dW5WNUtFVHpnbzR2WDFJQU1hTjArclRKQWl4Z0xFMTNDQUloQU90dDhFUWNudXFUT3JYKwpwcmp1cys4bndBdFkyTDVZeWtRekVtWTltMEd1Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0KLS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJkekNDQVIyZ0F3SUJBZ0lCQURBS0JnZ3Foa2pPUFFRREFqQWpNU0V3SHdZRFZRUUREQmhyTTNNdFkyeHAKWlc1MExXTmhRREUyTWpreU5qVTVNekl3SGhjTk1qRXdPREU0TURVMU1qRXlXaGNOTXpFd09ERTJNRFUxTWpFeQpXakFqTVNFd0h3WURWUVFEREJock0zTXRZMnhwWlc1MExXTmhRREUyTWpreU5qVTVNekl3V1RBVEJnY3Foa2pPClBRSUJCZ2dxaGtqT1BRTUJCd05DQUFTeE9kRVBnaUZtdWEwbkFQOTdMekhraGtLZkRJYVZIa0ZibTJ0WUU5dDMKM05aeGZGc3lLTDlTQjBIOS8zSGtjSnBzZ29CMlNVNk9iZW1DNzF6MTNDMEtvMEl3UURBT0JnTlZIUThCQWY4RQpCQU1DQXFRd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZEJnTlZIUTRFRmdRVTZkOXBNOFBRUGtGR2xBQ3JNSzI3Cjdra2FPWll3Q2dZSUtvWkl6ajBFQXdJRFNBQXdSUUloQUp5dVJJSjNMbnh3L1ZsR3RsaTdWWjFSc256RnZJN1MKWG5vdGZZOFpBdXdpQWlBYUU4QzlFbWxiMDZrMGdpTktDSWRZd2xzT2RNYy85UkxZSW1wSEo0UG9yQT09Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K     client-key-data: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSUVvUkdHZjJiOG9WcnlzMlBkWGZtQlJtU3N4azlNcHF5WmRVQ1NKdG1Hd3VvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFeXpiREUrZXA5R0pCVFo0SWJmVjRvS0REUExoMVUvTGNZUjRtNFhFakdzV2tzWkZDeFJPRwoyTE9Hb29ia2RRWHI4YVBKUTNyQXFZTVlza1d0Q3EwODRnPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=`

In our case we have a certificate to authenticate against our kubernetes cluster.

Kubectl is the cli tool that lets us interact with Kubernetes Clusters. By default it will search for a kubeconfig in your homedirectory in a folder called .kube inside of that folder it searches for a file called config.

It is also possible to specify kubeconfig files using an environment variable. This could be done like this:

export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

## Step 4: Understanding Kubernetes objects

Kubernetes objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

- What containerized applications are running (and on which nodes)
- The resources available to those applications
- The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

A Kubernetes object is a "record of intent"--once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you're effectively telling the Kubernetes system what you want your cluster's workload to look like; this is your cluster's _desired state_.

To work with Kubernetes objects--whether to create, modify, or delete them--you'll need to use the [Kubernetes API](https://kubernetes.io/docs/concepts/overview/kubernetes-api/). When you use the `kubectl` command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you. You can also use the Kubernetes API directly in your own programs using one of the [Client Libraries](https://kubernetes.io/docs/reference/using-api/client-libraries/).

### Object Spec and Status

Almost every Kubernetes object includes two nested object fields that govern the object's configuration: the object `spec` and the object `status`. For objects that have a `spec`, you have to set this when you create the object, providing a description of the characteristics you want the resource to have: its _desired state_.

The `status` describes the _current state_ of the object, supplied and updated by the Kubernetes system and its components. The Kubernetes continually and actively manages every object's actual state to match the desired state you supplied.

For example: in Kubernetes, a Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment `spec` to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application--updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction--in this case, starting a replacement instance.

For more information on the object spec, status, and metadata, see the [Kubernetes API Conventions](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md).

### Create a Kubernetes object

When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name). When you use the Kubernetes API to create the object (either directly or via `kubectl`), that API request must include that information as JSON in the request body. **Most of the times, you provide the information to `kubectl` in a .yaml file.** `kubectl` converts the information to JSON when making the API request.

Here's an example `.yaml` file that shows the required fields and object spec for a Kubernetes Pod:

pod.yml 

`apiVersion: v1 kind: Pod metadata:   name: single-pod   labels:     app: nginx spec:   containers:   - name: nginx     image: nginx:1.14.2     ports:     - containerPort: 80`

_Click to create pod.yml on **node1**_

What is a pod?:

In Kubernetes, a Pod represents the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod encapsulates an application container (or, in some cases, multiple closely related containers), storage resources, and a unique network IP.

Apply resources to the cluster:

One way to create a Pod using a `.yaml` file like the one above is to use the [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) command in the `kubectl` command-line interface, passing the `.yaml` file as an argument with the `-f` flag.

kubectl apply -f pod.yml

_Click to run on **node1**_

Make sure you created the pod manifest:

If the apply command fails make sure to create the manifest first by either clicking on the file or creating the file manually with any editor.

### Required Fields

In the `.yaml` file for the Kubernetes object you want to create, you'll need to set values for the following fields:

- `apiVersion` - Which version of the Kubernetes API you're using to create this object
- `kind` - What kind of object you want to create
- `metadata` - Data that helps uniquely identify the object, including a `name` string, `UID`, and optional `namespace`
- `spec` - What state you desire for the object

The precise format of the object `spec` is different for every Kubernetes object, and contains nested fields specific to that object. The [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/) can help you find the spec format for all of the objects you can create using Kubernetes.

For example, the reference for Pod details the [`spec` field](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec) for a Pod in the API, and the reference for Deployment details the [`spec` field](https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/deployment-v1/#DeploymentSpec) for Deployments. In those API reference pages you'll see mention of PodSpec and DeploymentSpec. These names are implementation details of the Golang code that Kubernetes uses to implement its API.

The pod that we created leads to Kubernetes ensuring that the desired state will be established. Kubernetes also keeps track of which ressource got created in order to accomplish that state.

Lets have a look at the pod that our cluster created. You should see the just created pod `single-pod`:

kubectl get pods

_Click to run on **node1**_

We can see some new nginx pods got created. If we now go ahead and delete the pod definition, Kubernetes will make sure to delete all the ressources which got created:

kubectl delete -f pod.yml

_Click to run on **node1**_

Verify that the pod is being deleted:

kubectl get pods

_Click to run on **node1**_

INFO:

We could also execute `kubectl delete pod single-pod` in order to delete the pod.

## Step 5: Deployments

Next up are deployments. A Deployment in Kubernetes is a higher-level concept that manages ReplicaSets and provides declarative updates to Pods along with a lot of other useful features. This means that a Deployment can handle updating from one version of an app to another in a controlled way.

Deployments represent a set of multiple, identical Pods with no unique identities. A Deployment runs multiple replicas of your application and automatically replaces any instances that fail or become unresponsive. In other words, Deployments help ensure that one or more instances of your application are available to serve user requests.

Here are some of the key features of Deployments:

- Updating and rolling back: Deployments can update an application to a new version seamlessly using a rolling update strategy, ensuring no downtime during the transition. If something goes wrong, Kubernetes provides a rollback feature where you can revert to the previous state of the Deployment.
- Scaling: You can scale up the number of Pods in a Deployment if you expect high traffic. When the traffic decreases, you can scale it down to save resources.
- Self-healing: If a Pod goes down, the Deployment will create a new one to replace it, ensuring your application is always available.

Given the following manifest

deployment.yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: nginx-deployment spec:   selector:     matchLabels:       app: nginx   replicas: 2 # tells deployment to run 2 pods matching the template   template:     metadata:       labels:         app: nginx     spec:       containers:       - name: nginx         image: nginx:1.14.2         ports:         - containerPort: 80`

_Click to create deployment.yaml on **node1**_

Create the deployment by running

kubectl apply -f deployment.yaml

_Click to run on **node1**_

See that the deployment was created

kubectl get deployments

_Click to run on **node1**_

The Controller Manager automaticalla created another ressource called `ReplicaSet`:

kubectl get replicasets

_Click to run on **node1**_

What is a ReplicaSet?:

A ReplicaSet in Kubernetes is a lower-level abstraction that ensures a specified number of pod replicas are running at any given time. ReplicaSets are designed to maintain a stable set of replica Pods running at any given time. As such, they are often used to guarantee the availability of a specified number of identical Pods.

The replicaset ensures we have 2 pods running, as defined in the deployment manifest:

kubectl get pods

_Click to run on **node1**_

### Updating

Maybe you noticed we set the image to `nginx:1.14.2` which is a pretty old version. A newer version available would be `nginx:1.24.0`.

Update the `deployment.yml` accordingly

deployment.yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: nginx-deployment spec:   selector:     matchLabels:       app: nginx   replicas: 2 # tells deployment to run 2 pods matching the template   template:     metadata:       labels:         app: nginx     spec:       containers:       - name: nginx         image: nginx:1.24.0         ports:         - containerPort: 80`

_Click to create deployment.yaml on **node1**_

Run the apply command again to update the image and watch the pods getting updated.

kubectl apply -f deployment.yaml
watch kubectl get pods -o wide

_Click to run on **node1**_

INFO:

to end live view use `ctrl+c`

### Scaling

We can simply scale the deployment to 5 replicas by running:

kubectl scale deployment/nginx-deployment --replicas=5

_Click to run on **node1**_

Verify that the number of pods has changed:

kubectl get pods

_Click to run on **node1**_

### Self-healing

Delete one of the pods:

Go ahead and delete one of the running pods using `kubectl delete pod <name>`. You can copy the name from the output of the previous `kubectl get pods` command.

What happened after you deleted the pod?

Actually the pods came up again. The replicaset was not deleted when you deleted the pod. A new pod started, this can be verified by executing

kubectl get pods

_Click to run on **node1**_

Look at the time the pods were created.

### Scale to Zero

To scale a deployment to zero, and also to remove all replicas, we can use the `--replicas=0` flag.

kubectl scale deployment/nginx-deployment --replicas=0

_Click to run on **node1**_

now check if your deployment still exists:

kubectl get deployment/nginx-deployment

_Click to run on **node1**_

## Step 6: Namespace

Kubernetes namespaces let us seperate different workloads. It is also possible to use namespaces for multi user scenarios in which user1 gets access to his own namespace and is not able to see the workloads of other users.

Lets see which namespaces are already there:

kubectl get namespaces

_Click to run on **node1**_

The output looks like this:

NAME              STATUS   AGE
default           Active   11m
kube-system       Active   11m
kube-public       Active   11m
kube-node-lease   Active   11m

Kubernetes starts with four initial namespaces:

- `default` The default namespace for objects with no other namespace
- `kube-system` The namespace for objects created by the Kubernetes system
- `kube-public` This namespace is created automatically and is readable by all users (including those not authenticated). This namespace is mostly reserved for cluster usage, in case that some resources should be visible and readable publicly throughout the whole cluster. The public aspect of this namespace is only a convention, not a requirement. For protecting a production cluster you should follow [this documentation](https://kubernetes.io/docs/tasks/administer-cluster/securing-a-cluster/)
- `kube-node-lease` This namespace holds Lease objects associated with each node. Node leases allow the kubelet to send heartbeats so that the control plane can detect node failure.

When using kubectl it always interacts with the default namespace.

So if we for example query for pods:

kubectl get pods

_Click to run on **node1**_

We will see that there are no pods:

No resources found in default namespace.

But this is not the case. It's just the case for the default namespace. If we would like to see specifically what pods are in the kube-system namespace, we can do it like this:

kubectl get pods --namespace kube-system

_Click to run on **node1**_

An even shorter way would be using `-n`:

kubectl get pods -n kube-system

_Click to run on **node1**_

If you would like to get all pods in our cluster we can use `-A`

kubectl get pods -A

_Click to run on **node1**_

INFO:

`-A` in this case is just short for `--all-namespaces` which will lead to the same result.

Now lets create our own own namespace `sva-rocks`:

kubectl create namespace sva-rocks

_Click to run on **node1**_

The output should look like this:

namespace/sva-rocks created

Now lets again get a list of all namespaces:

kubectl get namespaces

_Click to run on **node1**_

## Step 7: Create Pod

To deploy a container that generates some logs lets just deploy a pod.

wget https://sva-academy.s3.eu-central-1.amazonaws.com/shared-files/counter-pod.yml

_Click to run on **node1**_

lets check our pod configuration:

cat counter-pod.yml

_Click to run on **node1**_

it should look someting like this

yml 

`apiVersion: v1 kind: Pod metadata:   name: counter spec:   containers:   - name: count     image: busybox     args: [/bin/sh, -c, 'i=0; while [ $i -lt 1000 ]; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done']`

Lets just apply that spec to our cluster:

kubectl apply -f counter-pod.yml

_Click to run on **node1**_

Output looks like this:

pod/counter created

We can now also query this pod:

kubectl get pods

_Click to run on **node1**_

If we want to get even more information we can use the `describe` subcommand and kubernetes will tell us everything it knows about our pod:

kubectl describe pod counter

_Click to run on **node1**_

The output looks like this:

output 

`Name:         counter Namespace:    default Priority:     0 Node:         dynamic-6788f126-6c27bef6/168.119.125.166 Start Time:   Wed, 18 Aug 2021 08:25:19 +0200 Labels:       <none> Annotations:  <none> Status:       Running IP:           10.42.0.9 IPs:   IP:  10.42.0.9 Containers:   count:     Container ID:  docker://46534d667d2f5e153e931ae921b52bafdd621c237f66b3acac5c9c027b242559     Image:         busybox     Image ID:      docker-pullable://busybox@sha256:0f354ec1728d9ff32edcd7d1b8bbdfc798277ad36120dc3dc683be44524c8b60     Port:          <none>     Host Port:     <none>     Args:       /bin/sh       -c       i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done     State:          Running       Started:      Wed, 18 Aug 2021 08:25:22 +0200     Ready:          True     Restart Count:  0     Environment:    <none>     Mounts:       /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-wgp4t (ro) Conditions:   Type              Status   Initialized       True   Ready             True   ContainersReady   True   PodScheduled      True Volumes:   kube-api-access-wgp4t:     Type:                    Projected (a volume that contains injected data from multiple sources)     TokenExpirationSeconds:  3607     ConfigMapName:           kube-root-ca.crt     ConfigMapOptional:       <nil>     DownwardAPI:             true QoS Class:                   BestEffort Node-Selectors:              <none> Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s                              node.kubernetes.io/unreachable:NoExecute op=Exists for 300s Events:                      <none>`

## Step 8: Retreive Logs

After creating pods we often need to access logs to verify that everything works fine. `kubectl` also has a `logs` subcommand that allows us to see those.

kubectl logs counter

_Click to run on **node1**_

INFO:

Did you see that we did not need to provide the type of ressource, only the name of the pod? Logs can of course only be retreived from pods - therefore we only need to supply the pods name.

The command prints out all the logs that this pod created. It is also possible to attach to the logs using the `-f` flag.

kubectl logs -f counter

_Click to run on **node1**_

When you are done, you can cancel the log output using `CTRL + C`.

## Step 9: Execute Commands

In some situations it is needed to verify that config files are in the right place, have the right content or that processes are running the right way. In Kubernetes kubectl also lets us execute interactive commands in our container.

For this case the `exec` subcommand will help us:

kubectl exec --help

_Click to run on **node1**_

To get an interactive shell we need kubectl's `exec` subcommand. Similar to docker we have to specify whether we want this to be an interactive session and that there should be a TTY allocated. This is why we add `-it` Next comes the name of the pod followed by `--`. The double dash `--` separates the arguments you want to pass to the command from the kubectl arguments. As we just want a shell in our container, we add `/bin/ash`

kubectl exec -it counter -- /bin/ash

_Click to run on **node1**_

Using `ps` lets have a look at the processes inside the pod:

ps -ef

_Click to run on **node1**_

The output looks like this:

PID   USER     TIME  COMMAND
    1 root      0:00 /bin/sh -c i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1; done
 1497 root      0:00 /bin/ash
 1508 root      0:00 sleep 1
 1509 root      0:00 ps -ef

To exit a pod just type `exit`.

## Step 10: Conclusion

Kubectl is what ssh is for traditional servers. It lets us connect to servers, lets us deploy applications and also debug them.

Besides just aplying resources to our cluster, kubectl can help in debuging and even also handle [port forwardings](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) for us in case we need to connect to a cluster service from our local computer.

The kubeconfig in this case does the same what an ssh-key does and should always kept secure because those are the main credentials others will need to control your cluster.