Learn to use configmaps in kubernetes

## Step 1: Intro

Kubernetes ConfigMaps are a way to store and manage configuration data in Kubernetes clusters. ConfigMaps are used to separate configuration data from container images, which makes it easier to manage configuration across multiple environments.

ConfigMaps can be created using YAML or JSON files and can contain key-value pairs, directories, or files. These configurations can be injected into Kubernetes pods as environment variables or mounted as volumes.

### Environment variables

Rather than hardcoding environment variables into container images, they can be set as ConfigMaps and injected into the container as environment variables at runtime. This makes it easier to change configuration data without rebuilding the container image.

### Configuration files

Another use case for ConfigMaps is to store configuration files for applications. For example, a web application might require a configuration file with settings for a database connection. This file can be stored as a ConfigMap and mounted as a volume in the pod running the web application.

### Other purposes

ConfigMaps can also be used to store configuration data for Kubernetes objects such as services and deployments. This allows for central management of configuration data across the entire Kubernetes cluster.

In summary, ConfigMaps provide a flexible and easy way to manage configuration data in Kubernetes clusters, making it easier to maintain and manage applications running in Kubernetes.

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

## Step 3: Manifest

In this example, we're creating a ConfigMap named `my-configmap` with a single key-value pair. The key is `db_url` and the value is `my-db-url`.

configmap.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: my-configmap data:   db_url: my-db-url`

_Click to create configmap.yml on **node1**_

Apply the ConfigMap:

After clicking on the click-to-file the `configmap.yml` should be created for you on `node1`. Go ahead and apply the manifest of the ConfigMap we just created.

Verify the creation:

kubectl get cm my-configmap

_Click to run on **node1**_

Solution

Go ahead and apply the manifest as known before:

kubectl apply -f configmap.yml

_Click to run on **node1**_

## Step 4: ConfigMaps as Environment Variable

In this step, we're mounting the previously created ConfigMap named `my-configmap` and expose the `db_url` key as an environment variable named `DB_URL` in the container.

pod-env.yml 

`apiVersion: v1 kind: Pod metadata:   name: my-pod-env spec:   containers:   - name: my-container     image: nginx     env:     - name: DB_URL       valueFrom:         configMapKeyRef:           name: my-configmap           key: db_url`

_Click to create pod-env.yml on **node1**_

Remember that environment variables could be referenced under the `spec.env` key. The `valueFrom` key together with `configMapKeyRef` reference our configmap based on the ConfigMap name. We also have to provide the key `db_url` to reference the correct datapoint.

Create the pod:

After clicking on the click-to-file the `pod-env.yml` should be created for you on `node1`. Go ahead and apply the manifest of the ConfigMap we just created.

Verify the creation of the pod and that it is running:

kubectl get pod my-pod-env

_Click to run on **node1**_

Solution

kubectl apply -f pod-env.yml

_Click to run on **node1**_

Verify the environment variable:

Once the pod is running we can verify that the environment variable `DB_URL` is set inside the pod `my-pod-env`. Do so by executing the `env` command.

INFO:

The `env` command displays all exported environment variables

Hint

Remember that the `kubectl exec` command can be used to execute commands inside a pod. The syntax is `kubectl exec [POD] -- [COMMAND]`

The `env` command will give us all environment variables. We can filter output of commands using the `grep` command.

Solution

kubectl exec my-pod-env -- env | grep DB_URL

_Click to run on **node1**_

## Step 5: ConfigMaps mounted as a file

In this example, we're mounting a ConfigMap as a volume in the container at the path `/etc/config`. The ConfigMaps data will be available as files in the mounted volume.

my-configmap-files.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: my-configmap-files data:   config.yaml: |     database:       host: my-db-host       port: 5432       user: my-db-user       password: my-db-password   other-file.yaml: |     more: files`

_Click to create my-configmap-files.yml on **node1**_

pod-volume.yml 

