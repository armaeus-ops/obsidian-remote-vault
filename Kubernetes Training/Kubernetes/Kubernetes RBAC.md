Role-Based access control w/ Roles, RoleBindings and ServiceAccounts

## Step 1: Introduction to RBAC in Kubernetes

RBAC (Role-Based Access Control) is a method for controlling access to resources in Kubernetes.

With RBAC, you can assign roles to users or groups, and those roles determine what actions the users or groups can perform on the resources in the cluster. This allows for a more fine-grained control of access to resources, as opposed to traditional all-or-nothing access control.

RBAC is implemented as a Kubernetes API resource and can be managed using kubectl or other Kubernetes management tools.

## Step 2: Preparation

Please run the following command to install the k3s cluster:

curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=v1.28.5+k3s1 sh -

_Click to run on **node1**_

Verify the installation by running:

kubectl get nodes

_Click to run on **node1**_

Create a namespace named `dev`:

kubectl create ns dev

_Click to run on **node1**_

## Step 3: Kubernetes Users

In Kubernetes, users can be created using different authentication methods, depending on the setup of the cluster. Some of the most common methods for creating users in Kubernetes are:  

- `Basic Authentication`: Users are created with a username and password, and those credentials are used for authentication.
- `Token-based Authentication`: Users are created with a token, which is a unique string that is generated by the authentication server.
- `Client Certificate Authentication`: Users are created with a client certificate, which is a digital certificate that is used for authentication.  
    Once a user is created, it can be granted permissions to access resources in the cluster using RBAC. You can create roles and assign them to users or groups, this way you can grant access to specific resources or specific actions on those resources.

It's important to note that the process of creating a user can vary depending on the authentication method and the setup of the cluster. Some clusters use an external authentication provider such as an LDAP or an Active Directory, this means that the users are managed outside of Kubernetes and the process of creating a user is different.

In this Lab we will not create a user directly. We are relying on kubectl's impersonation feature, that allows us to query against the kubernetes API and get a result as if we are a given user

## Step 4: ServiceAccount

A ServiceAccount is a special type of user account that is associated with a Pod. It is used to give pods the ability to access the Kubernetes API. Each pod is associated with a ServiceAccount, and the ServiceAccount determines the permissions that the pod has to access the Kubernetes API.

When a pod is created, it is associated with a ServiceAccount. By default, the pod is associated with the "default" ServiceAccount in the same namespace. This default ServiceAccount has a limited set of permissions, and it can only access the Kubernetes API to retrieve information about the pod and its associated resources.

The default ServiceAccount is created by default for every namespace.

kubectl get serviceaccount default -n dev

_Click to run on **node1**_

You can also create custom ServiceAccounts with specific permissions, and then associate pods with those ServiceAccounts. This allows you to control the access that a pod has to the Kubernetes API. To Change the ServiceAccount used by a Pod you would have to set it on the `spec.serviceAccountName` field of the Pod or Deployment.

You can use RBAC to assign roles to a ServiceAccount, and then use that ServiceAccount in a pod, this way you can grant a pod the access to specific resources, or even specific actions on those resources.

It's important to note that the ServiceAccount is not the same as a user account, it's a way to grant permissions to a pod, and it's not meant to be used for human users.

## Step 5: Roles and ClusterRoles

In Kubernetes RBAC, roles are used to grant access to resources in the cluster. A role is a set of rules that defines the actions that can be performed on a specific set of resources. There are two types of roles: ClusterRoles and Roles.  

- A `ClusterRole` is a set of rules that can be applied to resources in the entire cluster, regardless of the namespace in which they reside.
- A `Role`, on the other hand, is a set of rules that can only be applied to resources within a specific namespace.

  
Each role is composed of one or more rules, and each rule defines a set of resources and the actions that can be performed on those resources. For example, a rule may allow a user or group to read and update pods in a specific namespace.

You can also use Kubernetes built-in roles to grant access to users. For example, the "cluster-admin" role grants full access to all resources in the cluster, and the "admin" role grants full access to resources within a specific namespace.

Note that some cluster-wide resources like `node` require `ClusterRoles` to be used.

## Step 6: Rule

