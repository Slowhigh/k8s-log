### Install general dependencies
You need to install packages on your system for the commands we will use later.
```bash
$ apt-get update

$ apt-get install -y apt-transport-https ca-certificates curl
```
### Install containerd
Although we have a few container runtimes to choose from, we’re going with containerd. Before we install containerd, we’ll create its configuration file.
```bash
$ curl -fsSLo containerd-config.toml https://gist.githubusercontent.com/oradwell/31ef858de3ca43addef68ff971f459c2/raw/5099df007eb717a11825c3890a0517892fa12dbf/containerd-config.toml

$ mkdir /etc/containerd

$ mv containerd-config.toml /etc/containerd/config.toml
```

Without delay, you can install containerd from their official GitHub repository as recommended using the following commands:

```bash
$ curl -fsSLo containerd-1.6.14-linux-amd64.tar.gz https://github.com/containerd/containerd/releases/download/v1.6.14/containerd-1.6.14-linux-amd64.tar.gz

# Extract the binaries
$ tar Cxzvf /usr/local containerd-1.6.14-linux-amd64.tar.gz

# Install containerd as a service
$ curl -fsSLo /etc/systemd/system/containerd.service https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

$ systemctl daemon-reload

$ systemctl enable --now containerd
```

### Install runc
Installing runc from their official GitHub repository is the recommended way.
```bash
$ curl -fsSLo runc.amd64 https://github.com/opencontainers/runc/releases/download/v1.1.3/runc.amd64

$ install -m 755 runc.amd64 /usr/local/sbin/runc
```

### Install CNI network plugins
Install Container Network Interface network plugins from their official GitHub repository.
```bash
$ curl -fsSLo cni-plugins-linux-amd64-v1.1.1.tgz https://github.com/containernetworking/plugins/releases/download/v1.1.1/cni-plugins-linux-amd64-v1.1.1.tgz

$ mkdir -p /opt/cni/bin

$ tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.1.1.tgz
```

### Forward IPv4 and let iptables see bridged network traffic
You need to enable overlay and br_netfilter kernel modules. Additionally, you need to allow iptables see bridged network traffic.
```bash
$ cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ modprobe -a overlay br_netfilter

# sysctl params required by setup, params persist across reboots
$ cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
$ sysctl --system
```

### Install kubeadm, kubelet & kubectl
You need to ensure the versions of kubeadm, kubelet and kubectl are compatible.
```bash
# Add Kubernetes GPG key
$ curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg

# Add Kubernetes apt repository
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | tee /etc/apt/sources.list.d/kubernetes.list

# Fetch package list
$ apt-get update

$ apt-get install -y kubelet kubeadm kubectl

# Prevent them from being updated automatically
$ apt-mark hold kubelet kubeadm kubectl
```

### Ensure swap is disabled
You have to disable the swap feature because Kubernetes does not support it. See the GitHub issue regarding swap on Kubernetes for details.
```bash
# See if swap is enabled
$ swapon --show

# Turn off swap
$ swapoff -a

# Disable swap completely
$ sed -i -e '/swap/d' /etc/fstab
```

### Create the cluster using kubeadm
It’s only a single command to initialise the cluster, but it won’t be very functional in single-node environments until we make some changes. Note that we’re providing “–pod-network-cidr” parameter as required by our CNI plugin (Flannel).
```bash
$ kubeadm init --pod-network-cidr=10.244.0.0/16
```

### Configure kubectl
To access the cluster, we have to configure kubectl.
```bash
$ mkdir -p $HOME/.kube

$ cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

$ chown $(id -u):$(id -g) $HOME/.kube/config
```

### Untaint node
We must untaint the node to allow pods to be deployed to our single-node cluster. Otherwise, your pods will be stuck in a pending state.
```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-             ==> ‼ If there is an error, ignore it

$ kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### Install a CNI plugin
For networking to function, you must install a Container Network Interface (CNI) plugin. With this in mind, we’re installing flannel.
```bash
$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### Install a Metrics Server
```bash
$ kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# fix metrics-server 
# Add "--kubelet-insecure-tls" to "Deployment.spec.template.spec.containers.args"
$ kubectl edit deploy -n kube-system metrics-server
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: apps/v1
kind: Deployment

  ...

    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls                                         ==> ADD
        image: registry.k8s.io/metrics-server/metrics-server:v0.6.3
        imagePullPolicy: IfNotPresent

  ...

  unavailableReplicas: 1
  updatedReplicas: 1

@@@
⌨ Esc => :wq + Enter
```


