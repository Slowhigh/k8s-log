# **Namespaces**
In Kubernetes, `namespaces` provides a mechanism for isolating groups of resources within a single cluster. Names of resources need to be unique within a namespace, but not across namespaces. Namespace-based scoping is applicable only for namespaced objects (e.g. Deployments, Services, etc) and not for cluster-wide objects (e.g. StorageClass, Nodes, PersistentVolumes, etc).


```bash
# Viewing namespaces
$ kubectl get namespace
$ kubectl get ns

# Setting the namespace for a request
$ kubectl run nginx --image=nginx --namespace=<insert-namespace-name-here>
$ kubectl get pods --namespace=<insert-namespace-name-here>

# Setting the namespace preference
$ kubectl config set-context --current --namespace=<insert-namespace-name-here>

$ kubectl config view --minify
$ kubectl config view --minify | grep namespace:

$ kubectl config unset contexts.<current-cluster-name-here>.namespace

$ kubectl config view --minify


# In a namespace
$ kubectl api-resources --namespaced=true

# Not in a namespace
$ kubectl api-resources --namespaced=false

```