A rule defines what a user or group is authorized to do on a set of resources, it also defines the scope of the resources that the rule applies to. It contains a set of conditions that determines what actions can be performed on a specific set of resources. Each rule is composed of three elements: resources, verbs, and apiGroups.

### Resources

The resources that the rule applies to. For example, "pods" or "deployments" or any custom resource.

### Verbs

In Kubernetes RBAC, the verbs determine the actions that are allowed on a set of resources. Here is a list of the verbs that can be used in a rule:

- `get`: Allow retrieval of a specific resource
- `list`: Allow retrieval of a list of resources
- `watch`: Allow watching a resource or a list of resources
- `create`: Allow creation of a new resource
- `update`: Allow updating of an existing resource
- `patch`: Allow partially updating an existing resource
- `delete`: Allow deletion of an existing resource
- `deletecollection`: Allow deletion of a collection of resources

It's important to note that not all resources support all verbs.

### ApiGroups

The ApiGroup that the resources belong to. For example, `""` (Core ApiGroup), `extensions`, `apps`, `batch`, etc.

It's also important to note that some apiGroups may have different resources, and those resources may have different verbs available, you should check the API documentation to verify which verbs are available for each resources.

### Access all resources/verbs/apiGroups

If a user should access all resources, verbs or ApiGroups the star operator `*` can be used.

## Step 7: Example Role

The following example `Role` has a single rule that grants access to the `pods` resource in the apiGroups ["", "extensions", "apps"]. The verbs that are allowed are "get", "list", "watch", and "update", meaning that users or groups that are bound to this role can read, list and update the pods in the dev namespace.

yaml 

`apiVersion: rbac.authorization.k8s.io/v1 kind: Role metadata:   name: pod-reader-updater   namespace: dev rules: - apiGroups: [""] # "" indicates the core ApiGroup   resources: ["pods"]   verbs: ["get", "list", "watch", "update"]`

Create the Role with `kubectl create role`:

kubectl create role pod-reader-updater --verb=get --verb=list --verb=watch --verb=update --resource=pods -n dev

_Click to run on **node1**_

This is just one example, you could create a role to update and create deployments, or a role to read and edit configmaps, or even a role to manage access to secrets and more. To give access to all resources you would change `["pods"]` to `'*'` Keep in mind that you can have different roles for different namespaces and different resources, you can even have different roles for different resources in the same namespace.

To create a global Role that targets all namespaces you would need to create a `ClusterRole`

### View Role

`kubectl describe` will output the granted rights in a nice manner

kubectl describe role pod-reader-updater -n dev

_Click to run on **node1**_

## Step 8: RoleBindings and ClusterRoleBinding

In addition to roles, Kubernetes RBAC also uses bindings to grant roles to users or groups. A `RoleBinding` or `ClusterRoleBinding` is used to grant a role to a user, group or ServiceAccount. This is done by specifying the role and the user or group that should have that role.

### subjects

All RoleBindings need to refer to a list of subjects that are bound to the refered roleRef. You can allow single users, groups or serviceaccounts.

Example User

yaml 

`subjects: - kind: User   name: "alice@example.com"   apiGroup: rbac.authorization.k8s.io`

Example Group

yaml 

`subjects: - kind: Group   name: "frontend-admins"   apiGroup: rbac.authorization.k8s.io`

Example ServiceAccount

yaml 

`subjects: - kind: ServiceAccount   name: default   namespace: kube-system`

You could allow all ServiceAccounts inside a namespace

- `system:serviceaccount`: (singular) is the prefix for service account usernames.
- `system:serviceaccounts`: (plural) is the prefix for service account groups.

Two default groups exist that can be used

- `system:authenticated`: groups all authenticated users
- `system:unauthenticated`: groups all unauthenticated users

### Example RoleBinding

This RoleBinding is named "pod-reader-updater-binding" and it's in the "dev" namespace. It binds the "pod-reader-updater" role created in the last step to the user "jane", and this user will have the ability to get, list, watch and update pods in the "dev" namespace. Apply the Role to the cluster.

yaml 

