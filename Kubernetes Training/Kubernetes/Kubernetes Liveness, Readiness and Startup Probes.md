Learn about Liveness, Readiness and Startup Probes and how to configure them. We will also cover basic usage of restart policies for pods.

## Step 1: Intro

When working with Kubernetes, it's essential to ensure that your applications are running smoothly and efficiently.

Kubernetes probes are diagnostic tools that help monitor the health and readiness of your containers. There are three types of probes: Liveness, Readiness, and Startup probes. Each serves a distinct purpose in assessing the state of your containers:

1. `Liveness Probe`: This probe checks if your container is alive and running. If the liveness probe fails, Kubernetes will restart the container, ensuring your application remains responsive.
2. `Readiness Probe`: This probe determines if your container is ready to accept traffic. If the readiness probe fails, Kubernetes stops sending traffic to the container, preventing requests from overwhelming a container that isn't prepared to handle them.
3. `Startup Probe`: This probe assesses if your container has started correctly. It's particularly useful for slow-starting containers, allowing them ample time to initialize before being subjected to liveness or readiness probes.

In this tutorial, we'll cover the fundamentals of each probe type, their configurations, and how to implement them in your Kubernetes deployments. By the end of this tutorial, you will be well-equipped to leverage Kubernetes probes for better application reliability, health, and performance. So, let's get started!

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

In this lab we are building a container image. We need to install Docker:

curl -fsSL get.docker.com | bash

_Click to run on **node1**_

## Step 3: Liveness Probes

### Step 1: Create a simple web application

Let's create a basic Python web application using Flask. Create the two files: `app.py` and `requirements.txt`.

app.py 

`from flask import Flask import os import time  app = Flask(__name__) start_time = time.time()  @app.route('/') def hello():     return "Hello, Kubernetes!"  @app.route('/healthz') def health_check():     elapsed_time = time.time() - start_time     if elapsed_time > 60:         return "FAIL", 500     return "OK", 200  if __name__ == "__main__":     app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))`

_Click to create app.py on **node1**_

requirements.txt 

`Flask==2.1.1 Werkzeug==2.2.2`

_Click to create requirements.txt on **node1**_

This simple web application responds with "Hello, Kubernetes!" when accessed at the root path and returns an `OK` status with a 200 HTTP response code at the `/healthz` endpoint. We return a 500 HTTP status code if the elapsed time is greater than 60 seconds, causing the health check to fail. We will use Python's time module to track the elapsed time since the application started.

### Step 2: Create a Dockerfile

Next, create a `Dockerfile` to containerize our web application:

Dockerfile 

`FROM python:3.9-slim  WORKDIR /app  COPY requirements.txt requirements.txt RUN pip install -r requirements.txt  COPY . .  CMD ["python3", "app.py"]`

_Click to create Dockerfile on **node1**_

Build and push the Docker image:

docker build -t push.svahub.io/-flask-k8s-liveness .
docker push push.svahub.io/-flask-k8s-liveness

_Click to run on **node1**_

### Step 3: Configure a Liveness probe

Now, we'll create a Kubernetes deployment manifest that includes a Liveness probe. Create a file named `deployment.yaml`:

deployment.yml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: flask-k8s-liveness spec:   replicas: 1   selector:     matchLabels:       app: flask-k8s-liveness   template:     metadata:       labels:         app: flask-k8s-liveness     spec:       containers:       - name: flask-k8s-liveness         image: push.svahub.io/-flask-k8s-liveness:latest         imagePullPolicy: Always         ports:         - containerPort: 8080         livenessProbe:           httpGet:             path: /healthz             port: 8080           initialDelaySeconds: 5           failureThreshold: 3           periodSeconds: 10`

_Click to create deployment.yml on **node1**_

In this manifest, we've defined a Liveness probe using an HTTP GET request to the `/healthz` endpoint. The probe will start checking after an initial delay of 5 seconds and continue to check every 10 seconds.

### Step 4: Deploy the application

Apply the deployment configuration to your Kubernetes cluster:

kubectl apply -f deployment.yml

_Click to run on **node1**_

kubectl expose deployment flask-k8s-liveness --type=ClusterIP --port=8080 --target-port=8080

_Click to run on **node1**_

Now, you can access the application using the service's cluster IP:

Find your service's IP:

kubectl get services

_Click to run on **node1**_

TASK:

Go ahead and curl against the service at port `8080` at the path `/healthz` until the check fails with `FAIL`.

`watch curl <service-ip>:8080/healthz`

TASK:

Desribe the pod:

kubectl describe pod

_Click to run on **node1**_

See that the Event `Liveness probe failed: HTTP probe failed with statuscode: 500` was thrown.

Watch the pod being restarted after 3 failed checks (~90 seconds after the start.)

watch kubectl get pods

_Click to run on **node1**_

Press `Ctrl` + `C` to quit the `watch` command

In this example, we've demonstrated how to implement a Liveness probe in a Kubernetes deployment to monitor the health of a containerized Flask application. By using probes, you can ensure that your application remains responsive and recovers automatically from any issues.

## Step 4: Readiness Probes

In this example, we'll demonstrate how to use a Readiness probe with the exec command in the same Flask application we created earlier. We'll modify the application to create a temporary file after a certain delay (e.g., 15 seconds) to simulate the time it takes for the container to be ready to accept traffic. The Readiness probe will use the exec command to check for the presence of the temporary file. When the file is found, the Readiness probe will pass, signaling that the container is ready to receive traffic.

Update your `app.py` to create a temporary file named `ready` after a delay of 15 seconds:

app.py 

`from flask import Flask import os import time import threading  app = Flask(__name__) start_time = time.time()  def create_temp_file():     time.sleep(15)     with open("ready", "w") as f:         f.write("ready")  @app.route('/') def hello():     return "Hello, Kubernetes!"  # Start a separate thread to create the temporary file file_creator = threading.Thread(target=create_temp_file) file_creator.start()  if __name__ == "__main__":     app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 8080)))`