`apiVersion: v1 kind: Pod metadata:   name: my-pod-volume spec:   containers:   - name: my-container     image: nginx     volumeMounts:     - name: config-volume       mountPath: /etc/config   volumes:   - name: config-volume     configMap:       name: my-configmap-files`

_Click to create pod-volume.yml on **node1**_

See that we created a volume named `config-volume` inside the manifest. It references our ConfigMap `my-configmap-files` under the `configMap` key. The Mounting of the volume is done with `volumeMounts` just like normal Volumes are mounted.

Verify the file existence:

Go ahead and apply the ConfigMap aswell as the Pod. Afterwards check the existence and the content of the files at `/etc/config`

Solution

First we need to apply both manifests:

kubectl apply -f my-configmap-files.yml
kubectl apply -f pod-volume.yml

_Click to run on **node1**_

In order to verify the existence of the two files we can see what the `ls` command displays for the given path.

kubectl exec my-pod-volume -- ls /etc/config 

_Click to run on **node1**_

We can also verify the content of the mounted files.

kubectl exec my-pod-volume -- cat /etc/config/config.yaml

_Click to run on **node1**_

## Step 6: Updating ConfigMaps (1)

Updating configurations should be possible without releasing a new image of the application. In order to test how updating ConfigMaps differs from environment variables to volumes we adapt some changes to the previously created ConfigMaps.

### Updating environment variables

Update the configmap for the environment variable. Note that we have changed the value of `db_url`from `my-db-url`to `my-new-db-url`.

configmap.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: my-configmap data:   db_url: my-new-db-url # This was "my-db-url"`

_Click to create configmap.yml on **node1**_

After updating the ConfigMap manifest, apply it to the cluster.

kubectl apply -f configmap.yml

_Click to run on **node1**_

Let`s see what value the pod gives us for the environment variable:

kubectl exec my-pod-env -- env | grep DB_URL

_Click to run on **node1**_

Oh no!:

The value is still `my-db-url` instead of `my-new-db-url`. How did that happen? Environment variables can not be updated on the fly. We have to restart the pod in order for the ConfigMap update to come into effect.

kubectl delete pod my-pod-env
kubectl apply -f pod-env.yml

_Click to run on **node1**_

When the pod is running test the value of the variable again, which should print the new value `my-new-db-url`

kubectl exec my-pod-env -- env | grep DB_URL

_Click to run on **node1**_

In a real-life scenario we could also make use of the `kubectl rollout restart deployment <deployment>` command.

### Updating volumes

Modify the configmap that is mounted by the pod `my-pod-volume` by clicking the following file. Note that we changed `my-db-host` to `my-new-db-host` and so on. We also added a new line to the `other-file.yaml`.

my-configmap-files.yml 

`apiVersion: v1 kind: ConfigMap metadata:   name: my-configmap-files data:   config.yaml: |     database:       host: my-new-db-host       port: 5432       user: my-new-db-user       password: my-new-db-password   other-file.yaml: |     more: files     update: in-place`

_Click to create my-configmap-files.yml on **node1**_

Apply the changes and let`s watch the mounted files being updated::

kubectl apply -f my-configmap-files.yml
watch kubectl exec my-pod-volume -- cat /etc/config/config.yaml

_Click to run on **node1**_

Wait until you see the updated content. (`my-new-db-host` etc.). This may take some seconds.

Exiting the watch command:

Press `Ctrl + C` to exit the `watch` command.

Huh!?:

The files are automatically updated! How did that happen? Files mounted in volumes are updated when the underlying ConfigMap is updated! The Pod does not need to be restarted. We can verify that the pod was not restarted by running

kubectl get pod my-pod-volume

_Click to run on **node1**_

## Step 7: Conclusion

Overall, ConfigMaps provide a flexible and powerful way to manage configuration data in Kubernetes clusters. By separating configuration data from container images, you can make your applications more portable and easier to manage across multiple environments.