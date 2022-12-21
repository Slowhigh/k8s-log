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