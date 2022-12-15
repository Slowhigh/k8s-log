# **Create a Cluster**

### Kubernetes Cluster
<img src="image/module_01_cluster.svg" alt="`Cluster Diagram" width="100%"/>

- `Kubernetes`는 컴퓨터들을 연결하여 단일 형상으로 동작하도록 컴퓨팅 `Computing Cluster`를 구성하고 높은 가용성을 제공하도록 조율한다. 
- `Kubernetes`는 이러한 애플리케이션 컨테이너를 `Cluster`에 분산시키고 스케줄링하는 일을 더욱 효율적으로 자동화한다.
- `Control Plain`은 `Cluster` 관리를 담당한다. `Control Plain`은 애플리케이션을 스케줄링하거나, 애플리케이션의 항상성을 유지하거나, 애플리케이션을 스케일링하고, 새로운 변경사항을 순서대로 반영(rolling out)하는 일과 같은 `Cluster` 내 모든 활동을 조율한다.
- `Node`는 `Cluster` 내 워커 머신으로 동작하는 VM 또는 물리적인 컴퓨터다. 
- 각 `Node`는 `Node`를 관리하고 `Control Plain`과 통신하는 `Kubelet`이라는 에이전트를 갖는다. 
- `Node`는 컨테이너 운영을 담당하는 `containerd` 또는 `Docker`와 같은 툴도 갖는다. 
- `Node`는 `Control Plain`이 제공하는 `Kubernetes API`를 통해서 `Control Plain`과 통신한다. 

<br>

### Minikube를 사용해서 Cluster 생성하기
```bash
$ minikube version

$ minikube start

$ kubectl version

$ kubectl cluster-info

$ kubectl get nodes
```

<br><br>

# **Deploy an App**

### Kubernetes Deployment
<img src="image/module_02_first_app.svg" alt="Kubernetes에 첫 번째 애플리케이션 배포하기" width="100%"/>

- 일단 `Cluster`를 구동시키면, 그 위에 컨테이너화된 애플리케이션을 배포할 수 있다.
- `Deployment`는 `Kubernetes`가 애플리케이션의 인스턴스를 어떻게 생성하고 업데이트해야 하는지를 지시한다.
- `Deployment Controller`는 머신의 장애나 정비에 대응할 수 있는 자동 복구(self-healing) 메커니즘을 제공한다.

<br>

### kubectl을 사용해서 Deployment 생성하기
```bash
$ kubectl get nodes

# kubectl create deployment [_Deployment Name_] --image=[_____________App Image Location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments

# open a new terminal and run the `kubectl proxy`
$ kubectl proxy

$ curl http://127.0.0.1:8001/version

$ kubectl get pods

# curl http://localhost:8001/api/v1/namespaces/default/pods/[______________Pod Name____________]/
$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubernetes-bootcamp-75c5d958ff-96n6l/

# kubectl delete deployment [_Deployment Name_]
$ kubectl delete deployment kubernetes-bootcamp
```

<br><br>

# **Explore Your App**

### Kubernetes Pod
<img src="image/module_03_pods.svg" alt="Pod 개요" width="100%"/>

`Pod`는 하나 또는 그 이상의 애플리케이션 컨테이너 (`Docker`와 같은)들의 그룹을 나타내는 `Kubernetes`의 추상적 개념으로 일부는 컨테이너에 대한 자원을 공유한다. 그 자원은 다음을 포함한다.
- 볼륨과 같은, 공유 스토리지
- `Cluster` IP 주소와 같은, 네트워킹
- 컨테이너 이미지 버전 또는 사용할 특정 포트와 같이, 각 컨테이너가 동작하는 방식에 대한 정보

<br>

### Kubernetes Node
<img src="image/module_03_nodes.svg" alt="Node 개요" width="100%"/>

`Pod`는 언제나 `Node` 상에서 동작한다. 
`Node`는 `Kubernetes`에서 워커 머신을 말하며 `Cluster`에 따라 가상 또는 물리 머신일 수 있다.

모든 `Node`는 최소한 다음과 같이 동작한다.
- Kubelet은, `Control Plain`과 `Node` 간 통신을 책임지는 프로세스이며, 하나의 머신 상에서 동작하는 `Pod`와 컨테이너를 관리한다.
- `Container Runtime`(`Docker`와 같은)은 레지스트리에서 컨테이너 이미지를 가져와 묶여 있는 것을 풀고 애플리케이션을 동작시키는 책임을 맡는다.

<br>

