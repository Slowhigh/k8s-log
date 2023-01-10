# **Kubernetes Object Management**

|Management technique               |Operates on            |Recommended environment    |Supported writers  |Learning curve |
|-----------------------------------|-----------------------|---------------------------|-------------------|---------------|
|Imperative commands                |Live objects           |Development projects       |1+                 |Lowest         |
|Imperative object configuration    |Individual files       |Production projects        |1                  |Moderate       |
|Declarative object configuration   |Directories of files   |Production projects        |1+                 |Highest        |

<br>

## Imperative commands
Run an instance of the nginx container by creating a Deployment object:
```bash
$ kubectl create deployment nginx --image nginx
```

<br>

## Imperative object configuration
In imperative object configuration, the kubectl command specifies the operation (create, replace, etc.), optional flags and at least one file name. The file specified must contain a full definition of the object in YAML or JSON format.
```bash
$ kubectl create -f config/nginx.yaml

$ kubectl create -f config/redis.yaml

$ kubectl get deployments

$ kubectl describe deployments

$ kubectl get pods

# Update the `replicas` from 2 to 4 of configs/nginx.yaml 

$ kubectl replace -f configs/nginx.yaml

$ kubectl get pods

$ kubectl delete -f configs/nginx.yaml -f configs/redis.yaml

$ kubectl get deployments

$ kubectl get pods
```

## Declartive object configuration
When using declarative object configuration, a user operates on object configuration files stored locally, however the user does not define the operations to be taken on the files. Create, update, and delete operations are automatically detected per-object by `kubectl`. This enables working on directories, where different operations might be needed for different objects.

```bash
# Process all object configuration files in the configs directory, 
# and create or patch the live objects. 
# You can first diff to see what changes are going to be made, 
# and then apply
$ kubectl diff -f configs/
$ kubectl apply -f configs/

# Recursively process directories
$ kubectl diff -R -f configs/
$ kubectl apply -R -f configs/
```