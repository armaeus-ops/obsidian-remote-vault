Learn to persist data using PersistentVolumes and PersistentVolumeClaims while building a simple wordpress application.

## Step 1: Intro

This tutorial shows you how to deploy a WordPress site and a MySQL database. Both applications use PersistentVolumes and PersistentVolumeClaims to store data.

A [PersistentVolume](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) (PV) is a piece of storage in the cluster that has been manually provisioned by an administrator, or dynamically provisioned by Kubernetes using a [StorageClass](https://kubernetes.io/docs/concepts/storage/storage-classes). A [PersistentVolumeClaim](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims) (PVC) is a request for storage by a user that can be fulfilled by a PV. PersistentVolumes and PersistentVolumeClaims are independent from Pod lifecycles and preserve data through restarting, rescheduling, and even deleting Pods.

![](https://media.sva.dev/k8s-pvc.png)

### Objectives

- Create PersistentVolumeClaims and PersistentVolumes
- Create a `kustomization.yaml` with
    - a Secret generator
    - MySQL resource configs
    - WordPress resource configs
- Apply the kustomization directory by `kubectl apply -k ./`
- Clean up

## Step 2: Install K3S

To install [k3s](https://k3s.io) we only need to start the installer script:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

To verify our installation we can run our first `kubectl` commands:

kubectl get nodes

_Click to run on **node1**_

Repeat the `kubectl get nodes` command until your node is Ready:

NAME                        STATUS   ROLES                  AGE   VERSION
dynamic-d611d165-8781479c   Ready    control-plane,master   12s   v1.20.6+k3s1

## Step 3: StorageClasses

Kubernetes StorageClasses offer a dynamic way to manage storage resources in a Kubernetes environment, providing flexibility, scalability, and the seamless integration of storage solutions.

StorageClasses in Kubernetes are a powerful abstraction mechanism that allows you to define different types of storage based on the underlying storage system. They enable administrators to define and manage the storage resources in a cluster dynamically, without having to bind them to a specific Persistent Volume (PV). This approach offers a more flexible and automated way to provision and allocate storage resources as needed by Persistent Volume Claims (PVCs).

List StorageClasses:

List all available StorageClasses in k3s

Solution

kubectl get storageclasses

_Click to run on **node1**_

k3s by default brings only one StorageClass: `local-path` This is also the defualt StorageClass. The default can be changed if other StorageClasses were applied.

### Available Provisioners

A list of available StorageClass CSI Provisioners can be found in the [kubernetes documentation](https://kubernetes-csi.github.io/docs/drivers.html)

The list includes:

- ceph
- portworx
- VMWare vSphere
- AWS
- Azure
- DellEMC
- Google Cloud
- HPE
- IBM Storage Scale
- Longhorn
- NFS
- Proxmox and many more.

## Step 4: PersistentVolumes

PersistentVolumes (PVs) are a crucial component in the Kubernetes storage architecture, providing an abstraction over physical storage resources. They offer a method for administrators to provision and manage storage resources outside of the pod lifecycle, thereby enabling data persistence across pod restarts and redeployments. A PV can be provisioned statically by an administrator or dynamically through the use of a StorageClass. Each PV contains details about the storage capacity, access modes, and the physical storage it represents, whether it's on-premises (like NFS or iSCSI) or in the cloud (such as AWS EBS, Google Persistent Disk, or Azure Disk Storage). PVs enable users to abstract the details of how storage is provided from how it is consumed, simplifying the deployment and management of applications that require persistent data storage.

## Step 5: PersistentVolumeClaims

PersistentVolumeClaims (PVCs) act as a user's request for storage within a Kubernetes cluster. PVCs specify the size, access modes (such as read/write or read-only), and sometimes the specific StorageClass required by the application. Kubernetes matches a PVC to an appropriate PV based on the specifications and binds them together. This binding process ensures that the PVC has exclusive use of that PV, respecting the storage requirements specified by the user. If no suitable PV exists, and if the PVC specifies a StorageClass, the cluster can dynamically provision a new PV that matches the claim's requirements and bind them together. PVCs abstract the details of the underlying storage infrastructure from the users, allowing developers to focus on the application's needs without worrying about the specifics of the storage system. This model facilitates a high level of portability and flexibility in application deployment across different environments and cloud providers.

Example for a PVC manifest

yaml 

`apiVersion: v1 kind: PersistentVolumeClaim metadata:   name: mysql-pv-claim   labels:     app: wordpress spec:   accessModes:     - ReadWriteOnce   resources:     requests:       storage: 20Gi`

### Available Access Modes

- `ReadWriteOnce`
- `ReadOnlyMany`
- `ReadWriteMany`
- `ReadWriteOncePod`

## Step 6: Using Storage in pods

By defining the `.Spec.volumes` key we can define what PVC to use by referencing the name. We also give the volume a name that is only used for this pod. The name will be referenced unter the `.Spec.containers.volumeMounts` section. Additionally we need to provide a `mountPath`.

yaml 

`apiVersion: v1 kind: Pod metadata:   name: pv-pod spec: containers:   - name: pv-recycler     image: "registry.k8s.io/busybox"     command: ["/bin/sh", "-c", „ls /data“]     volumeMounts:     - name: vol-data       mountPath: /data  volumes:   - name: vol-data     persistentVolumeClaim:       claimName: pvc-name`

The manifest might look different when using other providers like nfs:

yml 

`apiVersion: v1 kind: Pod metadata:   name: test-nfs spec:   containers:   - image: registry.k8s.io/test-webserver     name: test-container     volumeMounts:     - mountPath: /my-nfs-data       name: test-volume   volumes:   - name: test-volume     nfs:       server: my-nfs-server.example.com       path: /my-nfs-volume       readOnly: true`

## Step 7: Create a kustomization.yaml file

In this example of using storage we will utilize `kustomize`, a way of grouping and providing kubernetes resources. It could be seen as the docker compose of kubernetes.

### Add a Secret generator

A [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) is an object that stores a piece of sensitive data like a password or key. Since 1.14, `kubectl` supports the management of Kubernetes objects using a kustomization file. You can create a Secret by generators in `kustomization.yaml`.

Add a Secret generator in `kustomization.yaml` from the following command. You will need to replace `YOUR_PASSWORD` with the password you want to use.

kustomization.yaml 

`secretGenerator: - name: mysql-pass   literals:   - password=YOUR_PASSWORD`

_Click to create kustomization.yaml on **node1**_

Kustomize is a tool for customizing Kubernetes configurations. It has the following features to manage application configuration files:

- generating resources from other sources
- setting cross-cutting fields for resources
- composing and customizing collections of resources

You can find a lot more information in the [Kustomize Documentation](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)

## Step 8: Add resource configs for MySQL and WordPress

The following manifest describes a single-instance MySQL Deployment. The MySQL container mounts the PersistentVolume at `/var/lib/mysql`. The `MYSQL_ROOT_PASSWORD` environment variable sets the database password from the Secret.

We will create a service for the mysl instance

yaml 

`apiVersion: v1 kind: Service metadata:   name: wordpress-mysql   labels:     app: wordpress spec:   ports:     - port: 3306   selector:     app: wordpress     tier: mysql   clusterIP: None`

We take a PVC for our mysql database and request `20Gi` Storage. We also limit the accessMode to `ReadWriteOnce`

yaml 

`apiVersion: v1 kind: PersistentVolumeClaim metadata:   name: mysql-pv-claim   labels:     app: wordpress spec:   accessModes:     - ReadWriteOnce   resources:     requests:       storage: 20Gi`

Next follows the mysql Deployment. Watch for the `volumes` keyword at the end

yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: wordpress-mysql   labels:     app: wordpress spec:   selector:     matchLabels:       app: wordpress       tier: mysql   strategy:     type: Recreate   template:     metadata:       labels:         app: wordpress         tier: mysql     spec:       containers:       - image: mysql:8.0         name: mysql         env:         - name: MYSQL_ROOT_PASSWORD           valueFrom:             secretKeyRef:               name: mysql-pass               key: password         - name: MYSQL_DATABASE           value: wordpress         - name: MYSQL_USER           value: wordpress         - name: MYSQL_PASSWORD           valueFrom:             secretKeyRef:               name: mysql-pass               key: password         ports:         - containerPort: 3306           name: mysql         volumeMounts:         - name: mysql-persistent-storage           mountPath: /var/lib/mysql       volumes:       - name: mysql-persistent-storage         persistentVolumeClaim:           claimName: mysql-pv-claim`

Let's download the combined file:

curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml

_Click to run on **node1**_

The wordpress definition looks somewhat the same. What is different here is that we also use an ingress definition to make our wordpress service available using the ingress controller.

First we again have a service for our application

wordpress-service.yaml 

`apiVersion: v1 kind: Service metadata:   name: wordpress   labels:     app: wordpress spec:   ports:     - port: 80   selector:     app: wordpress     tier: frontend   type: ClusterIP`

_Click to create wordpress-service.yaml on **node1**_

In order to access the application we also need the corresponding ingress

wordpress-ingress.yaml 

`apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: wordpress-ingress spec:   ingressClassName: traefik   rules:   - http:       paths:       - path: /         pathType: Prefix         backend:           service:             name: wordpress             port:                number: 80`

_Click to create wordpress-ingress.yaml on **node1**_

For our wordpress instance we also need `20Gi` Storage

wordpress-pvc.yaml 

`apiVersion: v1 kind: PersistentVolumeClaim metadata:   name: wp-pv-claim   labels:     app: wordpress spec:   accessModes:     - ReadWriteOnce   resources:     requests:       storage: 20Gi`

_Click to create wordpress-pvc.yaml on **node1**_

Again use the PVC inside the deployment

wordpress-deployment.yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: wordpress   labels:     app: wordpress spec:   selector:     matchLabels:       app: wordpress       tier: frontend   strategy:     type: Recreate   template:     metadata:       labels:         app: wordpress         tier: frontend     spec:       containers:       - image: wordpress:6.2.1-apache         name: wordpress         env:         - name: WORDPRESS_DB_HOST           value: wordpress-mysql         - name: WORDPRESS_DB_PASSWORD           valueFrom:             secretKeyRef:               name: mysql-pass               key: password         - name: WORDPRESS_DB_USER           value: wordpress         ports:         - containerPort: 80           name: wordpress         volumeMounts:         - name: wordpress-persistent-storage           mountPath: /var/www/html       volumes:       - name: wordpress-persistent-storage         persistentVolumeClaim:           claimName: wp-pv-claim`

_Click to create wordpress-deployment.yaml on **node1**_

Add them to `kustomization.yaml` file.

cat <<EOF >>./kustomization.yaml
resources:
  - mysql-deployment.yaml
  - wordpress-pvc.yaml
  - wordpress-service.yaml
  - wordpress-deployment.yaml
  - wordpress-ingress.yaml
EOF

_Click to run on **node1**_

With adding the ressources to the `kustomization.yaml` we make sure that kubernetes will create a new secret for us and that it will also establish the reference between the mysql and wordpress deployments to this new secret. This means that we just generate the secret at one point and then multiple deployments can make use of it.

## Step 9: Apply and Verify

The `kustomization.yaml` contains all the resources for deploying a WordPress site and a MySQL database. Now lets bring everything together by applying the configs to our cluster

kubectl apply -k ./

_Click to run on **node1**_

Now you can verify that all objects exist. Verify that the Secret exists by running the following command:

kubectl get secrets

_Click to run on **node1**_

The response should be like this:

NAME                    TYPE                                  DATA   AGE
mysql-pass-c57bb4t7mf   Opaque                                1      9s

PVC:

Look at the two PersistentVolumeClaims we created Also look at the status of the PVC and the linked PV.

Solution

kubectl get pvc

_Click to run on **node1**_

See that two PersistentVolumes got dynamically created.

kubectl get pv

_Click to run on **node1**_

TASK:

After the application is running data is written to the PersistentVolume. Check what data was stored inside the PersistentVolume for wordpress.

In order to get the path where the data was stored you can execute a `kubectl describe` on the PersistentVolume with the wordpress data.

Solution

In order to get the storage path you can execute a describe command.

`kubectl describe pv <pv-name>` where `pv-name` is the Name of the PersistentVolume retreived from `kubectl get pv`

Copy the path that you see under `Source.Path` that looks like `/var/lib/rancher/k3s/storage/pvc-*` and execute `ls` in this path.

### Accessing wordpress

It might take a while for the application to come up. Verify that the Pod is running by running the following command:

kubectl get pods

_Click to run on **node1**_

It can take up to a few minutes for the Pod's Status to be `RUNNING`.

The response should be like this:

NAME                               READY     STATUS    RESTARTS   AGE
wordpress-mysql-1894417608-x5dzt   1/1       Running   0          40s

Verify that the Ingress is running by running the following command:

kubectl get ingress wordpress-ingress

_Click to run on **node1**_

The response should be like this:

NAME                CLASS    HOSTS   ADDRESS         PORTS   AGE
wordpress-ingress   <none>   *       ${vminfo:node1:public_ip}   80      44s

You can now access your wordpress instance following this [link to wordpress](http://$%7Bvminfo:node1:public_ip%7D).

## Step 10: Ephemeral Volumes

Ephemeral volumes in Kubernetes represent a temporary storage mechanism that is tightly coupled with the lifecycle of a Pod. Unlike PersistentVolumes that exist independently of Pod lifecycles and persist data across Pod restarts and deletions, ephemeral volumes are designed to provide lightweight, short-lived storage that is created and destroyed along with the Pod. This type of storage is particularly useful for applications that need a fast storage medium for temporary data processing, caching, or scratch space for computations.

Ephemeral volumes can be configured in several ways, with the most common being `emptyDir`. When a Pod is assigned to a node, Kubernetes creates an `emptyDir` volume for each ephemeral volume defined in the Pod specification. All containers in the Pod can read and write files in the `emptyDir` volume, though the data in an `emptyDir` volume is deleted forever when the Pod is removed from the node for any reason. Despite this, `emptyDir` volumes can be very useful for sharing files between containers in a Pod.

Other types of ephemeral volumes include:

- Ephemeral CSI Volumes: Introduced to allow Container Storage Interface (CSI) drivers to provide ephemeral storage. This is useful for drivers that need to provision lightweight and temporary storage that is automatically provisioned and managed.
- Secrets and ConfigMaps: While their primary purpose is to store sensitive information and configuration data, respectively, they are mounted into Pods as ephemeral volumes. This means that their lifecycle is tied to the Pod lifecycle, providing a secure and dynamic way to manage configuration and secrets.

Example for an ephemeral CSI volume (only `.Spec.volumes`). See that most of the configuration that normally is inside the PVC is defined here.

yml 

 `volumes:     - name: scratch-volume       ephemeral:         volumeClaimTemplate:           metadata:             labels:               type: my-frontend-volume           spec:             accessModes: [ "ReadWriteOnce" ]             storageClassName: "scratch-storage-class"             resources:               requests:                 storage: 1Gi`

Ephemeral volumes are ideal for stateless applications that do not require data to persist beyond the lifetime of a Pod. They offer a simple and secure way to manage temporary data, helping to maintain the cleanliness and efficiency of the Kubernetes node without the need for manual cleanup or complex data persistence strategies.