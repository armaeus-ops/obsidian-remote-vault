Learn to use secrets in kubernetes

## Step 1: Introduction about secrets

Kubernetes provides a powerful and flexible way to manage secrets - sensitive data such as passwords, tokens, and encryption keys that your application needs to function. Secrets in Kubernetes are designed to be secure and easy to manage, allowing you to store and distribute sensitive data to your pods without exposing it to unauthorized users or systems.

In this lab, we'll explore the basics of using secrets in Kubernetes. We'll start by creating a simple generic secret, which can be used to store any kind of sensitive data which can be mounted as a file inside the container or be exposed in a environment variable. Along the way, we'll also learn how to manage TLS secrets and how authentication against private image registries can be achieved.

By the end of this lab, you'll have a solid understanding of how to use secrets in Kubernetes to secure your applications and manage sensitive data.

## Step 2: Types of secrets

There are severaly types of secrets for many usecases:

1. `opaque`: A generic secret type that is used to store plain sensitive data. Opaque is also the default type.
    
2. `docker-registry`: This type of secret is used to store credentials for accessing private Docker registries. It includes fields for the registry server URL, username, and password. Docker registry secrets are used by Kubernetes when pulling images from a private registry.
    
3. `tls`: This type of secret is used to store TLS certificates and private keys. It includes fields for the certificate and key data, and can be used to secure connections to your pods or to external services.
    
4. `ssh`: This type of secret is used to store SSH keys. It includes fields for the private key data and the public key fingerprint, and can be used to access remote systems securely from your pods.
    
5. `service-account-token`: This type of secret is automatically created for each service account in your cluster, and it contains a token that can be used to authenticate requests to the Kubernetes API.
    

To create a secret of any type we can use `kubectl create secret <type> <name> <data>`. We will make use of this command in the next steps.

## Step 3: Security aspect of secrets

Secrets are stored unencrypted:

Kubernetes Secrets are, by default, stored unencrypted in the API server's underlying data store (etcd). Anyone with API access can retrieve or modify a Secret, and so can anyone with access to etcd. Additionally, anyone who is authorized to create a Pod in a namespace can use that access to read any Secret in that namespace; this includes indirect access such as the ability to create a Deployment.

To overcome this issues some steps should be taken:

1. Configure RBAC Roles
2. Restrict Secret access to specific containers
3. Use external secret store providers like Hashicorp Vault. GCP, AWS and Azure also each have a secret store provider.
4. Enable etcd encryption to ensure that the etcd stores data secure

Compared to ConfigMaps many things are the same for Secrets but kubernetes still applies some actions differently when it comes to secrets.

When a Pod running on a Kubernetes node requires access to a Secret, the Secret is only then sent to that node. To prevent the confidential data from being written to durable storage, the kubelet creates a copy of the Secret's data in a temporary file system (tmpfs) and mounts it into the Pod.

When the Pod is deleted, the kubelet automatically removes its local copy of the confidential data from the Secret. This ensures that the sensitive data is not retained on the node and remains secure.

Privileged Containers:

Containers that run in privileged mode can access all Secrets used on the node they are running on. In most cases containers do not require to be run in privileged mode.

## Step 4: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

## Step 5: Opaque Secrets

First of all we will cerate a secret of type `Opaque`

Creating Opaque secrets:

Opaque secrets will be created using the keyword `generic` when using `kubectl create secret`

kubectl create secret generic my-secret --from-literal=username=myuser --from-literal=password=mypassword

_Click to run on **node1**_

Verify that the secret was created.

kubectl get secrets

DATA:

The `DATA` column shows the number of data items stored in the Secret. In this case, 2 means you have created an secret with two key/value pairs (username and password).

See what the describe command says about this secret. Note that the sensitive data is not being displayed directly - only the size of the data is being displayed.

kubectl describe secret my-secret

_Click to run on **node1**_

However if we retreive the secret as yaml we will see the sensitive data although it is stored base64 encoded.

kubectl get secret my-secret -o yaml

_Click to run on **node1**_

Encoding is not encryption:

Note that encoding does not equal encryption. We can decode this data again into plain text without any key. It provides no additional confidentiality over plain text.

Execute the following to see the password being printed in plain text again.

echo 'bXlwYXNzd29yZA=='  | base64 -d; echo 

_Click to run on **node1**_

You can enable [encrypting data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) to make secrets safer in production.

To use a secret of a specific type in your pods, you can refer to the secret using the appropriate `secretKeyRef` field in your pod specification. For example, to mount a generic secret as an environment variable in your container, you can use the following specification:

pod-generic.yml 

`apiVersion: v1 kind: Pod metadata:   name: my-generic-pod spec:   containers:   - name: generic-container     image: nginx     env:     - name: MY_USERNAME       valueFrom:         secretKeyRef:           name: my-secret           key: username     - name: MY_PASSWORD       valueFrom:         secretKeyRef:           name: my-secret           key: password`

_Click to create pod-generic.yml on **node1**_

Verify the secrets:

Go ahead and apply the pod in the `pod-generic.yml` after creating it. Verify that the secret is available as environment variables.

INFO:

The `env` command displays all exported environment variables. `echo $VARIABLE` will output the value of the environment variable named `VARIABLE`.

Solution

First we apply the created manifest `pod-generic.yml`

kubectl apply -f pod-generic.yml

_Click to run on **node1**_

