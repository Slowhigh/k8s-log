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