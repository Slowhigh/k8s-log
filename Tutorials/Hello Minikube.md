### Create a minikube cluster
[PowerShell A]
```sh
PS C:\> minikube start
```

[PowerShell B]
```sh
PS C:\> minikube dashboard
```

### Create a Deployment
```sh
PS C:\> kubectl create deployment hello-node --image=registry.k8s.io/echoserver:1.4
PS C:\> kubectl get deployments
PS C:\> kubectl get pods
PS C:\> kubectl get events
PS C:\> kubectl config view
```

### Create a Service
```sh
PS C:\> kubectl expose deployment hello-node --type=LoadBalancer --port=8080
PS C:\> kubectl get services
PS C:\> minikube service hello-node
```

### Enable addons
```sh
PS C:\> minikube addons list
PS C:\> minikube addons enable metrics-server
PS C:\> kubectl get pod,svc -n kube-system
PS C:\> minikube addons disable metrics-server
PS C:\> metrics-server was successfully disabled
```

### Clean up
```sh
PS C:\>  kubectl delete service hello-node
PS C:\>  kubectl delete deployment hello-node
PS C:\>  minikube stop
PS C:\>  minikube delete
```