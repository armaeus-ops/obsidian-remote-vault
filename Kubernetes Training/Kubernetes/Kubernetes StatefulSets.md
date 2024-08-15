Learn about StatefulSets

## Step 1: Intro

StatefulSets are intended to be used with stateful applications and distributed systems. However, the administration of stateful applications and distributed systems on Kubernetes is a broad, complex topic. In order to demonstrate the basic features of a StatefulSet, and not to conflate the former topic with the latter, you will deploy a simple web application using a StatefulSet.

After this module, you will be familiar with the following.

- How to create a StatefulSet
- How a StatefulSet manages its Pods
- How to delete a StatefulSet
- How to scale a StatefulSet
- How to update a StatefulSet's Pods

## Step 2: Install K3S

To install [k3s](https://k3s.io) we need to start the installer script:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

To verify our installation we can run our first `kubectl` commands:

kubectl get nodes

_Click to run on **node1**_

Repeat the `kubectl get nodes` command until your node is Ready:

NAME                        STATUS   ROLES                  AGE   VERSION
dynamic-d611d165-8781479c   Ready    control-plane,master   12s   v1.20.6+k3s1

## Step 3: Create a StatefulSet

The following manifest file in yaml format contains two different definitions consisting of a Service definition and a StatefulSet definition.

yaml 

`apiVersion: v1 kind: Service metadata:   name: nginx   labels:     app: nginx spec:   ports:   - port: 80     name: web   clusterIP: None   selector:     app: nginx`

yaml 

`apiVersion: apps/v1 kind: StatefulSet metadata:   name: web spec:   serviceName: "nginx"   replicas: 2   selector:     matchLabels:       app: nginx   template:     metadata:       labels:         app: nginx     spec:       containers:       - name: nginx         image: k8s.gcr.io/nginx-slim:0.8         ports:         - containerPort: 80           name: web         volumeMounts:         - name: www           mountPath: /usr/share/nginx/html   volumeClaimTemplates:   - metadata:       name: www     spec:       accessModes: [ "ReadWriteOnce" ]       resources:         requests:           storage: 1Gi`

Now lets apply that yaml manifests to our Kubernetes cluster:

kubectl apply -f https://raw.githubusercontent.com/svalabs/website/master/content/en/examples/application/web/web.yaml

_Click to run on **node1**_

After applying our configuration we need to verify that the pods are actually starting. As you can see in the manifest file, the app label _nginx_ is used. Therefore we can use this to search more granular by running the following kubectl command.

Get more output:

The argument `-o wide` is used to modify the default output of kubectl. In this case, you will see more details regarding the queried resources.

The created service is headless. That means, the service does not have any IP address allocated in the cluster.

kubectl get service nginx

_Click to run on **node1**_

Your command output should be similar to this:

NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   None         <none>        80/TCP    7m43s

Let's take a look at the statefulset we have created.

kubectl get statefulset web

_Click to run on **node1**_

Your command output should be similar to this:

NAME   READY   AGE
web    2/2     9m6s

## Step 4: Examine StatefulSets

### Ordered Pod Creation

For a StatefulSet with n replicas, when Pods are being deployed, they are created sequentially, ordered from {0..n-1}. Examine the output of the `kubectl get` command in the terminal. Eventually, the output will look like the example below.

kubectl get pods -l app=nginx -o wide

_Click to run on **node1**_

Expected output:

NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          13m
web-1   1/1     Running   0          13m

Notice that the `web-1` Pod is not launched until the `web-0` Pod is in `Running` state and is Ready.

Pods in a StatefulSet have a unique ordinal index and a stable network identity.

### Using Stable Network Identities

Each Pod of our nginx application has a stable hostname based on its ordinal index. To see this in action, please run the following commands:

kubectl exec "web-0" -- sh -c 'hostname'
kubectl exec "web-1" -- sh -c 'hostname'

_Click to run on **node1**_

This command retreives the hostname of the `web-0` and `web-1` pod.

Use `kubectl run` to execute a container that provides the `nslookup` command from the `dnsutils` package. Using `nslookup` on the Pods' hostnames, you can examine their in-cluster DNS addresses:

kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
nslookup web-0.nginx
nslookup web-1.nginx

_Click to run on **node1**_

The output should be like the following:

root@k3s-etcd01:~# kubectl run -i --tty --image busybox:1.28 dns-test --restart=Never --rm
If you don't see a command prompt, try pressing enter.
/ # nslookup web-0.nginx
Server:    10.43.0.10
Address 1: 10.43.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 10.42.0.9 web-0.nginx.default.svc.cluster.local

**Please don't forget to `exit` the container after you have finished examining the DNS addresses!**

Let's watch a StatefulSet while it's working. We will delete all pods with label app=nginx and then watch while the pods will get created again:

kubectl delete pod -l app=nginx >/dev/null
kubectl get pod -l app=nginx -w

_Click to run on **node1**_

As soon as all shown pods are _Ready_ and _Running_ again, you should stop watch them with `strg + c` and check the pods hostnames again.

kubectl exec "web-0" -- sh -c 'hostname'
kubectl exec "web-1" -- sh -c 'hostname'

_Click to run on **node1**_

You will see, that it's the same output, although the pods were recreated.

### Summary of StatefulSet Facts

The Pods' ordinals, hostnames, SRV records, and A record names have not changed, but the IP addresses associated with the Pods may have changed. In the cluster used for this tutorial, they have. This is why it is important not to configure other applications to connect to Pods in a StatefulSet by IP address.

If you need to find and connect to the active members of a StatefulSet, you should query the CNAME of the headless Service (`nginx.default.svc.cluster.local`). The SRV records associated with the CNAME will contain only the Pods in the StatefulSet that are Running and Ready.

If your application already implements connection logic that tests for liveness and readiness, you can use the SRV records of the Pods ( web-0.nginx.default.svc.cluster.local, web-1.nginx.default.svc.cluster.local), as they are stable, and your application will be able to discover the Pods' addresses when they transition to Running and Ready.

CAUTION:

Please keep in mind, that using the local path provisioner is for demonstration purposes. There are better and reliable ways to provide persistent storage to your Apps and Services.

## Step 5: Persistent / Stable Storage

### Storage

in Real Life Scenarios there will always be at some point the question regarding a safer and non volatile way of handling files and data which get created / modified during runtime of your apps.

### Stable Storage

Part of the manifest file in the first step was a section about storage. Let's check how this looks like in your single node cluster.

Get the PVCs (PersistentVolumeClaims) for both pods in the StatefulSet:

kubectl get pvc -l app=nginx

_Click to run on **node1**_

The output is similar to:

NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-35e9fd90-ee64-4a18-a132-50650e237152   1Gi        RWO            local-path     33m
www-web-1   Bound    pvc-e59580fa-45ee-4f61-a945-99ecf3aa521b   1Gi        RWO            local-path     33m

The StatefulSet controller created two PersistentVolumeClaims that are bound to two PersistentVolumes.

K3s comes with a built in Storage Provisioner as the default storage backend to dynamically provision PersistentVolumes, the PersistentVolumes were created and bound automatically.

Let's prove the functionality of the local-path storage class which was automatically used.

The NGINX webserver, by default, serves an index file from `/usr/share/nginx/html/index.html`. The volumeMounts field in the StatefulSet's spec ensures that the `/usr/share/nginx/html` directory is backed by a PersistentVolume.

Write the Pods' hostnames to their index.html files and verify that the NGINX webservers serve the hostnames. **You can use the provided commands below or on any other way you'd like to use.**

for i in 0 1; do kubectl exec "web-$i" -- sh -c 'echo "$(hostname)" > /usr/share/nginx/html/index.html'; done

for i in 0 1; do kubectl exec -i -t "web-$i" -- curl "http://localhost/" ; done

_Click to run on **node1**_

If the output contains the pod names, please go ahead and delete both pods and wait until they are _Ready_ and _Running_ again before you should check again, which html files are served:

kubectl delete pod -l app=nginx
kubectl get pod -w -l app=nginx

_Click to run on **node1**_

Verify the web servers continue to serve their hostnames:

for i in 0 1; do kubectl exec -i -t "web-$i" -- curl "http://localhost/" ; done

_Click to run on **node1**_

Even though `web-0` and `web-1` were rescheduled, they continue to serve their hostnames because the PersistentVolumes associated with their PersistentVolumeClaims are remounted to their volumeMounts.

CAUTION:

Please keep in mind, that using the local path provisioner is for demonstration purposes. There are better and reliable ways to provide persistent storage to your Apps and Services.

## Step 6: Scaling a StatefulSet

Scaling a StatefulSet refers to _increasing_ or _decreasing_ the number of _replicas_. This is accomplished by updating the replicas field. You can use either `kubectl scale` or `kubectl patch` to scale a StatefulSet.

## Step 7: Scaling Up

let's check the actual state of your pods:

kubectl get pods -l app=nginx

_Click to run on **node1**_

Now let's scale up the StatefulSet. The abbreviation for this noun is `sts`

kubectl scale sts web --replicas=5

_Click to run on **node1**_

Expected output:

statefulset.apps/web scaled

Examine the output of `kubectl get` command. Due to the last scenarios and steps you should already know the necessary command for this.

The StatefulSet controller scaled the number of replicas. As with StatefulSet creation, the StatefulSet controller created each Pod sequentially with respect to its ordinal index, and it waited for each Pod's predecessor to be _Running_ and _Ready_ before launching the subsequent Pod.

## Step 8: Scaling Down

As mentioned before, there are different ways to accomplish a desired operation / modification. Please scale down the StatefulSet by using the following `kubectl patch` command:

kubectl patch sts web -p '{"spec":{"replicas":3}}'

_Click to run on **node1**_

Watch and Wait for web-4 and web-3 to transition to Terminating.

kubectl get pods -w -l app=nginx

_Click to run on **node1**_

### Ordered Pod Termination

The controller deleted one Pod at a time, in reverse order with respect to its ordinal index, and it waited for each to be completely shutdown before deleting the next.

Get the StatefulSet's PersistentVolumeClaims:

kubectl get pvc -l app=nginx

_Click to run on **node1**_

As you can see, there are still PVCs for each Pod of the StatefulSet, although Pod `web-3` and `web-4` were deleted due to the downscaling.

There are still five PersistentVolumeClaims and five PersistentVolumes. When exploring a Pod's stable storage, we saw that the PersistentVolumes mounted to the Pods of a StatefulSet are not deleted when the StatefulSet's Pods are deleted. This is still true when Pod deletion is caused by scaling the StatefulSet down.

## Step 9: Updating StatefulSets

In Kubernetes 1.7 and later, the StatefulSet controller supports automated updates. The strategy used is determined by the `spec.updateStrategy` field of the StatefulSet API Object. This feature can be used to upgrade the container images, resource requests and/or limits, labels, and annotations of the Pods in a StatefulSet. There are two valid update strategies, `RollingUpdate` and `OnDelete`.

`RollingUpdate` update strategy is the default for StatefulSets.

## Step 10: Rolling Update

The `RollingUpdate` update strategy will update all Pods in a StatefulSet, in reverse ordinal order, while respecting the StatefulSet guarantees.

Patch the web StatefulSet to apply the RollingUpdate update strategy:

kubectl patch statefulset web -p '{"spec":{"updateStrategy":{"type":"RollingUpdate"}}}'

_Click to run on **node1**_

statefulset.apps/web patched

In the terminal window, patch the `web` StatefulSet to change the container image again and watch the progress:

kubectl patch statefulset web --type='json' -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"gcr.io/google_containers/nginx-slim:0.8"}]'
kubectl get pod -l app=nginx -w

