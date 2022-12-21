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