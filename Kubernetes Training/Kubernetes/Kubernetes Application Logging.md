Setup log aggregation with loki and query logs using grafana.

## Step 1: Logging in Kubernetes

When logging data from applications in a classic non-containerized environment, most of the time logs were appended to a file in the hosts file system. If the application crashes the logs are preserved and can be read from this file.

However Kubernetes (or containers in general) changed the way we can access logs from applications. When a pod crashes it will be restarted by kubernetes. So... what happens to the logs? With `kubectl logs` we are only able to get the logs of the current running pod. Log entries of previous pods are lost in the void.

Aggregating the logs will solve this issue for us. Loki is a log aggregator in the Kubernetes context developed by Grafana Labs.

### log aggregating - what is this?

When we say log aggregation we simply mean that we collect the logs at one central point. In our case Loki is this central point that will collect logs from all the pods in our cluster - even when a pod crashes or is deleted.

![](https://fission.io/docs/usage/observability/assets/stack.png)

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by checking the nodes status.

kubectl get nodes

_Click to run on **node1**_

Install Helm and export the kubeconfig in order for helm to find it.

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml

_Click to run on **node1**_

## Step 3: Installing Loki

First we need to add the required repositories to helm:

The first command will add the loki repository to helm, and also enables helm to locate other grafana packages. After adding the repository we will fetch new information.

helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

_Click to run on **node1**_

Deploying a loki stack is as simple as:

helm upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false

Next to loki we will get a grafana dashboard & Prometheus

## Step 4: Logging before loki

We will create a simple application that logs a timestamp.

Create a pod with the following manifest:

pod.yaml 

`apiVersion: v1 kind: Pod metadata:   name: busybox-sleep-less   labels:     app: log-example spec:   containers:   - name: busybox     image: busybox     command: ["/bin/sh"]     args: ["-c", "echo start; date; echo end;"]`

_Click to create pod.yaml on **node1**_ Solution

After creating the pod you need to apply it with the following command.

kubectl apply -f pod.yaml

_Click to run on **node1**_

Note that this pod has the label `app` set to `log-example`. This will be used in further steps.

Applying this manifest to the cluster will create a simple pod running a busybox image.

On Startup the container will echo `start`, followed by the current timestamp and `end`

Run kubectl logs to see the output of the container

kubectl logs busybox-sleep-less

_Click to run on **node1**_

You will see the timestamp in between of the start and end strings. Remember this timestamp.

What happens if you delete this pod and rerun the kubectl logs command?

kubectl delete pod busybox-sleep-less

_Click to run on **node1**_

Finally reapply the manifest so that the busybox pod is running before progressing to the next step.

## Step 5: Access Grafana

List all services that were created:

kubectl get svc

_Click to run on **node1**_

Create the following ingress to access the grafana dashboard.

ingress.yaml 

`apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: loki-ingress   annotations:     nginx.ingress.kubernetes.io/rewrite-target: / spec:   rules:   - http:       paths:       - path: /         pathType: Prefix         backend:           service:             name: loki-grafana             port:               number: 80`

_Click to create ingress.yaml on **node1**_

Apply the ingress:

Go ahead and apply the created ingress definition.

We should now have access to the [Grafana Dashboard](http://$%7Bvminfo:node1:public_ip%7D) by accessing the public IP at port 80.

To get the password for the `admin` user we need to retrieve it from the secret that was generated for us:

kubectl get secret loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

_Click to run on **node1**_

Continue if you are successfully logged into the dashboard.

## Step 6: Loki as a datasource

Loki is accessible inside our cluster at `http://loki:3100/` We can add this as a datasource in our Grafana dashboard.

INFO:

We have installed the loki stack, so Grafana automatically added loki as data source. Otherwise we would need to follow these steps:

1. Click on Configuration -> Data sources
2. Add Datasource
3. Choose Loki from the list
4. Enter `http://loki:3100/` as URL

## Step 7: Viewing logs

In the left sidebar under "Explore" we can explore logs of different data sources. Choose Loki in the dropdown on the top side.

Enter the following query:

query 

`{app="log-example"}`

and press `SHIFT` + `ENTER` or click on the `RUN QUERY` button on the top right

Remember that the pod we created had the `app` label set to `log-example`.

How many log entries can you see? Remember that we deleted the pod and lost the logs of previous executions when running `kubectl logs`.

Loki got our back and we can see all logs aggregated in one place. Even logs from the pod that was deleted and reapplied later on.