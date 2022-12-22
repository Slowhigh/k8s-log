# **Deploy and Access the Kubernetes Dashboard**

<img src="image/ui-dashboard.png" alt="`Cluster Diagram" width="100%"/>

```bash
# Deploying the Dashboard UI
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.6.1/aio/deploy/recommended.yaml

$ kubectl get serviceaccount -A
# NAMESPACE              NAME                                 SECRETS   AGE
# default                default                              0         47h
# kube-node-lease        default                              0         47h
# kube-node-lease        default                              0         47h
# kube-public            default                              0         47h
#
# ...
# 
# kubernetes-dashboard   default                              0         47h
# kubernetes-dashboard   kubernetes-dashboard                 0         47h           <-- [Name Space] [Service Account Name]

kubectl -n kubernetes-dashboard describe serviceaccount kubernetes-dashboard
kubectl -n default create token default

# Dashboard URL: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
$ kubectl proxy
```