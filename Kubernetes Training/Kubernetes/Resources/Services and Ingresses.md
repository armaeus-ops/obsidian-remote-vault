Learn about Services and Ingresses while deploying a GuestBook in Kubernetes.

## Step 1: Intro

In this Scenario we will bring up our first application. It is a guestbook which contains a few combined ressources.

We will:

- Use a [Kubernetes ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to make our application publically available
- Make use of [Kubernetes services](https://kubernetes.io/docs/concepts/services-networking/service/) to make our application components available inside of the kubernetes cluster
- With the help of [Kubernetes Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) we can start multiple pods and manage the number of replicas in the cluster

## Step 2: Install K3s

To install[k3s](https://k3s.io) we need to start the installer script:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

To verify our installation we can run our first `kubectl` commands:

kubectl get nodes

_Executed on **node1**_

Repeat the `kubectl get nodes` command until your node is Ready:

NAME                        STATUS   ROLES                  AGE   VERSION
dynamic-d611d165-8781479c   Ready    control-plane,master   12s   v1.20.6+k3s1

## Step 3: Set up MongoDB

As next step let's deploy a simple _(not production ready)_, multi-tier web application using Kubernetes. This example consists of the following components:

- A single-instance [MongoDB](https://www.mongodb.com/) to store guestbook entries
- Multiple web frontend instances

## Step 4: Start up the Mongo Database

The guestbook application uses MongoDB to store its data.

### Creating the Mongo Deployment

The manifest file, included below, specifies a Deployment controller that runs a single replica MongoDB Pod.

mongo-deployment.yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: mongo   labels:     app.kubernetes.io/name: mongo     app.kubernetes.io/component: backend spec:   selector:     matchLabels:       app.kubernetes.io/name: mongo       app.kubernetes.io/component: backend   replicas: 1   template:     metadata:       labels:         app.kubernetes.io/name: mongo         app.kubernetes.io/component: backend     spec:       containers:       - name: mongo         image: mongo:4.2         args:           - --bind_ip           - 0.0.0.0         resources:           requests:             cpu: 100m             memory: 100Mi         ports:         - containerPort: 27017`

_Click to create mongo-deployment.yaml on **node1**_

Create the file and afterwards apply that the manifest to our Kubernetes cluster:

kubectl apply -f mongo-deployment.yaml

_Click to run on **node1**_

After applying our configuration we need to verify that the pods are actually starting.

kubectl get pods

_Click to run on **node1**_

The result should look like:

NAME                     READY   STATUS    RESTARTS   AGE
mongo-75f59d57f4-jf8bj   1/1     Running   0          80s

Now that our pod has started and is in the `Running` state, lets have a look at the logs:

kubectl logs -f deployment/mongo

_Click to run on **node1**_

To stop following the logs, you can just type `CTRL + C`

## Step 5: Creating the MongoDB Service

The guestbook application needs to communicate to the MongoDB to write its data. You need to apply a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) to proxy the traffic to the MongoDB Pod. A Service defines a policy to access the Pods.

Our service definition looks like this:

mongo-service.yaml 

`apiVersion: v1 kind: Service metadata:   name: mongo   labels:     app.kubernetes.io/name: mongo     app.kubernetes.io/component: backend spec:   ports:   - port: 27017     targetPort: 27017   selector:     app.kubernetes.io/name: mongo     app.kubernetes.io/component: backend`

_Click to create mongo-service.yaml on **node1**_

Now lets apply that manifest to our cluster.

kubectl apply -f mongo-service.yaml

_Click to run on **node1**_

Query the list of Services to verify that the MongoDB Service is running:

kubectl get service

_Click to run on **node1**_

The response should be similar to this:

NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kubernetes   ClusterIP   10.43.0.1      <none>        443/TCP     23m
mongo        ClusterIP   10.43.54.164   <none>        27017/TCP   8s

## Step 6: Creating the Guestbook Frontend Deployment

### Set up and Expose the Guestbook Frontend

The guestbook application has a web frontend serving the HTTP requests written in PHP. It is configured to connect to the `mongo` Service to store Guestbook entries.

### Creating the Guestbook Frontend Deployment

Again everything starts with the a yaml file that describes our deployment:

frontend-deployment.yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: frontend   labels:     app.kubernetes.io/name: guestbook     app.kubernetes.io/component: frontend spec:   selector:     matchLabels:       app.kubernetes.io/name: guestbook       app.kubernetes.io/component: frontend   replicas: 3   template:     metadata:       labels:         app.kubernetes.io/name: guestbook         app.kubernetes.io/component: frontend     spec:       containers:       - name: guestbook         image: paulczar/gb-frontend:v5         # image: gcr.io/google-samples/gb-frontend:v4         resources:           requests:             cpu: 100m             memory: 100Mi         env:         - name: GET_HOSTS_FROM           value: dns         ports:         - containerPort: 80`

_Click to create frontend-deployment.yaml on **node1**_

Now lets apply that yaml file to our cluster:

kubectl apply -f https://raw.githubusercontent.com/svalabs/website/master/content/en/examples/application/guestbook/frontend-deployment.yaml

_Click to run on **node1**_

Query the list of Pods to verify that the three frontend replicas are running:

kubectl get pods -l app.kubernetes.io/name=guestbook -l app.kubernetes.io/component=frontend

_Click to run on **node1**_

## Step 7: Creating the Frontend Service

The `mongo` Services you applied is only accessible within the Kubernetes cluster because the default type for a Service is [ClusterIP](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types). `ClusterIP` provides a single IP address for the set of Pods the Service is pointing to. This IP address is accessible only within the cluster.

If you want guests to be able to access your guestbook, you must configure the frontend Service to be externally visible, so a client can request the Service from outside the Kubernetes cluster.

In this case we will use a `nodePort`

frontend-service.yaml 

`apiVersion: v1 kind: Service metadata:   name: frontend   labels:     app.kubernetes.io/name: guestbook     app.kubernetes.io/component: frontend spec:   type: NodePort   ports:   - name: "frontend-service"     port: 80     nodePort: 30080   selector:     app.kubernetes.io/name: guestbook     app.kubernetes.io/component: frontend`

_Click to create frontend-service.yaml on **node1**_

Now lets apply that spec file to our cluster:

kubectl apply -f frontend-service.yaml

_Click to run on **node1**_

Now lets see if this service is deployed:

kubectl get svc frontend

_Click to run on **node1**_

The output should look like this:

NAME       TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
frontend   NodePort   10.43.142.70   <none>        80:30080/TCP   66s

If everything worked well, we should be able to see our gestbook following this [link](http://$%7Bvminfo:node1:public_ip%7D:30080).

In many organizations it is not allowed to access ports like our `30080`. In this case we do have another option. instead of nodeport we can make use of the [Kubernetes Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/):

![](https://media.sva.dev/k8s-ingress.png)

The ingress controller usualy acts like a usual [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) and redirects trafic based on rules which can then match different services.

TASK:

Create a new file called `ingress.yaml` which contains this ingress definition and **apply** it to the cluster.

ingress.yaml 

`apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: guestbook-ingress spec:   ingressClassName: traefik   rules:   - http:       paths:       - path: /         pathType: Prefix         backend:           service:             name: frontend             port:               number: 80`

_Click to create ingress.yaml on **node1**_

Apply the Ingress:

Go ahead and apply the ingress definition

To verify you can use `kubectl` to query the ingress ressources and see if our ingress got created.

kubectl get ingress

_Click to run on **node1**_

The response reveals that we created the ingress and that it is listening on the public IP of your server:

NAME                CLASS    HOSTS   ADDRESS         PORTS   AGE
guestbook-ingress   <none>   *       ${vminfo:node1:public_ip}   80      19s

This means that we should now be able to access our guestbook application using port 80: [Our Guestbook](http://$%7Bvminfo:node1:public_ip%7D)

## Step 8: Scale the Web Frontend

You can scale up or down as needed because your servers are defined as a Service that uses a Deployment controller. That means it is very easy to respond to higher usage of your application. This can even be done automatically using the [Horizontal Pod Autoscaler](https://kubernetes.io/de/docs/tasks/run-application/horizontal-pod-autoscale/)

TASK:

The number of replicas is defined in our deployment. Try using kubectl to scale the deployment `frontend` to 5 replicas.

Help

Kubeectl can tell you a lot about what it can do. So the thing we need to do is scale a deployment. So let us search for scale in the help of kubectl

kubectl | grep scale

_Click to run on **node1**_

So it seems like `kubectl scale` is the right command.

Try do get even more help from kubectl:

kubectl scale --help

_Click to run on **node1**_ Solution

Run the following command to scale up the number of frontend Pods:

kubectl scale deployment frontend --replicas=5

_Click to run on **node1**_

Query the list of Pods to verify the number of frontend Pods running:

kubectl get pods

_Click to run on **node1**_

The result should look like this:

NAME                            READY     STATUS    RESTARTS   AGE
frontend-3823415956-70qj5       1/1       Running   0          5s
frontend-3823415956-dsvc5       1/1       Running   0          54m
frontend-3823415956-k22zn       1/1       Running   0          54m
frontend-3823415956-w9gbt       1/1       Running   0          54m
frontend-3823415956-x2pld       1/1       Running   0          5s
mongo-1068406935-3lswp          1/1       Running   0          56m

TASK:

Now try to scale the deloyment `frontend` down to 2 replicas.

Solution

kubectl scale deployment frontend --replicas=2

_Click to run on **node1**_

Query the list of Pods to verify the number of frontend Pods running:

kubectl get pods

_Click to run on **node1**_

The response should look similar to this:

NAME                            READY     STATUS    RESTARTS   AGE
frontend-3823415956-k22zn       1/1       Running   0          1h
frontend-3823415956-w9gbt       1/1       Running   0          1h
mongo-1068406935-3lswp          1/1       Running   0          1h