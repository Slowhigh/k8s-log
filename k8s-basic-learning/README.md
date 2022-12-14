# Cluster 생성하기

## Minikube를 사용해서 Cluster 생성하기
### Kubernetes Cluster
<img src="image/module_01_cluster.svg" alt="`Cluster Diagram" width="100%"/>

- `Kubernetes`는 컴퓨터들을 연결하여 단일 형상으로 동작하도록 컴퓨팅 `Computing Cluster`를 구성하고 높은 가용성을 제공하도록 조율한다. 
- `Kubernetes`는 이러한 애플리케이션 컨테이너를 `Cluster`에 분산시키고 스케줄링하는 일을 더욱 효율적으로 자동화한다.
- `Control Plain`은 `Cluster` 관리를 담당한다. `Control Plain`은 애플리케이션을 스케줄링하거나, 애플리케이션의 항상성을 유지하거나, 애플리케이션을 스케일링하고, 새로운 변경사항을 순서대로 반영(rolling out)하는 일과 같은 `Cluster` 내 모든 활동을 조율한다.
- `Node`는 `Cluster` 내 워커 머신으로 동작하는 VM 또는 물리적인 컴퓨터다. 
- 각 `Node`는 `Node`를 관리하고 `Control Plain`과 통신하는 `Kubelet`이라는 에이전트를 갖는다. 
- `Node`는 컨테이너 운영을 담당하는 `containerd 또는 Docker`와 같은 툴도 갖는다. 
- `Node`는 `Control Plain`이 제공하는 `Kubernetes API`를 통해서 `Control Plain`과 통신한다. 

```bash
$ minikube version

$ minikube start

$ kubectl version

$ kubectl cluster-info

$ kubectl get nodes
```

# 앱 배포하기

## kubectl을 사용해서 Deployment 생성하기
### Kubernetes Deployment
<img src="image/module_02_first_app.svg" alt="Kubernetes에 첫 번째 애플리케이션 배포하기" width="100%"/>

- 일단 `Cluster`를 구동시키면, 그 위에 컨테이너화된 애플리케이션을 배포할 수 있다.
- `Deployment`는 `Kubernetes`가 애플리케이션의 인스턴스를 어떻게 생성하고 업데이트해야 하는지를 지시한다.
- `Deployment Controller`는 머신의 장애나 정비에 대응할 수 있는 자동 복구(self-healing) 메커니즘을 제공한다.

```bash
$ kubectl get nodes

# kubectl create deployment [_deployment name_] --image=[_____________app image location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get deployments

# open a new terminal and run the `kubectl proxy`
$ kubectl proxy

$ curl http://127.0.0.1:8001/version

$ kubectl get pods

# curl http://localhost:8001/api/v1/namespaces/default/pods/[______________pod name____________]/
$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubernetes-bootcamp-75c5d958ff-96n6l/

# kubectl delete deployment [_deployment name_]
$ kubectl delete deployment kubernetes-bootcamp
```

# 앱 조사하기

## Pod와 Node 보기
### Kubernetes Pod
<img src="image/module_03_pods.svg" alt="Pod 개요" width="100%"/>

`Pod`는 하나 또는 그 이상의 애플리케이션 컨테이너 (`Docker`와 같은)들의 그룹을 나타내는 `Kubernetes`의 추상적 개념으로 일부는 컨테이너에 대한 자원을 공유한다. 그 자원은 다음을 포함한다.
- 볼륨과 같은, 공유 스토리지
- `Cluster` IP 주소와 같은, 네트워킹
- 컨테이너 이미지 버전 또는 사용할 특정 포트와 같이, 각 컨테이너가 동작하는 방식에 대한 정보

### Kubernetes Node
<img src="image/module_03_nodes.svg" alt="Node 개요" width="100%"/>

`Pod`는 언제나 `Node` 상에서 동작한다. 
`Node`는 `Kubernetes`에서 워커 머신을 말하며 `Cluster`에 따라 가상 또는 물리 머신일 수 있다.

모든 `Node`는 최소한 다음과 같이 동작한다.
- Kubelet은, `Control Plain`과 `Node` 간 통신을 책임지는 프로세스이며, 하나의 머신 상에서 동작하는 `Pod`와 컨테이너를 관리한다.
- `Container Runtime`(`Docker`와 같은)은 레지스트리에서 컨테이너 이미지를 가져와 묶여 있는 것을 풀고 애플리케이션을 동작시키는 책임을 맡는다.

```bash
# kubectl create deployment [_deployment name_] --image=[_____________app image location___________]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

$ kubectl get pods

$ kubectl describe pods

# open a new terminal and run the `kubectl proxy`
$ kubectl proxy

# curl http://localhost:8001/api/v1/namespaces/default/pods/[______________pod name____________]/proxy/
$ curl http://localhost:8001/api/v1/namespaces/default/pods/kubernetes-bootcamp-75c5d958ff-96n6l/proxy/

# kubectl logs [______________pod name____________]
$ kubectl logs kubernetes-bootcamp-75c5d958ff-96n6l

# kubectl delete deployment [_deployment name_]
$ kubectl delete deployment kubernetes-bootcamp
```

##