_Click to run on **node1**_

The Pods in the StatefulSet are updated in reverse ordinal order. The StatefulSet controller terminates each Pod, and waits for it to transition to _Running_ and _Ready_ prior to updating the next Pod. Note that, even though the StatefulSet controller will not proceed to update the next Pod until its ordinal successor is _Running_ and _Ready_, it will restore any Pod that fails during the update to its current version.

Pods that have already received the update will be restored to the updated version, and Pods that have not yet received the update will be restored to the previous version. In this way, the controller attempts to continue to keep the application healthy and the update consistent in the presence of intermittent failures.

Get the Pods to view their container images:

for p in 0 1 2; do kubectl get pod "web-$p" --template '{{range $i, $c := .spec.containers}}{{$c.image}}{{end}}'; echo; done

_Click to run on **node1**_

## Step 11: OnDelete

The `OnDelete` update strategy implements the legacy (1.6 and prior) behavior, When you select this update strategy, the StatefulSet controller will not automatically update Pods when a modification is made to the StatefulSet's `.spec.template` field. This strategy can be selected by setting the `.spec.template.updateStrategy.type` to `OnDelete`

## Step 12: Deleting StatefulSets

StatefulSet supports both _Non-Cascading_ and _Cascading_ deletion. In a Non-Cascading Delete, the StatefulSet's Pods are not deleted when the StatefulSet is deleted. In a Cascading Delete, both the StatefulSet and its Pods are deleted.