_Click to create app.py on **node1**_

Update the Kubernetes Deployment configuration: Add the Readiness probe to the container configuration in your `deployment.yaml` file. Instead of `httpGet`, we will use the `exec` command to check for the presence of the ready file:

deployment.yml 

`apiVersion: apps/v1 kind: Deployment metadata:   name: flask-k8s-readiness spec:   replicas: 1   selector:     matchLabels:       app: flask-k8s-readiness   template:     metadata:       labels:         app: flask-k8s-readiness     spec:       containers:       - name: flask-k8s-readiness         image: push.svahub.io/-flask-k8s-readiness:latest         imagePullPolicy: Always         ports:         - containerPort: 8080         readinessProbe:           exec:             command:             - sh             - -c             - 'test -f /app/ready'           periodSeconds: 5`

_Click to create deployment.yml on **node1**_

This Readiness probe will run the `test -f /app/ready` shell command to check if the `ready` file exists in the `/app` directory. The probe will start after an initial delay of 5 seconds and continue to check every 5 seconds.

Rebuild and redeploy the application:

TASK:

Rebuild the Docker image, push it to the Docker registry, and apply the updated deployment configuration: Watch the Pod going into status READY `1/1` after ~15 seconds.

docker build -t push.svahub.io/-flask-k8s-readiness:latest .
docker push push.svahub.io/-flask-k8s-readiness:latest
kubectl apply -f deployment.yml
watch kubectl get pods

_Click to run on **node1**_

With these changes, the Readiness probe will use the `exec` command to check for the presence of the `ready` file in the container. The container will be marked as ready and start receiving traffic once the temporary file is created (15 seconds after the container starts). This demonstrates how you can use a Readiness probe with the `exec` command to control when a container should start accepting traffic in a Kubernetes deployment.

## Step 5: Startup Probes

Startup probes are used to check when a container application has started correctly. These probes are helpful for applications that can take a significant amount of time to start and initialize before becoming ready to serve traffic. A Startup probe allows you to delay other probes, such as Liveness and Readiness probes, until the application has successfully started.

You would combine the startup probe with a liveness probe like the following:

yml 

`livenessProbe:   httpGet:     path: /healthz   failureThreshold: 1   periodSeconds: 10  startupProbe:   httpGet:     path: /healthz   failureThreshold: 30   periodSeconds: 10`

This would allow the container to have a startup time of 5 Minutes (30 * 10 seconds) before the liveness probe is trying to query the endpoint. If the startup probe succeeds before this 5 minutes, the liveness probe starts early. If the startup probe never succeeds the pod is killed after 300 seconds.

## Step 6: Handle failing probes

What happens when the liveness probe fails? In the last examples we encountered that the pod is restarted after the given `failureThreshold` is reached.

The `restartPolicy` field in a Kubernetes Pod specification determines the behavior of containers within the pod when they exit or encounter a failure. It helps control how and when the containers are restarted. There are three possible values for `restartPolicy`:

1. `Always`: This is the default restart policy for a Pod. When the `restartPolicy` is set to `Always`, the containers in the Pod are restarted every time they exit, regardless of the exit status. This means that even if the container exits successfully (with a 0 exit code), it will still be restarted.
2. `OnFailure`: When the `restartPolicy` is set to `OnFailure`, the containers in the Pod are only restarted if they exit with a non-zero status (i.e., they encountered a failure). If the container exits successfully with a 0 exit code, it will not be restarted.
3. `Never`: When the `restartPolicy` is set to `Never`, the containers in the Pod are never restarted, regardless of their exit status. Once a container exits, it remains terminated.

For example, to set the `restartPolicy` to `OnFailure` for a Pod, you can add the following in the Pod specification:

yaml 

`apiVersion: v1 kind: Pod metadata:   name: my-pod spec:   restartPolicy: OnFailure   containers:   - name: my-container     image: my-image`

Please note that the `restartPolicy` is applicable at the Pod level, and not the container level. It applies to all containers within the Pod. Also, keep in mind that when using higher-level Kubernetes objects like Deployments, StatefulSets, or DaemonSets, the `restartPolicy` is automatically set to Always, as these controllers manage the desired state and ensure the specified number of replicas are running.

## Step 7: Conclusion

In conclusion, this hands-on tutorial covered essential Kubernetes probes - Liveness, Readiness, and Startup probes - and their use cases. We discussed how to create and configure probes using various methods, such as `httpGet` and `exec`. We also demonstrated how to set up and troubleshoot these probes within a Flask application running in a Kubernetes cluster.

Additionally, we touched upon the `restartPolicy` field in Kubernetes Pod specifications, which determines the container restart behavior within a Pod.

By understanding and effectively using these Kubernetes features, you can ensure that your applications are monitored and managed efficiently in a Kubernetes environment. Probes help maintain the health and availability of your applications, while the restart policy allows you to control container restart behavior based on specific requirements. Together, these features contribute to the robustness and resilience of your Kubernetes deployments, ultimately leading to more reliable and better-performing applications.

### More information

See [the Kubernetes documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) about probes for more information.