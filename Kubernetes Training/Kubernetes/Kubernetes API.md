Learn to interact with the Kubernetes API

## Step 1: Introduction to the Kubernetes API

![Kubernetes architecture](https://sva-academy.s3.eu-central-1.amazonaws.com/assets/k8s/k8s_scheduling/01.png)

The Kubernetes API is a core component of the Kubernetes platform that enables users to manage and control Kubernetes clusters programmatically. In the above diagram, you can see that the Kubernetes API server is the central point of communication for practically all k8s components. You as a user also (unwittingly) send commands to the API server, when using Kubernetes command line tools.

The Kubernetes API provides a standardized interface for creating, updating, and deleting Kubernetes resources such as pods, services, and deployments, as well as querying the state of the cluster.

It is designed to be extensible, making it easy to integrate with third-party tools and create custom resources and controllers. By using the Kubernetes API, users can automate workflows and streamline the management of their Kubernetes cluster.

Overall, the k8s API is a powerful tool for managing and controlling Kubernetes clusters, providing a programmatic interface that enables users to interact with the platform in a flexible and customizable way.

When building tools around the Kubernetes platform, it is quite important to have a certain proficiency in the geography of its API landscape. For this particular reason, let us look at ways to explore and work with API objects directly, with and without Kubernetes client tools.

By the end of this lab, you will have a strong understanding of the Kubernetes API and how to interact with it using different methods. You will be able to use the Kubernetes API to manage Kubernetes resources and automate workflows to make your cluster more efficient and scalable.

## Step 2: Why is the API Important?

Everything in Kubernetes revolves around the API. Without it, nothing would work. You may not realize it, but nearly everything you do when managing your cluster interacts with the API. Even if you're not deep in the roots of your cluster, internal services are communicating with each other, making sure that your cluster is healthy.

You can think of Kubernetes as an all-encompassing service that consists of several microservices. There are multiple components that need to communicate with each other in order to carry out specific tasks. Rather than building a specific API purely for one purpose, the creators of Kubernetes decided to create a general API, making it easier for people to contribute and expand on their clusters. You can read more about the components of Kubernetes here.

## Step 3: Creating, Reading, Watching, Updating, Patching and Deleting objects

The Kubernetes API supports the following [operations](https://github.com/kubernetes/community/blob/7f3f3205448a8acfdff4f1ddad81364709ae9b71/contributors/devel/sig-architecture/api-conventions.md#verbs-on-resources) on various kinds of Kubernetes objects:

1. GET `/<resourcePlural>` - Retrieve a list of `<resourceName>`
    
2. POST `/<resourcePlural>` - Create a new resource from the JSON object provided by the client
    
3. GET `/<resourcePlural>/<name>` - Retrieves a single resource with the given `<name>`
    
4. DELETE `/<resourcePlural>/<name>` - Delete the single resource with the given `<name>`
    
5. DELETE `/<resourcePlural>` - Deletes a list of `<resourceName>`
    
6. PUT `/<resourcePlural>/<name>` - Update or create the resource with the given name with the JSON object provided by client
    
7. PATCH `/<resourcePlural>/<name>` - Selectively modify the specified fields of the resource
    
8. GET `/<resourcePlural>?watch=true`- Receive a stream of JSON objects corresponding to changes made to any resource of the given kind over time
    

API:

The API is RESTful, so the above mapping of HTTP methods on the resource actions should look familiar.

Even though the documentation only mentions JSON objects, submitting YAML payloads is supported if the `Content-Type` header is set to `application/yaml`.

## Step 4: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Install [jq](https://stedolan.github.io/jq/) for improved json parsing:

apt-get update && apt-get -y install jq

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

## Step 5: Accessing the API with kubectl

Basically, every `kubectl` command issues API calls in the background. It may be a rare occurrence, but there will be times when you need to make a direct call to the Kubernetes API server to issue some command directly. Or maybe you are just curious about the REST API and what is happening under the covers.

The [Kubernetes documentation](https://kubernetes.io/docs/home/) does a great job on showing the various commands available and also how to access the API. But how do you start discovering the different API calls for different operations?

The answer is: by using `kubectl`! You can add verbose logging levels to get the neccessary API calls.

KUBECTL OUTPUT VERBOSITY:

The verbosity of `kubectl` output is controlled with the `-v` flag, followed by an integer that represents the intended log level (e.g. `-v=8`).

The interesting log levels are:

1. `-v=4`: Debug level verbosity
    
2. `-v=5`: Trace level verbosity
    
3. `-v=6`: The requested resources are visible as API calls
    
4. `-v=7`: Additional to `-v=6`, HTTP request headers are visible
    
5. `-v=8`: Additional to `-v=7`, HTTP request contents are visible
    

Let us take a closer look and see this in action. Say you want to get the Kubernetes API call to get all of the pods in the `kube-system` namespace:

kubectl get pods -n kube-system -v=6

_Click to run on **node1**_

You see the requested information, as well as the API call, which can be found in the second line of output. Now you know, the API call to get all pods in the `kube-system` namespace is a `GET` request on `api/v1/namespaces/kube-system/pods`.

A more complicated call:

Let's take a look at another task:

Show all resources, in every namespace, with the label `k8s-app=kube-dns` with a verbosity level of `6`:

kubectl get all -l k8s-app=kube-dns -A -v=6

_Click to run on **node1**_

Looking at the output, you can see several API calls, which are `GET`requests on different paths like `api/v1/services?labelSelector=k8s-app%3Dkube-dns` or `apis/apps/v1/deployments?labelSelector=k8s-app%3Dkube-dns`

Now you know how to query for certain resources, but what if you want to create or modify a Kubernetes resource? You can still do the same thing as above, but there is a good chance you don't want kubectl doing the creation or modification. You might just want kubectl to tell you the API call, but don't do it. We can add the parameter `--dry-run=server` to get this information without modifying anything.

Imitate creating objects:

Another example: Find out the API call to create a namespace, without creating the namespace.

Imitate creating a Kubernetes `namespace` called `test`, with a verbosity level of `8`.

You will see in the output that it is a `POST` request to `api/v1/namespaces?fieldManager=kubectl-create&fieldValidation=Strict` with the request body of `{"apiVersion":"v1","kind":"Namespace","metadata":{"creationTimestamp":null,"name":"test"},"spec":{},"status":{}}` (you will receive more information on what this means in specific on page 7/9).

Lastly check that indeed no namespace called `test` was created.

Solution

Imitate creation:

kubectl create namespace test -v=8 --dry-run=server

_Click to run on **node1**_

Check that namespace exists:

kubectl get namespace

_Click to run on **node1**_

## Step 6: Accessing Kubernetes API using kubectl raw mode

[kubectl](https://kubernetes.io/docs/reference/kubectl/) is a pretty advanced tool, and even simple commands like `kubectl get` have a non-trivial amount of code behind them, to make it work. However, when the `--raw` flag is used, the implementation boils down to converting the only argument into an API endpoint URL and invoking the raw REST API client.

One of the easiest ways to see the guts of the Kubernetes API objects is to use the `kubectl get` command with the `--raw flag. This lets you to explore your cluster's API landscape without the need of additional tools. For instance, you can issue the following command to get a list of all API paths on the cluster:

kubectl get --raw / | jq '.'

_Click to run on **node1**_

By using `kubectl`, you can explore all accessible objects in your cluster. For instance, the following returns a NodeList resource which is a collection of all nodes represented in the cluster. The output of the command is piped to the `jq` package for improved readability:

kubectl get --raw /api/v1/nodes | jq '.'

_Click to run on **node1**_

Here you can see a whole lot of information about the node in your cluster. If you scroll to the very top, you will see fields like `kind`, `apiVersion` or `name` in the `metadata` section. But there's also `annotations` for internal IP or things like environment variables.

Of course you can also use `kubectl get --raw` to retrieve API objects directly. The following example lets you inspect the JSON object representing the only node in our cluster.

QUERY SPECIFIC RESOURCE:

To retrieve the name of the node to include in your command, see the output of the previous one. It includes a `name` field.

Solution

kubectl get --raw /api/v1/nodes/<NODENAME> | jq '.'

The neat thing about querying the API like this and piping it to `jq` is, that you are also able to directly filter the output. Let's take the following example, in which you retrieve a list of all `nodes` again but filter them only by fields you want to be displayed:

kubectl get --raw /api/v1/nodes | jq '[.items [] | {nodeName: .metadata.name, CPU: .status.capacity.cpu, Memory: .status.capacity.memory, ContainerRuntime: .status.nodeInfo.containerRuntimeVersion}]'

_Click to run on **node1**_

Here, you just filter by nodename, available CPU and memory as well as the container runtime used on the nodes. This would be displayed for each node in the cluster, if there was more than one.

## Step 7: Accessing the API server directly with kubectl proxy

It is also possible to access the API server directly over HTTP, by using a tool like [cURL](https://curl.se), a web browser or some other REST tool.

WHY?:

If you already have working `kubectl`, why would you want to call the Kubernetes API directly?

There are multiple reasons. For instance, you might be developing a controller and want to play around with the API queries without writing extra code. Or, you may be unhappy with the way kubectl handles stuff under the hood while manipulating resources, which makes you want to have more fine-grained control over the operations on Kubernetes objects. Maybe you want to write some external tool, that doesn't just call `kubectl`.

If you need direct HTTP access to the API server, the easiest and quickest way is to use command `kubectl proxy`. This command sets up a proxy allowing access to the running API server. For instance, the following command starts the proxy to forward all requests to localhost port 8080 to the API server:

kubectl proxy --port=8080 &

_Click to run on **node1**_

Not for production use:

While it is possible to access the API server by using the `kubectl proxy` command, this is not the way that is meant for use in production.

The proxy is designed for use in a trusted environment. A solution that can also be used in production environments will be shown on the next page.

For testing purposes or to quickly verify something, the use of `kubectl proxy` to access the API might be okay.

Once the proxy is running, it is possible to access the API server via HTTP using your favorite tool. The following uses cURL, to return all available API paths on the server (the same as `kubectl get --raw /` on the previous page):

curl http://localhost:8080/ | jq '.'

_Click to run on **node1**_

Again, `jq` is used to make the output easier readable. And as you did before, you can access resource lists or access API objects directly as. For example, to retrieve a list of all nodes in the cluster using cURL:

Solution

curl http://localhost:8080/api/v1/nodes | jq '.'

_Click to run on **node1**_

While cURL is being used in these examples, you can of course use any programming language (which often even provide dedicated packages for acessing the Kubernetes API) or other REST tools. You can also submit API objects directly via HTTP requests.

API Objects:

Kubernetes API objects are the fundamental building blocks of a Kubernetes cluster. They are used to define the desired state of the system, such as the deployment of a particular application or the configuration of a network service. Each Kubernetes API object has an `apiVersion`, which identifies the version of the Kubernetes API that the object is defined against. The `apiVersion` helps the Kubernetes API server understand the structure and format of the object, as well as how to handle it.

Another important aspect of an API object is its `kind`. The `kind` determines the type of object being created or modified, such as a `Pod`, `Service` or `Deployment`. Each `kind` has its own set of attributes, which define the desired state of the object. For example, a Pod object might include attributes such as the image to use and the ports to expose, while a Deployment object might also include the number of replicas.

In addition to kind and apiVersion, Kubernetes API objects have associated verbs that define the operations that can be performed on them. These verbs can be used to modify the state of an object, such as creating a new Pod or scaling up an existing Deployment. The most common verbs are:

- `create`: Creates a new Kubernetes API object based on the specified YAML or JSON file.
- `get`: Retrieves the current state of a Kubernetes API object or a list of objects.
- `update`: Updates an existing Kubernetes API object with the specified changes.
- `delete`: Deletes a Kubernetes API object from the cluster.
- `watch`: Monitors a Kubernetes API object for changes and streams updates as they occur.
- `list`: Retrieves a list of Kubernetes API objects that match the specified criteria.

The `kubectl api-resources` command can be used to list all of the API objects that are available in a cluster, along with their `apiVersion`, `kind` and supported verbs. This command is a useful starting point for understanding the Kubernetes API and the objects that can be used to manage a cluster.

Just go ahead and test it right here. Take a look at the output and take note of all the different resources available:

kubectl api-resources

_Click to run on **node1**_

For instance, the following submits a JSON object to create a new Kubernetes namespace called `test-namespace`:

curl http://localhost:8080/api/v1/namespaces/ -k -H "Content-Type: application/json" -XPOST -d '
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "name": "test-namespace"
    }
}'

_Click to run on **node1**_

If you take a closer look at the respone to the command, you can see that an object of kind `namespace` and with the `name` field set to `test-namespace` was created. In the `managedFields` section of the output, we can also see the exact date and time when the command was issued as well as the client (`curl`). By retrieving the list of namespaces now, you can see that the namespace `test-namespace` was indeed properly created.

Idempotency:

All objects you can create via the API have a unique object name to allow idempotent creation and retrieval, except that virtual resource types may not have unique names if they are not retrievable, or do not rely on idempotency. Within a namespace, only one object of a given kind can have a given name at a time. However, if you delete the object, you can make a new object with the same name. Some objects are not namespaced (for example: Nodes), and so their names must be unique across the whole cluster.

In Kubernetes, idempotency is a concept that ensures that a given operation can be performed multiple times without changing the result after the first execution. This means that if a specific action is performed multiple times, it should have the same result as the first time it was executed. This concept is important in Kubernetes because it ensures that resources are deployed and managed consistently, reliably, and without unexpected behavior.

In Kubernetes, this concept of idempotency is applied to all resource management operations, such as creating, updating, or deleting a resource. For instance, if a user requests to create a resource that already exists, the system will recognize the request as a duplicate and will not perform any action. Similarly, if a user requests to delete a resource that is already deleted, the system will recognize the request as redundant and will not perform any action.

You can easily test what was just mentioned about idempotency, by just running the command to create the above `namespace` resource again:

curl http://localhost:8080/api/v1/namespaces/ -k -H "Content-Type: application/json" -XPOST -d '
{
    "apiVersion": "v1",
    "kind": "Namespace",
    "metadata": {
        "name": "test-namespace"
    }
}'

_Click to run on **node1**_

As you can see in the output messages, a resource of that type and name already exists in this scope (in that partiular case cluster wide) and thus will not be created again.

## Step 8: Accessing the API server directly with bearer tokens

If using `kubectl proxy` is not an option, you can access the API server directly without proxied traffic. This implies, however, that you must provide the server with proper authorization credentials. To do this, you first setup a `serviceaccount` and `clusterrolebinding` and then scrape the API server address and the service account authorization token from Kubernetes using kubectl:

Create service account:

kubectl create serviceaccount cluster-admin

_Click to run on **node1**_

Create cluster role binding for the servuce account

kubectl create clusterrolebinding cluster-admin-manual --clusterrole=cluster-admin --serviceaccount=default:cluster-admin

_Click to run on **node1**_

Check if account was create successfully:

kubectl get serviceaccount cluster-admin -o yaml

_Click to run on **node1**_

Create a secret for the serviceaccount (this doesn't happen automatically anymore since Kubernetes 1.24):

secret.yml 

`apiVersion: v1 kind: Secret type: kubernetes.io/service-account-token metadata:   name: cluster-admin   annotations:     kubernetes.io/service-account.name: "cluster-admin"`

_Click to create secret.yml on **node1**_

Apply the secret:

Go ahead and apply the secret in the `secret.yml` after creating it:

kubectl apply -f secret.yml

_Click to run on **node1**_

Verify that the secret is available:

kubectl get secrets

_Click to run on **node1**_

Now you can scrape the API server address and the service account authorization token and save them to environment varibales:

APISERVER=$(kubectl config view -o jsonpath='{.clusters[*].cluster.server}')

TOKEN=$(kubectl get secrets -o jsonpath='{.items[?(@.type=="kubernetes.io/service-account-token")].data.token}' | base64 --decode)

_Click to run on **node1**_

Once that is done, check the contents of both variables, to see that `APISERVER` indeed holds the address of the API server and `TOKEN` contains the Bearer Token to access the cluster:

echo $APISERVER
echo $TOKEN

_Click to run on **node1**_

Finally, we can use the `APISERVER` and `TOKEN` variables as server address and bearer token as part of the HTTP request header:

curl $APISERVER/version --header "Authorization: Bearer $TOKEN" --insecure | jq '.'

_Click to run on **node1**_

Again, here you can use a client of your choice (still cURL in this lab) to retrieve or send any requests to the Kubernetes API:

curl $APISERVER/api/v1/namespaces/ --header "Authorization: Bearer $TOKEN" --insecure | jq '.'

_Click to run on **node1**_

As you have seen on the previous page, it is possible to create API objects via HTTP requests. However, it is also possible to delete them via HTTP requests. You can now delete the namespace that you created before like this:

curl -X DELETE -H "Content-Type: application/json" $APISERVER/api/v1/namespaces/test-namespace --header "Authorization: Bearer $TOKEN" --insecure | jq '.'

_Click to run on **node1**_

Check namespace deletion:

Now check whether the namespace was deleted properly:

Solution

kubectl get ns

As you have read in the introduction parts of this lab, it is also possible, to send `YAML` instead of `JSON` objects. To do this, a `YAML` payload with the `Content-Type` header set to `application/yaml` needs to be sent.

Create namespace via curl & yaml:

Now create (HTTP `POST` request) the namespace `test-namespace` that you have just deleted again, but this time by sending a yaml payload:

To do this, you need to modify the call that you have used on the previous page a bit. First you need to change `-H "Content-Type: application/json"` to -H `"Content-Type: application/yaml"`, also instead of the `JSON` object you need to include the yaml object to create a namespace.

curl -H "Content-Type: application/yaml" $APISERVER/api/v1/namespaces/ \
--header "Authorization: Bearer $TOKEN" \
--insecure \
-d 'apiVersion: v1
kind: Namespace
metadata:
  name: test-namespace
'

_Click to run on **node1**_

Copy & paste the above to your terminal and execute it.

Again, verify that the namespace has been created properly.

## Step 9: Conclusion

Congratulations, you have now completed the Kubernetes API lab.

A Kubernetes cluster is a collection of API objects that are managed by loosely-coupled components. Creating any third party tools on the Kubernetes platform requires you to understand how and know where to get access to these API objects. This lab provides a walkthrough of simple methods and tools you can use to explore Kubernetes API objects without using any programming languages.

### References

- [Kubernetes API documentation](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)
    
- [Kubernetes API reference](https://kubernetes.io/docs/reference/kubernetes-api/)
    
- [Using Kubernetes API](https://kubernetes.io/docs/reference/using-api/)