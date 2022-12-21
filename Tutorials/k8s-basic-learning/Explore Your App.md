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