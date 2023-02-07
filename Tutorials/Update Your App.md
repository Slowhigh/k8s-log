# **Update Your App**

### 애플리케이션 업데이트하기
- `Rolling Update`는 `Pod` 인스턴스를 점진적으로 새로운 것으로 업데이트하여 `Deployment` 업데이트가 서비스 중단 없이 이루어질 수 있도록 해준다.
- 업데이트가 이루어지는 동안 이용 불가한 파드의 최대 개수와 생성 가능한 새로운 파드의 최대 개수는 각각 1개 이다.

<img src="image/module_06_rollingupdates1.svg" alt="Rolling updates overview" width="49%"/>
<img src="image/module_06_rollingupdates2.svg" alt="Rolling updates overview" width="49%"/>
<img src="image/module_06_rollingupdates3.svg" alt="Rolling updates overview" width="49%"/>
<img src="image/module_06_rollingupdates4.svg" alt="Rolling updates overview" width="49%"/>

- `Deployment`가 외부로 노출되면, `Service`는 업데이트가 이루어지는 동안 오직 가용한 `Pod`에게만 트래픽을 로드밸런스 한다.
- 이전 버전으로 롤백하는 기능을 제공해 준다.

<br>

### Rolling Update 하기
```bash
# kubectl create deployment [_Deployment Name_] --image=[_____________App Image Location___________] --replicas=[복제 개수]
$ kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1 --replicas=4

$ kubectl get deployments

$ kubectl get pods

# [_P_] : The range of valid ports is 30000-32767
# kubectl create service nodeport [_Deployment Name_] --node-port=[_P_] --tcp=8080:8080
$ kubectl create service nodeport kubernetes-bootcamp --node-port=30000 --tcp=8080:8080

# kubectl describe service [__Service Name___]
$ kubectl describe service kubernetes-bootcamp

# curl localhost:[_P_]
$ curl localhost:30000
# -------------------------------------------------------------------------------------------------
# Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-75c5d958ff-mb9pv | v=1

# kubectl set image deployment [_Deployment Name_] [_Container Name__]=[______New Image Location______]
$ kubectl set image deployment kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2

# kubectl rollout status deployment [_Deployment Name_]
$ kubectl rollout status deployment kubernetes-bootcamp
# -------------------------------------------------------------------------------------------------
# deployment "kubernetes-bootcamp" successfully rolled out

$ kubectl get pods

# curl localhost:[_P_]
$ curl localhost:30000
# -------------------------------------------------------------------------------------------------
# Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-c564db98f-cdnnw | v=2

# kubectl set image deployment [_Deployment Name_] [_Container Name__]=[___________Invalid Image Location__________]
$ kubectl set image deployment kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10

# kubectl rollout status deployment [_Deployment Name_]
$ kubectl rollout status deployment kubernetes-bootcamp
# -------------------------------------------------------------------------------------------------
# Waiting for deployment "kubernetes-bootcamp" rollout to finish: 2 out of 4 new replicas have been updated...
# ...
# ^C

$ kubectl get pods

# curl localhost:[_P_]
$ curl localhost:30000
# -------------------------------------------------------------------------------------------------
# Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-c564db98f-cdnnw | v=2

# kubectl rollout undo deployment [_Deployment Name_]
$ kubectl rollout undo deployment kubernetes-bootcamp

# kubectl rollout status deployment [_Deployment Name_]
$ kubectl rollout status deployment kubernetes-bootcamp
# -------------------------------------------------------------------------------------------------
# deployment "kubernetes-bootcamp" successfully rolled out

$ kubectl get pods

# curl localhost:[_P_]
$ curl localhost:30000
# -------------------------------------------------------------------------------------------------
# Hello Kubernetes bootcamp! | Running on: kubernetes-bootcamp-c564db98f-cdnnw | v=2
```