https://sangvhh.net/exposing-kubernetes-services-with-metallb-and-nginx-ingress-controller/
### MetalLB-LoadBalancer + Nginx-Ingress
```bash
# First, confirm if your Kubernetes cluster is responsive and you're able to use kubectl cluster management command-line tool.
$ kubectl cluster-info
Kubernetes control plane is running at https://10.0.0.8:6443
CoreDNS is running at https://10.0.0.8:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

# Next, download the MetalLB repository using the below command on the control-plane node.
$ git clone https://github.com/metallb/metallb.git

# Jump through many folders to manifests location.
$ cd metallb/config/manifests

# To install MetalLB in your Kubernetes cluster, the following manifest must be deployed on the master node.
$ kubectl apply -f metallb-native.yaml
namespace/metallb-system created
customresourcedefinition.apiextensions.k8s.io/addresspools.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io configured
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io configured
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller configured
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker unchanged
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller unchanged
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker unchanged
secret/webhook-server-cert created
service/webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration configured

# We inspect the components deployed under the metallb-system namespace if the installation of MetalLB was successful.
$ kubectl get all -n metallb-system
NAME                              READY   STATUS    RESTARTS   AGE
pod/controller-548b968c7f-spncz   1/1     Running   0          2m26s
pod/speaker-64b2w                 1/1     Running   0          2m26s

NAME                      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/webhook-service   ClusterIP   10.108.140.145   <none>        443/TCP   2m26s

NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   2m26s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/controller   1/1     1            1           2m26s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/controller-548b968c7f   1         1         1       2m26s

# The installation manifest does not include a configuration file. All MetalLB components are started, but will remain in idle state until you define and deploy necessary configurations.

# All the IPs allocated via IPAddressPool contribute to the pool of IPs that MetalLB uses to assign IPs to services.
$ nano ipaddress-pools.yaml
apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
  name: first-pool
  namespace: metallb-system
spec:
  addresses:
  - 10.0.0.8-10.0.0.8
---
apiVersion: metallb.io/v1beta1
kind: L2Advertisement
metadata:
  name: l2-advert
  namespace: metallb-system

# To apply this configuration, run the following command.
$ kubectl apply -f ipaddress-pools.yaml
ipaddresspool.metallb.io/first-pool created
l2advertisement.metallb.io/l2-advert created

# Once configuration is deployed, list the created IPAddressPool and L2Advertisement.
$ kubectl get ipaddresspools.metallb.io -n metallb-system
NAME         AUTO ASSIGN   AVOID BUGGY IPS   ADDRESSES
first-pool   true          false             ["10.0.0.8-10.0.0.8"]

$ kubectl get l2advertisements.metallb.io -n metallb-system
NAME        IPADDRESSPOOLS   IPADDRESSPOOL SELECTORS   INTERFACES
l2-advert   

# We execute below kubectl command on the control-plane node to install Nginx ingress.
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/baremetal/deploy.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx unchanged
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx unchanged
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission unchanged
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx unchanged
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission configured

# ingress-nginx-controller의 서비스 구성에서 spec.type 필드를 "NodePort"에서 "LoadBalancer" 변경
$ kubectl edit svc ingress-nginx-controller -n ingress-nginx
service/ingress-nginx-controller edited

# Using this command, we check if the EXTERNAL-IP field is pending or not. Normally, the first address of the MetalLB IP pool should be allocated to the ingress service.
kubectl -n ingress-nginx get svc
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.103.25.76     10.0.0.8      80:30568/TCP,443:30856/TCP   100s
ingress-nginx-controller-admission   ClusterIP      10.109.146.111   <none>        443/TCP                      100s
```