Verify the pod is running

kubectl get pods

_Click to run on **node1**_

When the pod is running execute inside the pod

kubectl exec -it my-generic-pod -- /bin/bash

_Click to run on **node1**_

Inside the pod verify the environment variables `MY_USERNAME` and `MY_PASSWORD` are set to the values we gave to the secret.

echo $MY_USERNAME
echo $MY_PASSWORD

_Click to run on **node1**_

or

env

_Click to run on **node1**_

Log out of the pod using `exit`.

Using files instead of literals:

We can also mount files or whole directories stored in secrets.

See the output of `kubectl create secret generic --help` for more info

# Create a new secret named my-secret with keys for each file in folder bar
kubectl create secret generic my-secret --from-file=path/to/bar

## Step 6: TLS Secrets

In addition to the opaque secret, Kubernetes also supports a tls type that can be used to store TLS certificates and private keys. This can be useful for securing connections between your pods and external services.

First we will create some dummy TLS data

openssl req -x509 -newkey rsa:4096 -nodes -keyout tls.key -out tls.crt -subj "/CN=my-domain.com"

_Click to run on **node1**_

This command generates a 4096-bit RSA private key (`tls.key`) and a self-signed X.509 certificate (`tls.crt`) with a subject of `/CN=my-domain.com`. You can replace `my-domain.com` with your own domain name or IP address.

self-signed certificates:

Note that this self-signed certificate should only be used for testing and development purposes, and not for production environments where a trusted certificate authority should be used.

We will then create a secret with this certificate and key.

kubectl create secret tls my-tls-secret --cert=tls.crt --key=tls.key

_Click to run on **node1**_

Verify the creation

kubectl get secrets

_Click to run on **node1**_

Describe the created secret to see the certificate and key stored inside.

kubectl describe secret my-tls-secret

_Click to run on **node1**_

This following pod will mount the `my-tls-secret` secret as a read-only volume at the path `/etc/tls` inside the container.

pod-tls.yml 

`apiVersion: v1 kind: Pod metadata:   name: my-tls-pod spec:   containers:   - name: tls-container     image: nginx     volumeMounts:     - name: tls-certs       mountPath: /etc/tls       readOnly: true   volumes:   - name: tls-certs     secret:       secretName: my-tls-secret`

_Click to create pod-tls.yml on **node1**_

Verify the TLS data:

Go ahead and apply the pod in the `pod-tls.yml` after creating it. Verify that the TLS certificate is available at `/etc/tls` inside the container.

Solution

Apply the pod after creating the manifest:

kubectl apply -f pod-tls.yml

_Click to run on **node1**_

View the TLS data mounted inside the container

kubectl exec -it my-tls-pod -- /bin/bash
ls /etc/tls
cat /etc/tls/tls.crt
cat /etc/tls/tls.key
exit

_Click to run on **node1**_

## Step 7: Image pull secrets

INFO:

This step only provides information about authentication against private registries. There are no manual hands-on involved here.

Image pull secrets are used to authenticate with a container registry when pulling images for a Kubernetes deployment. The kubelet is the component of the Kubernetes node that is responsible for managing and running the pods.

When a pod is scheduled on a node, the kubelet pulls the required container images from the specified container registry. If the container registry requires authentication, the kubelet uses the credentials stored in an image pull secret to authenticate with the registry.

The type `docker-registry` allows you to specify the Docker registry URL, username, and password in the secret data.

Example creation of a pull secret

kubectl create secret docker-registry <name> \
  --docker-server=DOCKER_REGISTRY_SERVER \
  --docker-username=DOCKER_USER \
  --docker-password=DOCKER_PASSWORD \
  --docker-email=DOCKER_EMAIL

Once the image pull secret is created, you can specify it in the pod specification by adding the `imagePullSecrets` field to the pod's `.spec` section. This tells the kubelet to use the specified image pull secret to authenticate with the container registry.

INFO:

Pods can only reference image pull secrets in their own namespace, so this process needs to be done one time per namespace.

example manifest of a pod using the pull secret

yml 

`apiVersion: v1 kind: Pod metadata:   name: foo spec:   containers:     - name: foo       image: my-image   imagePullSecrets:     - name: my-pull-secret`

## Step 8: Secret providers

External secret providers are third-party tools or services that allow you to store secrets outside of Kubernetes. Examples include [HashiCorp Vault](https://www.vaultproject.io/docs/) and [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/). They provide additional security benefits and can simplify secrets management in complex environments.

However, it's important to carefully evaluate the security and compliance implications of using external secret providers, and to ensure that the provider you choose is compatible with your organization's security policies and regulatory requirements.

## Step 9: Conclusion

Congratulations, you have completed the Kubernetes Secrets lab!

In this lab, you learned how to create, manage, and use secrets in Kubernetes. You learned about the different types of secrets available, including Opaque, TLS secrets and image pull secrets. You also learned about the importance of securing secrets and the use of external secret providers.

Now that you have a better understanding of how to manage secrets in Kubernetes, you can use this knowledge to secure your own applications running in Kubernetes. Remember to always follow security best practices and to regularly review and rotate your secrets to reduce the risk of compromise.

### References

- [Kubernetes Secrets documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
    
- [HashiCorp Vault documentation](https://www.vaultproject.io/docs/)
    
- [AWS Secrets Manager documentation](https://aws.amazon.com/secrets-manager/)