### Pod와 Node 보기
```bash
# kubectl create deployment [_Deployment Name_] --image=[_____________App Image Location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get pods

$ kubectl describe pods

# open a new terminal and run the `kubectl proxy`
$ kubectl proxy

# curl http://localhost:8001/api/v1/namespaces/default/pods/[______________Pod Name____________]/proxy/
$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubernetes-bootcamp-75c5d958ff-96n6l/proxy/

# kubectl logs [______________Pod Name____________]
$ kubectl logs kubernetes-bootcamp-75c5d958ff-96n6l

# kubectl delete deployment [_Deployment Name_]
$ kubectl delete deployment kubernetes-bootcamp
```

<br><br>

# **Expose Your App Publicly**

### Kubernetes Service들에 대한 개요
- `Kubernetes Service`는 논리적 `Pod` 셋을 정의하고 외부 트래픽 노출, 로드밸런싱 그리고 그 `Pod`들에 대한 서비스 디스커버리를 가능하게 해주는 추상 계층이다.
- `Worker Node`가 죽으면, `Node` 상에서 동작하는 `Pod`들 또한 종료된다.
- `ReplicaSet`은 여러분의 애플리케이션이 지속적으로 동작할 수 있도록 새로운 `Pod`들의 생성을 통해 동적으로 `Cluster`를 미리 지정해 둔 상태로 되돌려 줄 수도 있다.
- `Service`가 대상으로 하는 `Pod` 셋은 보통 `LabelSelector`에 의해 결정된다 
- 비록 각 `Pod`들이 고유의 IP를 갖고 있기는 하지만, 그 IP들은 `Service`의 도움없이 `Cluster` 외부로 노출되어질 수 없다. 
- `Service`들은 ServiceSpec에서 type을 지정함으로써 다양한 방식들로 노출시킬 수 있다
    - *ClusterIP* (기본값) - `Cluster` 내에서 내부 IP 에 대해 `Service`를 노출해준다. 이 방식은 오직 `Cluster` 내에서만 `Service`가 접근될 수 있도록 해준다.
    - *NodePort* - NAT가 이용되는 `Cluster` 내에서 각각 선택된 노드들의 동일한 포트에 `Service`를 노출시켜준다. <NodeIP>:<NodePort>를 이용하여 `Cluster` 외부로부터 `Service`가 접근할 수 있도록 해준다. ClusterIP의 상위 집합이다.
    - *LoadBalancer* - (지원 가능한 경우) 기존 클라우드에서 외부용 로드밸런서를 생성하고 `Service`에 고정된 공인 IP를 할당해준다. NodePort의 상위 집합이다.
    - *ExternalName* - CNAME 레코드 및 값을 반환함으로써 `Service`를 externalName 필드의 내용(예를 들면, foo.bar.example.com)에 매핑한다. 어떠한 종류의 프록시도 설정되지 않는다. 이 방식은 kube-dns v1.7 이상 또는 CoreDNS 버전 0.0.8 이상을 필요로 한다.

<br>

### Services와 Labels
- `Service`는 `Kubernetes`의 객체들에 대해 논리 연산을 허용해주는 기본 그룹핑 단위인, `Label`과 `Selector`를 이용하여 `Pod` 셋과 매치시킨다. 레이블은 오브젝트들에 붙여진 키/밸류 쌍으로 다양한 방식으로 이용 가능하다.

- `Label`은 오브젝트의 생성 시점 또는 이후 시점에 붙여질 수 있다. 언제든지 수정이 가능하다.

<img src="image/module_04_labels.svg" alt="Node 개요" width="100%"/>

<br>