## Step 13: Non-Cascading Delete

Use `kubectl delete` to delete the StatefulSet. Make sure to supply the `--cascade=orphan` parameter. This specific parameter instructs Kubernetes to solely eliminate the StatefulSet, without deleting any associated Pods.

kubectl delete statefulset web --cascade=orphan

_Click to run on **node1**_

kubectl get pods -l app=nginx

_Click to run on **node1**_

Even though the statefulset `web` has been deleted, all of the Pods are still _Running_ and _Ready_.

TASK:

Delete pod `web-0`

Expected output:

pod "web-0" deleted

solution

kubectl delete pod web-0

_Click to run on **node1**_

Get the StatefulSet's Pods:

kubectl get pods -l app=nginx

_Click to run on **node1**_

If you see only 2 pods, then the last command was successful.

Now let's "repair" the statefulset by reapplying the originating manifest:

kubectl apply -f https://raw.githubusercontent.com/svalabs/website/master/content/en/examples/application/web/web.yaml

_Click to run on **node1**_

When the web StatefulSet was recreated, it first relaunched `web-0`. Since `web-1` was already Running and Ready, when `web-0` transitioned to Running and Ready, it adopted this Pod. Since you recreated the StatefulSet with replicas equal to 2, once `web-0` had been recreated, and once `web-1` had been determined to already be Running and Ready, `web-2` was terminated.

