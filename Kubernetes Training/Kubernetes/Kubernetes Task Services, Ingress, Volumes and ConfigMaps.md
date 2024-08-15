Test your knowledge about services, ingress, configmaps and volumes

## Step 1: Intro

This task will test your knowledge about Services, Ingress, Volumes and ConfigMaps. You should also know about Deployments, Pods and Namespaces.

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Executed on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

## Step 3: The Task

TASK:

Your task is to build a nginx webserver that serves images of cats under the `/cats` route. It should be possible to swap out the images without stopping the server.

We will support you with hints thoughout the progress. You can also try to do it on your own without using the hints. A solution is found on the last page.

You may also look into the Kubernetes documentation found [here](https://kubernetes.io/docs/concepts/)

## Step 4: nginx deployment

TASK:

A good start would be deploying nginx. Go ahead and deploy a simple nginx webserver inside the `catserver` namespace. The name of the deployment should be `catserver` too.

Tip: Use a manifest rather than creating the deployment with imperative commands. We will need to modify this deployment later on.

### Verification

Verify that the following command displays your deployment.

kubectl get deployment catserver -n catserver

_Click to run on **node1**_

### Hints

What image should i use?

Just use `nginx:latest`. In a productive environment we should use a specific version of nginx to prevent automatic upgrades to newer versions when the pod is restarted.

How to generate a deployment manifest?

To generate a deployment manifest you could use the options `--dry-run=client -o yaml` together with any `kubectl create` command. This will print out a simple yaml deployment manifest. You can save this into a file by appending `> manifest.yml` to create a `manifest.yml` that contains the deployment manifest.

Example manifest

Example nginx deployment manifest

yaml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: nginx-deployment   labels:     app: nginx spec:   replicas: 3   selector:     matchLabels:       app: nginx   template:     metadata:       labels:         app: nginx     spec:       containers:       - name: nginx         image: nginx:1.14.2         ports:         - containerPort: 80`

## Step 5: Exposing the catserver

TASK:

The webserver should be accessible on port `80`.

### Verification

Verify that the following command displays the nginx default page.

curl ${vminfo:node1:public_ip}:80

_Click to run on **node1**_

also verify that other routes like `/cats` also point to the nginx webserver. This will result in a 404 returned from nginx, but this is okey for us as we are implementing the route in a future step.

curl ${vminfo:node1:public_ip}:80/cats

_Click to run on **node1**_

### Hints

How to create a service?

Find more info on how to create a service by exeucting

kubectl create service

_Click to run on **node1**_ How to create a ingress?

Find more info on how to create a ingress by exeucting

kubectl create ingress --help

_Click to run on **node1**_ What type should the Service have?

It should be of type `ClusterIP`

How to expose a specific port?

The `kubectl create service` accepts the flag `--tcp=<PORT>` when creating services of type `ClusterIP`

What ingress pathType should be used?

You can use either `Exact` when providing `/cats` as a path directly, or type `Prefix` if you want all traffic redirected to the service. Note that if you use type Exact the verification of the nginx default page will fail.

Example service manifest

This is a simple example manifest of a service

yaml 

`apiVersion: v1 kind: Service metadata:   name: my-service spec:   selector:     app: MyApp   ports:     - protocol: TCP       port: 80       targetPort: 9376`

Example ingress manifest

Ingress manifest example for a ingress that redirects all trafic to a given service.

yaml 

`apiVersion: networking.k8s.io/v1 kind: Ingress metadata:   name: my-ingress spec:   rules:   - http:       paths:       - backend:           service:             name: my-service             port:               number: 1337         path: /         pathType: Exact`

## Step 6: Configuring nginx

TASK:

The webserver should display another page under the `/cats` path. This task shall be solved by also configuring the nginx webserver to not serve anything under the `/` path.

You may use any valid html like the following to be served.

html 

`<h1> Cats are here </h1>`

### Verification:

A curl to `/cats` should now display the new created page.

curl ${vminfo:node1:public_ip}:80/cats

_Click to run on **node1**_

Curling the default `/` path should result in a 404.

curl ${vminfo:node1:public_ip}:80/

_Click to run on **node1**_

### Hints

How does nginx configuration work

nginx relies on a file called `nginx.conf` laying at `/etc/nginx/nginx.conf`

Here is a default nginx configuration file that serves requests to `/dogs` with files from `/usr/share/nginx/html`. A request to `/dogs` will be served with the file `/usr/share/nginx/html/dogs/index.html`. Request to `/dogs/dog.html` will be served with `/usr/share/nginx/html/dogs/dog.html`.

user nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log;
events {
  worker_connections  1024;
}
http {
  server {
      listen       80;
      server_name  localhost;
      location /dogs {
        root   /usr/share/nginx/html;
        index  index.html;
    }
  }
}

How to configure the nginx deployment

We should modify the created deployment manifest and add a ConfigMap containing the nginx configuration as a volume to be mounted at `/etc/nginx/nginx.conf`.

What files does nginx serve?

All files under the configured root path are served. The default path is `/usr/share/nginx/html` Request to `/dog` will be served with `/usr/share/nginx/html/dog/index.html`

## Step 7: Persist the cats

TASK:

Now we need to add some pictures of cats to be served. The images should be interchangable without restarting the webserver. They images should be stored under `/usr/share/nginx/html/cats/images` inside the container

You may download some images of cats from unsplash. For example:

https://images.unsplash.com/photo-1514888286974-6c03e2ca1dba
https://images.unsplash.com/photo-1573865526739-10659fec78a5

### Verification:

The images should be accessible by visiting your webserver on `/cats/images/filename-here.jpg`

### Hints

How to download files in linux?

Make use of `wget`: For example `wget https://example.com/image-123.jpg -O image.jpg` will download a file `image-123.jpg` and saves it locally as `image.jpg`

Where is the PV stored?

You can find out by using `kubectl describe` on the created PV. Look for `Source.Path`

My ConfigMap does not update

Maybe you can fix it by running

`kubectl rollout restart deployment/<deploymentName> -n <namespace>` on your deployment

## Step 8: Solution

WARNING:

This is the solution page, you should try to solve the task yourself or look at the hints before searching for clues here

### The whole thing

All manifests can be found on [GitHub](https://github.com/svalabs/k8s-examples-catserver).

Further on we show you a detailed solution with some comments:

### Creating the nginx deployment

In order to use the `catserver` namespace we need to create it first.

kubectl create namespace catserver

Afterwards we create an example deployment manifest prefilled with the name, the image and the namespace. We save this manifest as `deployment.yml`

kubectl create deployment catserver --image=nginx --dry-run=client -o yaml -n catserver > deployment.yml

We can deploy this by applying the manifest

kubectl apply -f deployment.yml

Verify the deployment is up and running

kubectl get deployment catserver -n catserver

_Click to run on **node1**_

### Exposing the webserver

We are using an service type clusterip. By providing `--tcp=80` we are exposing port `80` of the inside of the container to service port `80`.

kubectl create service clusterip catserver --tcp=80 -n catserver

kubectl get services catserver -n catserver

_Click to run on **node1**_

Example output

NAME        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
catserver   ClusterIP   10.43.17.58   <none>        80/TCP    6s

Create the ingress called `webserver` redirecting all traffic to the service called `catserver` on configured port `80`. The pathType here will be `Prefix` - we could also use the direct path `/cats` for an pathType of `Exact`.

kubectl create ingress catserver --rule="/*=catserver:80" -n catserver

You should be able to visit the [default nginx Page](http://$%7Bvminfo:node1:public_ip%7D:80) now. When you are visiting the `/cats` subpath you will see a 404 Not Found page delivered by our nginx deployment.

### Configure nginx

We create a ConfigMap that contains a nginx configuration file.

nginx-configmap.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: nginx-conf   namespace: catserver data:   nginx.conf: |     user nginx;     worker_processes  1;     error_log  /var/log/nginx/error.log;     events {       worker_connections  1024;     }     http {       server {           listen       80;           server_name  localhost;           location /cats {             root   /usr/share/nginx/html;             index  index.html;         }       }     }`

_Click to create nginx-configmap.yml on **node1**_

Apply the ConfigMap

kubectl apply -f nginx-configmap.yml -n catserver

Another ConfigMap contains the `cats.html` file

cats-configmap.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: cats-html   namespace: catserver data:   index.html: |     <h1> Cats are here </h1>`

_Click to create cats-configmap.yml on **node1**_

We need to add a volumeMount containing the Mountpoint under the template of the container inside the deployment manifest

`.spec.template.spec.containers.[0]`

yaml 

`volumeMounts: - name: nginx-conf   mountPath: /etc/nginx/nginx.conf   subPath: nginx.conf   readOnly: true - name: html-conf   mountPath: /usr/share/nginx/html/cats/index.html   subPath: index.html   readOnly: true`

We also need to define the volume under the template spec.

`.spec.template.spec`

yaml 

`volumes: - name: nginx-conf   configMap:     name: nginx-conf     items:       - key: nginx.conf         path: nginx.conf - name: html-conf   configMap:     name: cats-html     items:       - key: index.html         path: index.html`

Full deployment.yml

deployment.yml 

`apiVersion: apps/v1 kind: Deployment metadata:   creationTimestamp: null   labels:     app: catserver   name: catserver   namespace: catserver spec:   replicas: 1   selector:     matchLabels:       app: catserver   strategy: {}   template:     metadata:       creationTimestamp: null       labels:         app: catserver     spec:       containers:       - image: nginx         name: nginx         resources: {}         volumeMounts:         - name: nginx-conf           mountPath: /etc/nginx/nginx.conf           subPath: nginx.conf           readOnly: true         - name: html-conf           mountPath: /usr/share/nginx/html/cats/index.html           subPath: cats.html           readOnly: true       volumes:       - name: nginx-conf         configMap:           name: nginx-conf           items:            - key: nginx.conf              path: nginx.conf       - name: html-conf         configMap:           name: cats-html           items:            - key: cats.html              path: cats.html`

_Click to create deployment.yml on **node1**_

### Persist the cats

Create a PersistentVolumeClaim

pvc.yml 

`apiVersion: v1 kind: PersistentVolumeClaim metadata:   name: cats-pvc   namespace: catserver spec:   accessModes:     - ReadWriteOnce   storageClassName: local-path   resources:     requests:       storage: 1Gi`

_Click to create pvc.yml on **node1**_

Add the volume under in the created deployment manifest:

`.spec.template.spec.containers.[0].volumeMounts[]`

yaml 

`- name: images   mountPath: /usr/share/nginx/html/cats/images`

`.spec.template.spec.volumes[]`

yaml 

`- name: images   persistentVolumeClaim:     claimName: cats-pvc`

full deployment manifest

deployment.yml 

`apiVersion: apps/v1 kind: Deployment metadata:   creationTimestamp: null   labels:     app: catserver   name: catserver   namespace: catserver spec:   replicas: 1   selector:     matchLabels:       app: catserver   strategy: {}   template:     metadata:       creationTimestamp: null       labels:         app: catserver     spec:       containers:       - image: nginx         name: nginx         resources: {}         volumeMounts:         - name: nginx-conf           mountPath: /etc/nginx/nginx.conf           subPath: nginx.conf           readOnly: true         - name: html-conf           mountPath: /usr/share/nginx/html/cats/index.html           subPath: cats.html           readOnly: true         - name: images           mountPath: /usr/share/nginx/html/cats/images       volumes:       - name: nginx-conf         configMap:           name: nginx-conf           items:            - key: nginx.conf              path: nginx.conf       - name: html-conf         configMap:           name: cats-html           items:            - key: cats.html              path: cats.html       - name: images         persistentVolumeClaim:            claimName: cats-pvc`

_Click to create deployment.yml on **node1**_

Apply the deployment manifest again.

We now need to check if the PersistentVolume is created:

kubectl get pv -n catserver

_Click to run on **node1**_

We should see a PV created for us. We need to see where this PV is stored. Execute `kubectl describe pv <PV-NAME> -n catserver` And see the path under Source.Path starting with `/var/lib/rancher/k3s/storage`

Now we copy our images inside to this path. For example

`cp cat1.jpg /var/lib/rancher/k3s/storage/pvc-abc123-abc123_catserver_cats-pvc/cat1.jpg`

We did this for both of our images.

Test that the image is served by nginx by visiting `/cats/images/cat1.jpg` in your browser. Now we need to modify the ConfigMap containing the html to serve the images:

yaml 

`apiVersion: v1 kind: ConfigMap metadata:   name: cats-html   namespace: catserver data:   cats.html: |     <h1> Cats are here </h1>     <img src="/cats/images/cat1.jpg" width="500px" height="auto">     <img src="/cats/images/cat2.jpg" width="500px" height="auto">`

After changing the configmap the deployment should automatically change to deliver the modified file that delivers us a nice website displaying our cat. You may need to restart the pod manually

`kubectl rollout restart deployment/catserver -n catserver`