`apiVersion: rbac.authorization.k8s.io/v1 kind: RoleBinding metadata:   name: pod-reader-updater-binding   namespace: dev subjects: # More than one subject is allowed - kind: User # User, Group or ServiceAccount   name: jane   apiGroup: rbac.authorization.k8s.io roleRef:   kind: Role # Role or ClusterRole   name: pod-reader-updater # Matches the name of the (Cluster)Role   apiGroup: rbac.authorization.k8s.io`

Create rhe RoleBindingt with `kubectl create rolebinding`

kubectl create rolebinding pod-reader-updater-binding --role=pod-reader-updater --user=jane --namespace=dev

_Click to run on **node1**_

## Step 9: Verify that RBAC works

We created a Role that allows the verbs get, list, watch and update of pods in the `dev` namespace. With the RoleBinding we bound that Role to a user named `jane`.

### Setup Pod

Create a Pod in the dev namespace. It does not matter what specific container it runs - only that it runs and is in the dev namespace.

kubectl run nginx --image nginx:latest -n dev

_Click to run on **node1**_

Verify that the pod is running

kubectl get pods -n dev

_Click to run on **node1**_

### Assumptions

What is the expected output for `jane` when running the `kubectl get pods` command inside the dev namespace? What is the expected output for other namespaces?

Solution

Jane would see the Pod we have created inside the dev namespace. Jane can not list other pods inside other namespace (default, kube-system etc.) due to insufficient rights.

What is the expected output for a user named `oliver` when executing the same command?

Solution

Oliver can not see any Pods due to insufficient rights.

### Verify our assumptions

We can impersonate users using the `--as` flag when executing kubectl. Groups can be impersonated using `--group=[]`. Multiple groups can be specified.

To show which pods jane is able to see inside the dev namespace we can execute

kubectl get pods -n dev --as='jane'

_Click to run on **node1**_

We can see the pod we have create above. Now the same for any other namespace

kubectl get pods -n default --as='jane'

_Click to run on **node1**_

Jane is not able to list pods from any other namespace and we get the following error message.

Error from server (Forbidden): pods is forbidden: User "jane" cannot list resource "pods" in ApiGroup "" in the namespace "default"

How about listing pods with the `-A` or `--all` flag?

kubectl get pods -A --as='jane'

_Click to run on **node1**_

Interesting. We do not get the pods from the dev namespace but the error message that we need cluster scope access.

Now the same for oliver. Can oliver list pods in the dev namespace?

kubectl get pods -n dev --as='oliver'

_Click to run on **node1**_

We get the same error message we got when jane was accessing resources from any other namespace than dev. Oliver does not have any RoleBinding that grants him access to the pod resource.

### Check if a right is granted

kubectl provides the `kubectl auth can-i` command to check granted rights for the current user.

Can the user jane create pods in the dev namespace?

kubectl auth can-i create pods --namespace dev --as jane

_Click to run on **node1**_

The answer is simply answered with yes or no. In this case the answer is 'no', according to our Role jane can only get, list, update and watch.

## Step 10: RBAC for specific resources.

In the last steps we configured RBAC for whole resources. Kubernetes RBAC even allows to grant access to specific resources.

kubectl create role pod-reader-specific --verb=get --verb=list --resource=pods --resource-name=nginx --resource-name=anotherpod -n dev

_Click to run on **node1**_

kubectl get role pod-reader-specific -n dev -o yaml

_Click to run on **node1**_

kubectl create rolebinding pod-reader-specific-binding --role=pod-reader-specific --user=oliver -n dev

_Click to run on **node1**_

Now oliver can access the pods "nginx" and "anotherpod" but not any other pods. he is also not able to list other pods.

kubectl get pods nginx -n dev --as='oliver'

_Click to run on **node1**_

kubectl get pods -n dev --as='oliver'

_Click to run on **node1**_

## Step 11: Further documentation

- Further documentation on RBAC can be found in the [offical kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
- More on [how to create ServiceAccounts for Pods.](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)
- To access Kubernetes resources programatically you should read the [documentation on kubernets libraries](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/#programmatic-access-to-the-api) aswell as the [docs on hot to access the api from a pod](https://kubernetes.io/docs/tasks/run-application/access-api-from-pod/).