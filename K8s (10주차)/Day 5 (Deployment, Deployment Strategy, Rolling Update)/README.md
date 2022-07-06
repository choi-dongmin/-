# Deployment
- 애플리케이션의 업데이트, 배포를 더욱 편리하게 하기위해서 사용한다. 즉, "애플리케이션을 배포, 관리하는 역할을 담담"
- 시스템의 처리 능력을 분산시키고 장애가 발생한 서버들을 제외한 나머지 서버에게 부하를 분산하는 기능을 하는 오브젝트.
- 무중단 배포 방식 지원
- [K8s Deployment Doc](https://kubernetes.io/ko/docs/concepts/workloads/controllers/deployment/)

* Deployment
![Deployment](https://blog.kakaocdn.net/dn/mVW7g/btqFNFa7u1E/RTURWaMRdcLNpDzxj3GAwk/img.png)

## Deployment 기능
- 배포관리
	1. rollback : Deployment 배포 이력 관리
	2. 배포 전략 선언

- replicaSet 단위로 배포, 관리

## 배포 전략
1. Re-Create
	- 가장 일반적인 방법으로 동작중이던 파드를 중지, 삭제 후 새로운 파드를 생성, 동작하는 절차를 가진 배포
	- 위와같은 배포 절차로인하여 다운타임이 필연적으로 발생
	- 그로 인해 해당 App을 막아놓고 작업을 진행함
	- 낮은 비용

2. Rolling Update
	- 업데이트를 진행 할때에 여러 파드들을 하나씩 전환시킴
	- 모두 전환 될 때까지 시간이 오래 걸림
	- 낮은 비용

3. Blue-Green Update
	- Blue = v1, Green = v2
	- 업데이트를 진행 할때에 두가지 버전을 모두 배포 진행중 동시에 전환을 시킨다.
	- 즉, 한꺼번에 삭제, 전환이 이루어지며 문제가 발생시 해결이 어렵다 그로인해 애플리케이션에 대한 검증이 잘 되어야 한다.
	![Blue-Green Updata](https://postfiles.pstatic.net/MjAyMTExMjhfMjI2/MDAxNjM4MDk4OTAyMzUx.PbMTGuKmwqenjBVLv5UH0wafEnKnMnT1PLEz5h-yC-gg.-zXKWFZJIvJom9oy1oG-3zJWpX_-pcU7irqRwDt1af4g.PNG.koys007/image.png?type=w773)

4. Canary Update
	- 카나리아가 유독가스에 민감하여 사람의 치사량보다 적은 양으로 죽는다는 점을 착안한 방법으로 카나리아 배포 방법은 위험을 빠르게 감지할 수 있는 배포 방법
	- v1 파드들이 동작중에, v2 파드들을 미리 생성하여 조금씩 1%.. 전환을 시킴, 시간이 흐름에 따라 % 가 늘어감
	- 조금씩 트래픽을 전환한다
	- 문제가 없으면 100% 전환 됨
	- 단, 어떤 클라이언트에 대해서 전환을 진행하는지 알 수 없고 불특정 다수에게 무작위 전환
	![화면 캡처 2022-07-01 193556](https://user-images.githubusercontent.com/57117748/176878891-87f5d0b0-ed36-4af1-80a1-198201fd3c87.png)


5. A/B testing
	- 기본적인 배포방식은 Canary 배포 방식과 비슷하다
	- 하지만 A / B testing 은 특정한 대상을 위주로 트래픽 전환진행
	- 제품, application 기능을 확인할 때 주로 사용함.
	- Ex ) 일반 브라우저 : v1  / 모바일 : v2

6. Shadow
	- 구현이 어려움
	- v1 운영 중 v2 를 생성
	- v1로 들어오는 트래픽들을 복제해 v2로 전송

## Deployment 기본 명령어
- Deployment Create Command
```
$ kubectl apply -f [yaml 파일]	
```

- Deployment Delete Command
```
kubectl delete -f [yaml 파일]	
```

- Deployment yaml Composition
```
apiVersion : apps/v1 설정
kind : Deployment
metadata :
  name: 
spec : 
  replicas : Number of Pods 
  selector : Finding and Matching with Labels. 
    matchLabels:
      [key] : [value]
  template : 
    metadata:
	  labels: if selector can't find pods that got same labels then make new pods 
	spec:
	  containers: container spec
```

## Deployment 생성
- Deployment Manifest 작성
```
vim nginx-deployment.yml
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx
        image: nginx:1.16
```

- nginx-deployment 생성
```
kubectl apply -f nginx-deployment.yml
```

- 생성 확인
```
$ kubectl get deployments.apps
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           3m37s
```

* 기본적으로 deployment는 replicaSet 단위로 관리되기 때문에 파드유지, scale out 등 을 지원한다.

## Deployment Update
- rollout 이라고도 하며 쿠버네티스 클러스터에 새로운 소프트웨어나 서비스를 개시하는 기능이다. 즉, 컨테이너의 버전을 업데이트하는 것.

* nginx-deployment update

- deployment에 속한 컨테이너 중 이전 버전 확인
```
$ kubectl describe pod nginx-deployment-86b995658-2htgq | grep "image"
  Normal  Pulling    18m   kubelet            Pulling image "nginx:1.16"
  Normal  Pulled     18m   kubelet            Successfully pulled image "nginx:1.16"
```

- nginx image version upgrade
```
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.17
```

- update 확인
```
$ kubectl describe deployments.apps nginx-deployment | tail -10
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  28m    deployment-controller  Scaled up replica set nginx-deployment-86b995658 to 3
  Normal  ScalingReplicaSet  6m43s  deployment-controller  Scaled up replica set nginx-deployment-77b4d8bbb9 to 1
  Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled down replica set nginx-deployment-86b995658 to 2
  Normal  ScalingReplicaSet  6m25s  deployment-controller  Scaled up replica set nginx-deployment-77b4d8bbb9 to 2
  Normal  ScalingReplicaSet  6m8s   deployment-controller  Scaled down replica set nginx-deployment-86b995658 to 1
  Normal  ScalingReplicaSet  6m8s   deployment-controller  Scaled up replica set nginx-deployment-77b4d8bbb9 to 3
  Normal  ScalingReplicaSet  5m51s  deployment-controller  Scaled down replica set nginx-deployment-86b995658 to 0

$ kubectl describe pod nginx-deployment-77b4d8bbb9-pwxrw | grep "image"
  Normal  Pulling    3m35s  kubelet            Pulling image "nginx:1.17"
  Normal  Pulled     3m23s  kubelet            Successfully pulled image "nginx:1.17"
```

- rollout 확인
```
$ kubectl rollout status deployment/nginx-deployment
deployment "nginx-deployment" successfully rolled out



$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         <none> 	// 한번 업데이트 된것을 확인
```

## rollback

- rollback
```
$ kubectl rollout undo deployment nginx-deployment
deployment.apps/nginx-deployment rolled back
```

- 확인
```
$ kubectl rollout history deployment/nginx-deployment
deployment.apps/nginx-deployment
REVISION  CHANGE-CAUSE
2         <none>
3         <none>

$ kubectl describe pod nginx-deployment-86b995658-fr5mh | grep "image"
  Normal  Pulled     91s   kubelet            Container image "nginx:1.16" already present on machine
```

# 무중단 배포
- 무중단 배포란 서버를 실제로 서비스할 때 서비스적인 장애와 배포에 있어서 부담감을 최소화하고, "서비스가 중단되지 않도록 배포"하는 기술
- 온프레미스 환경에서는 실질적으로 무중단 배포를 구현하기 어려웠지만 클라우드 환경에 들어오면서 무중단 배포를 하는데 더 편하게 진행할 수 있다.

* 트래픽 경로
![트래픽 경로](http://tech.kakao.com/files/kubernetes-traffic.jpg)


## Rolling Update
- Rolling Update는 새 버전을 배포하면서, 새 버전 인스턴스를 하나씩 늘려가고 기존 버전의 인스턴스를 하나식 줄여나가는 방식입니다. 
- 이 경우 새로운 버전의 파드와 이전 버전의 파드가 동시에 존재 할 수 있다는 단점이 있다
- 쿠버네티스가 addon을 하지 않고 지원 가능한 유일한 무중단 배포 방식

![Rolling Update](https://blog.kakaocdn.net/dn/Rablv/btqFKCMzjIB/yrXxS13TlM2zI0Kkj3Zqi0/img.gif) 

* Rolling Update Manifest 작성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-ru
  labels:
    app: nginx-webs
spec:
  replicas: 10
  minReadySeconds: 1 		# Rolling Update 속도 설정 Pod의 status 가 Ready가 될 때까지의 최소 대기 시간. 설정 시간(초단위) 동안 트래픽 받지 않음
  strategy:
    type: RollingUpdate 	# 배포 시 RollingUpate / default 값
    rollingUpdate:
      maxSurge: 1   		# Rolling Update 시 동시에 생성할 수 있는 Pod의 최대 개수 설정.
      maxUnavailable: 1 	# Rolling Update 시 동시에 삭제할 수 있는 Pod의 최대 개수 설정
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx-webs
        image: nginx:1.16 	# 1.16 버전의 nginx 사용
        ports:
        - containerPort: 8080
```

* 실행 
```
$ kubectl apply -f deployment-ru.yml
```

* 생성 확인
```
$ kubectl get all -o wide
NAME                                 READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
pod/deployment-ru-85cb58b4cc-4htr9   1/1     Running   0          7m47s   10.233.101.151   kube-node1   <none>           <none>
pod/deployment-ru-85cb58b4cc-56462   1/1     Running   0          7m47s   10.233.76.187    kube-node3   <none>           <none>
pod/deployment-ru-85cb58b4cc-6g9zz   1/1     Running   0          7m47s   10.233.103.144   kube-node2   <none>           <none>
pod/deployment-ru-85cb58b4cc-lgtq2   1/1     Running   0          7m47s   10.233.76.185    kube-node3   <none>           <none>
pod/deployment-ru-85cb58b4cc-t277m   1/1     Running   0          7m47s   10.233.103.142   kube-node2   <none>           <none>
pod/deployment-ru-85cb58b4cc-v97s8   1/1     Running   0          7m47s   10.233.101.152   kube-node1   <none>           <none>
pod/deployment-ru-85cb58b4cc-vw5h6   1/1     Running   0          7m47s   10.233.103.143   kube-node2   <none>           <none>
pod/deployment-ru-85cb58b4cc-wpzrj   1/1     Running   0          7m47s   10.233.101.153   kube-node1   <none>           <none>
pod/deployment-ru-85cb58b4cc-xkjq9   1/1     Running   0          7m47s   10.233.76.184    kube-node3   <none>           <none>
pod/deployment-ru-85cb58b4cc-ztt8n   1/1     Running   0          7m47s   10.233.76.186    kube-node3   <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   3d1h   <none>

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES       SELECTOR
deployment.apps/deployment-ru   10/10   10           10          7m47s   nginx-webs   nginx:1.16   app=nginx-web

NAME                                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES       SELECTOR
replicaset.apps/deployment-ru-85cb58b4cc   10        10        10      7m47s   nginx-webs   nginx:1.16   app=nginx-web,pod-template-hash=85cb58b4cc
```

* 정보 업데이트(동일한 deployment-ru.yml 파일)
```
    spec:
      containers:
      - name: nginx-webs
        image: nginx:1.17 	# 1.17 버전으로 update
        ports:
        - containerPort: 8080
```

* 업데이트 실행
```
$ kubectl apply -f deployment-ru.yml
```

* Rolling Update 확인
```
$ kubectl get pods --watch
deployment-ru-d66fcf574-vksg8    0/1     Pending   0          0s
deployment-ru-85cb58b4cc-56462   1/1     Terminating   0          13m
deployment-ru-d66fcf574-vksg8    0/1     Pending       0          0s
deployment-ru-d66fcf574-vksg8    0/1     ContainerCreating   0          0s
deployment-ru-d66fcf574-nndj5    0/1     Pending             0          0s
deployment-ru-d66fcf574-nndj5    0/1     Pending             0          0s
deployment-ru-d66fcf574-nndj5    0/1     ContainerCreating   0          0s
deployment-ru-85cb58b4cc-56462   0/1     Terminating         0          13m
deployment-ru-d66fcf574-nndj5    1/1     Running             0          2s
deployment-ru-d66fcf574-vksg8    1/1     Running             0          2s
deployment-ru-85cb58b4cc-v97s8   1/1     Terminating         0          13m
deployment-ru-d66fcf574-sxprj    0/1     Pending             0          0s
deployment-ru-d66fcf574-sxprj    0/1     Pending             0          0s
.
.
.


$ kubectl get all -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP               NODE         NOMINATED NODE   READINESS GATES
pod/deployment-ru-d66fcf574-4rqxs   1/1     Running   0          3m42s   10.233.103.146   kube-node2   <none>           <none>
pod/deployment-ru-d66fcf574-65gpp   1/1     Running   0          3m32s   10.233.101.156   kube-node1   <none>           <none>
pod/deployment-ru-d66fcf574-gthkb   1/1     Running   0          3m29s   10.233.76.191    kube-node3   <none>           <none>
pod/deployment-ru-d66fcf574-hg7rf   1/1     Running   0          3m36s   10.233.76.190    kube-node3   <none>           <none>
pod/deployment-ru-d66fcf574-hgd9p   1/1     Running   0          3m33s   10.233.103.147   kube-node2   <none>           <none>
pod/deployment-ru-d66fcf574-jjlcq   1/1     Running   0          3m39s   10.233.76.189    kube-node3   <none>           <none>
pod/deployment-ru-d66fcf574-nndj5   1/1     Running   0          3m46s   10.233.101.154   kube-node1   <none>           <none>
pod/deployment-ru-d66fcf574-sxprj   1/1     Running   0          3m43s   10.233.76.188    kube-node3   <none>           <none>
pod/deployment-ru-d66fcf574-vksg8   1/1     Running   0          3m46s   10.233.103.145   kube-node2   <none>           <none>
pod/deployment-ru-d66fcf574-zjr5w   1/1     Running   0          3m37s   10.233.101.155   kube-node1   <none>           <none>

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE    SELECTOR
service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP   3d1h   <none>

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
deployment.apps/deployment-ru   10/10   10           10          17m   nginx-webs   nginx:1.17   app=nginx-web

NAME                                       DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES       SELECTOR
replicaset.apps/deployment-ru-85cb58b4cc   0         0         0       17m     nginx-webs   nginx:1.16   app=nginx-web,pod-template-hash=85cb58b4cc
replicaset.apps/deployment-ru-d66fcf574    10        10        10      3m46s   nginx-webs   nginx:1.17   app=nginx-web,pod-template-hash=d66fcf574


$ kubectl rollout history deployment deployment-ru
deployment.apps/deployment-ru
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

* kubernetes.io/change-cause (동일한 deployment-ru.yml)
- kubernetes.io/change-cause: "nginx:1.18 Rolling Update"
- nginx 1.18 update
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployment-ru
  labels:
    app: nginx-webs
  annotations:
    kubernetes.io/change-cause: "nginx-webs v1.18 Rolling Update"
spec:
  replicas: 10
  minReadySeconds: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: nginx-web
  template:
    metadata:
      labels:
        app: nginx-web
    spec:
      containers:
      - name: nginx-webs
        image: nginx:1.18
        ports:
        - containerPort: 8080
```

* 업데이트 실행
```
$ kubectl apply -f deployment-ru.yml
```

* kubectl rollout history deployment 확인
```
$ kubectl rollout history deployment deployment-ru
deployment.apps/deployment-ru
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         nginx-webs v1.18 Rolling Update
```

## Blue - Green Update
- Blue/Green은 기존에 띄워져 있는 Pod 개수와 동일한 개수의 신규 Pod를 생성 후 신규 Pod가 이상 없이 정상적으로 떴는지 확인, 들어오는 트래픽을 LoadBalaner 설정을 변경하여 한번에 신규 Pod 쪽으로 옮기는 방법
- Blue: Old Veriosn / Green : New Version
- 앱을 출시 하기전 준비하는 빠른 방법
- 배포판에 issue 가 감지되었을 시 빠르게 Rollback 할 수 있음
- 그러나 한 번에 두배의 Pod를 실행해야 하기 때문에 클러스터의 리소스가 2배 이상 필요함

* 진행 순서
```
1. Blue 배포 중 
2. update 버전의 Green 을 생성
3. Blue 으로 가던 트래픽을 Green 으로 변경하여 Update를 완료함
4. Green을 모니터링하다가 버그/오류가 발생하면 다시 Blue로 변경
5. Green에서 장시간 정상 동작을 확인하면 Blue를 제거한 후 Green을 Blue로 사용
```

![화면 캡처 2022-07-03 140355](https://user-images.githubusercontent.com/57117748/177025585-3aa82e84-8189-418c-858c-1235188e029e.png)

* Blue Manifest 작성
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blue-deployment
  labels:
    app: blue-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
        color: blue
    spec:
      containers:
      - name: blue-nginx
        image: nginx:1.16
        ports:
        - containerPort: 8080
```
```
$ kubectl apply -f blue-deployment
```


* Blue 와 연결할 ClusterIP Service와 ingress Service 생성
1. Service Label 설정이 blue=color 설정
2. image version 설정
```
$ vi service_ingress.yml
```
```
# ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: bluegreen-service
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: nginx-pod
    color: blue

---

# ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: bluegrenn-ingress
  namespace: default
spec:
  rules:
  - host: test.bluegreen.com
    http:
      paths:
      - path: /
        backend:
          serviceName: bluegreen-service
          servicePort: 8080
```
```
$ kubectl create -f service_ingress.yml
```

* 확인
1. Service Label 설정이 blue=color 확인
2. image version 확인
```
$ kubectl get all,ing -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP               NODE         NOMINATED NODE   READINESS GATES
pod/blue-deployment-55cfc89884-2vj24   1/1     Running   0          24m   10.233.76.203    kube-node3   <none>           <none>
pod/blue-deployment-55cfc89884-jm2pp   1/1     Running   0          24m   10.233.103.157   kube-node2   <none>           <none>

NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE     SELECTOR
service/bluegreen-service   ClusterIP   10.233.18.215   <none>        8080/TCP   11s     app=nginx-pod,color=blue
service/kubernetes          ClusterIP   10.233.0.1      <none>        443/TCP    4d15h   <none>

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES       SELECTOR
deployment.apps/blue-deployment   2/2     2            2           24m   blue-nginx   nginx:1.16   app=nginx-pod

NAME                                         DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES       SELECTOR
replicaset.apps/blue-deployment-55cfc89884   2         2         2       24m   blue-nginx   nginx:1.16   app=nginx-pod,pod-template-hash=55cfc89884

NAME                                   CLASS    HOSTS                ADDRESS   PORTS   AGE
ingress.extensions/bluegrenn-ingress   <none>   test.bluegreen.com             80      11s
```

* Green Manifest 생성 및 실행
- metadata.name:green-deployment 수정
- spec.template.metadata.labels.color:green 수정
- spec.template.spec.containers.name: green-nginx 수정
- spec.template.spec.containers.image: nginx:1.17
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: green-deployment
  labels:
    app: green-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx-pod
  template:
    metadata:
      labels:
        app: nginx-pod
        color: green
    spec:
      containers:
      - name: green-nginx
        image: nginx:1.17
        ports:
        - containerPort: 8080
```

* ClusterIP Service를 Update Green를 바라보게 함
- spec.selector.color:green
```
# ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: bluegreen-service
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: nginx-pod
    color: green

---

# ingress
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: bluegrenn-ingress
  namespace: default
spec:
  rules:
  - host: test.bluegreen.com
    http:
      paths:
      - path: /
        backend:
          serviceName: bluegreen-service
          servicePort: 8080
```


## 출처
[Deployment 이미지 출처](https://ooeunz.tistory.com/124)
[트래픽 경로 이미지 출처](https://tech.kakao.com/2018/12/24/kubernetes-deploy/)
[Rolling Update 이미지 출처](https://ooeunz.tistory.com/124)
[Blue- Green Update 이미지 출처](https://ikcoo.tistory.com/22)