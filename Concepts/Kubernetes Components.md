# Kubernetes Components

![components-of-kubernetes](image/components-of-kubernetes.svg)

## Control Plane Components

### kube-apiserver
The `API server` is a component of the Kubernetes control plane that exposes the `Kubernetes API`. The `API server` is the front end for the Kubernetes control plane.

### etcd
Consistent and highly-available key value store used as Kubernetes' backing store for all cluster data.

### kube-scheduler
Component that watches for newly created `Pods` with no assigned `node`, and selects a `node` for them to run on.

### kube-controller-manager
Component that runs controller processes.

- `Node Controller`: Responsible for noticing and responding when nodes go down.
- `Job controller`: Watches for Job objects that represent one-off tasks, then creates Pods to run those tasks to completion.
- `EndpointSlice controller`: Populates EndpointSlice objects (to provide a link between Services and Pods).
- `ServiceAccount controller`: Create default ServiceAccounts for new namespaces.

## Node Components

### kubelet
An agent that runs on each `node` in the `cluster`. It makes sure that containers are running in a `Pod`. (The kubelet doesn't manage containers which were not created by Kubernetes.)

### kube-proxy
`kube-proxy` is a network proxy that runs on each `node` in your `cluster`, implementing part of the Kubernetes `Service` concept.

### Container runtime
The `container runtime` is the software that is responsible for running containers.