Let's take another look at the contents of the index.html file served by the Pods' webservers:

kubectl exec -i -t "web-0" -- curl "http://localhost/"
kubectl exec -i -t "web-1" -- curl "http://localhost/"

_Click to run on **node1**_

Even though you deleted both the StatefulSet and the `web-0` Pod, it still serves the hostname originally entered into its index.html file. This is because the StatefulSet never deletes the PersistentVolumes associated with a Pod. When you recreated the StatefulSet and it relaunched `web-0`, its original PersistentVolume was remounted.

## Step 14: Cascading Delete

Please delete the StatefulSet again. This time, omit the `--cascade=orphan` parameter.

kubectl delete statefulset web

_Click to run on **node1**_

Examine the output of the kubectl get command running in the terminal, and wait for all of the Pods to transition to Terminating.

As you saw in the Scaling Down step, the Pods are terminated one at a time, with respect to the reverse order of their ordinal indices. Before terminating a Pod, the StatefulSet controller waits for the Pod's successor to be completely terminated.

INFO:

Although a cascading delete removes a StatefulSet together with its Pods, the cascade does not delete the headless Service associated with the StatefulSet. You must delete the nginx Service manually.

kubectl delete service nginx

_Click to run on **node1**_

Attention:

The PVCs created by the statefulset will continue to exist. You will have to remove them manually in a real life environment, but this time it's fine to simply finish this scenario.