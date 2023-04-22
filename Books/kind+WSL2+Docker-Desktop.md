# kind + WSL2 + Docker-Desktop
***reference: https://kubernetes.io/blog/2020/05/21/wsl-docker-kubernetes-on-the-windows-desktop/ ***

#### Prerequisites
- Docker used the WSL2 based engin.

#### [Optional] Update the sudoers
```bash
# Edit the sudoers with the visudo command
$ sudo visudo

# Change the %sudo group to be password-less
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL

# Press CTRL+X to exit
# Press Y to save
# Press Enter to confirm
```

#### Update Ubuntu
```bash
# Update the repositories and list of the packages available
$ sudo apt update

# Update the system based on the packages installed > the "-y" will approve the change automatically
$ sudo apt upgrade -y
```

#### kind: Kubernetes made easy in a container 
[latest version of kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-from-release-binaries)
```bash
# Download the latest version of kind
$ curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64

# Make the binary executable
$ chmod +x ./kind

# Move the binary to your executable path
$ sudo mv ./kind /usr/local/bin/kind
```

#### kind: the first cluster
```bash
# Check if the KUBECONFIG is not set
$ echo $KUBECONFIG

# Check if the .kube directory is created > if not, no need to create it
$ ls $HOME/.kube

# Create the cluster and give it a name (optional)
$ kind create cluster --name wslkind

# Check if the .kube has been created and populated with files
$ ls $HOME/.kube

# Check the cluster information
$ kubectl cluster-info

# Check how many nodes it created
$ kubectl get nodes -A

# Check the services for the whole cluster
$ kubectl get all --all-namespaces

# Delete the existing cluster
$ kind delete cluster --name wslkind
```

#### Multi-Node Cluster
```bash
# Create a config file for a 3 nodes cluster
$ cat << EOF > kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF

# Create a new cluster with the config file
$ kind create cluster --name wslkindmultinodes --config ./kind-3nodes.yaml

# Check how many nodes it created
$ kubectl get nodes

# Check the services for the whole cluster
$ kubectl get all --all-namespaces
```

#### Kubernetes Dashboard (localhost, not remote)
```bash
# Install the Dashboard application into our cluster
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

# Check the resources it created based on the new namespace created
$ kubectl get all -n kubernetes-dashboard

# Start a kubectl proxy
$ kubectl proxy
# Enter the URL on your browser: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

# Create a new ServiceAccount
$ kubectl apply -f - <<EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Create a ClusterRoleBinding for the ServiceAccount
$ kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF

# Create the Token for logging in.
$ kubectl -n kubernetes-dashboard create token admin-user
# Copy the token and paste it into the `Enter token` field on the login screen.

$ kubectl -n kubernetes-dashboard delete serviceaccount admin-user
$ kubectl -n kubernetes-dashboard delete clusterrolebinding admin-user
```

