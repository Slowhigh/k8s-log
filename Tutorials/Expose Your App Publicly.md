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