### 앱 노출을 위해 서비스 이용하기
```bash
$ kubectl get pods

$ kubectl get services

# kubectl create deployment [_Deployment Name_] --image=[_____________App Image Location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

# [_E_]:[_I_]는 현재 Cluster가 열어 놓은 Port 번호 중에 하나를 선택
# kubectl create service nodeport [_Deployment Name_] --node-port=[_I_] --tcp=8080:8080
$ kubectl create service nodeport kubernetes-bootcamp --node-port=32443 --tcp=8080:8080

# curl localhost:[_E_]
$ curl localhost:57440

# kubectl describe pods [______________Pod Name____________]
$ kubectl describe pods kubernetes-bootcamp-75c5d958ff-695ph
# -------------------------------------------------------------------------------------------------
# Name:             kubernetes-bootcamp-75c5d958ff-695ph
# Namespace:        default
# Priority:         0
# Service Account:  default
# Node:             minikube/192.168.49.2
# Start Time:       Thu, 15 Dec 2022 13:59:16 +0900
# Labels:           app=kubernetes-bootcamp                                         <--- This Label
#                   pod-template-hash=75c5d958ff
# Annotations:      <none>
# Status:           Running
# IP:               172.17.0.3
# IPs:
#   IP:           172.17.0.3
# Controlled By:  ReplicaSet/kubernetes-bootcamp-75c5d958ff
# Containers:
#   kubernetes-bootcamp:
#     Container ID:   docker://442102be84f6b1d848d488916f...
#     Image:          gcr.io/google-samples/kubernetes-bootcamp:v1
#     Image ID:       docker-pullable://gcr.io/google-samples/...
#     Port:           <none>
#     Host Port:      <none>
#     State:          Running
#       Started:      Thu, 15 Dec 2022 13:59:17 +0900
#     Ready:          True
#     Restart Count:  0
#     Environment:    <none>
#     Mounts:
#       /var/run/secrets/kubernetes.io/serviceaccount from...
# Conditions:
#   Type              Status
#   Initialized       True
#   Ready             True
#   ContainersReady   True
#   PodScheduled      True
# Volumes:
#   kube-api-access-d4kcl:
#     Type:                    Projected (a volume that con...
#     TokenExpirationSeconds:  3607
#     ConfigMapName:           kube-root-ca.crt
#     ConfigMapOptional:       <nil>
#     DownwardAPI:             true
# QoS Class:                   BestEffort
# Node-Selectors:              <none>
# Tolerations:                 node.kubernetes.io/not-ready...
#                              node.kubernetes.io/unreachable...
# Events:                      <none>

# kubectl get pods -l [______This Label_____]
$ kubectl get pods -l app=kubernetes-bootcamp
# -------------------------------------------------------------------------------------------------
# NAME                                   READY   STATUS    RESTARTS   AGE
# kubernetes-bootcamp-75c5d958ff-695ph   1/1     Running   0          126m            <--- Pod Name

# kubectl get services -l [______This Label_____]
$ kubectl get services -l app=kubernetes-bootcamp

# kubectl label pods [______________Pod Name____________] [NewLabel]
$ kubectl label pods kubernetes-bootcamp-75c5d958ff-695ph version=v1

# kubectl describe pods [______________Pod Name____________]
$ kubectl describe pods kubernetes-bootcamp-75c5d958ff-695ph

# kubectl get pods -l [NewLabel]
$ kubectl get pods -l version=v1

# kubectl delete service -l [______This Label_____]
$ kubectl delete service -l app=kubernetes-bootcamp

$ kubectl get services

# curl localhost:[_E_]
$ curl localhost:57440

# kubectl exec -ti [______________Pod Name____________] -- curl localhost:8080
$ kubectl exec -ti kubernetes-bootcamp-75c5d958ff-695ph -- curl localhost:8080

# kubectl delete deployment [_Deployment Name_]
$ kubectl delete deployment kubernetes-bootcamp

# kubectl exec -ti [______________Pod Name____________] -- curl localhost:8080
$ kubectl exec -ti kubernetes-bootcamp-75c5d958ff-695ph -- curl localhost:8080
```

<br><br>

# **Scale Your App**

### 애플리케이션을 스케일하기
- `Deployment`의 복제 수를 변경하면 스케일링이 수행된다
- `kubectl create deployment` 명령에 `--replicas` 파라미터를 사용해서 처음부터 복수의 인스턴스로 구동되는 `Deployment`를 만들 수도 있다

<img src="image/module_05_scaling1.svg" alt="스케일링 개요" width="49%"/>
<img src="image/module_05_scaling2.svg" alt="스케일링 개요" width="49%"/>

- `Deployment`를 스케일 아웃하면 신규 `Pod`가 생성되어 가용 가능한 자원이 있는 `Node`에 스케줄된다.
- `Service`는 노출된 `Deployment`의 모든 `Pod`에 네트워크 트래픽을 분산시켜줄 통합된 로드밸런서를 갖는다.
- `Service`는 엔드포인트를 이용해서 구동중인 `Pod`를 지속적으로 모니터링함으로써 가용한 `Pod`에만 트래픽이 전달되도록 해준다.

<br>

### 복수의 앱 인스턴스를 구동하기
```bash
# rs: ReplicaSet
$ kubectl get rs

# kubectl create deployment [_Deployment Name_] --image=[_____________App Image Location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments

# rs: ReplicaSet
$ kubectl get rs

# [_E_]:[_I_]는 현재 Cluster가 열어 놓은 Port 번호 중에 하나를 선택
# kubectl create service nodeport [_Deployment Name_] --node-port=[_P_] --tcp=8080:8080
$ kubectl create service nodeport kubernetes-bootcamp --node-port=32443 --tcp=8080:8080

# curl localhost:[_E_]
$ curl localhost:57440

# Set a new size for a deployment, replica set, replication controller, or stateful set.
# kubectl scale deployments [_Deployment Name_] --replicas=4
$ kubectl scale deployments kubernetes-bootcamp --replicas=4

$ kubectl get deployments

$ kubectl get pods -o wide

# kubectl describe deployments [_Deployment Name_]
$ kubectl describe deployments kubernetes-bootcamp

$ kubectl get services

# kubectl describe services [__Services Name__]
$ kubectl describe services kubernetes-bootcamp
```