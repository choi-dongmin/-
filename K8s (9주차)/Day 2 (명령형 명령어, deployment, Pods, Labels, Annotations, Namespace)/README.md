# 쿠버네티스 오브젝트 관리(명령형 / 선언형)
- kubctl을 사용하는 방법 : 명령형과 선언형
- 명령형은 수행하려는 작업을 지시하는 방식으로 적은 리소스에 대해서 처리가 빠르다는 장점을 가진다.
- 선언형은 원하는 상태를 선언하는것으로 시스템이 선언에 된 상태로 맞춰간다. 이 점에서 코드로 관리할 수있어 git을 통해서 관리할 수 있고 그로인해 코드의 변경과정을 알 수 있기 유지가 용이하고 선언에 맞춰가는 특성상 멱등성이 보장된다.

# 명령형 오브젝트 구성(Imperative Object Configuration)
- Obj을 YAML 혹은 JSON 파일 형식으로 정의함
- 일회성 작업에 권장되고 가장 단순한 방식
- 적은 리소스에 대한 빠른 처리

## Deployment
![화면 캡처 2022-06-21 173319](https://user-images.githubusercontent.com/57117748/174754982-983a1c23-4581-4008-a3a9-32ba81e194e2.png)
- ReplicaSet과 Pod 정보를 정의하는 상위 오브젝트
- ReplicaSet : 포드의 개수을 유지 시켜주는 오브젝트(그러나 1 pods 에 1개의 컨테이너 권장)
- Pod : 1개 이상의 컨테이너로 이루어진 컨테이너들의 집합

* Deployment yml 예시
```
apiVersion: apps/v1

# 해당 yml 이 deployment 로 생성할 것임을 명시
kind: Deployment 
metadata:
  name: my-nginx-deployment
# 유지할 포드의 개수를 1으로 레플리카셋에 지정
spec:
  replicas: 1
  selector:
    # my-nginx 라는 라벨의 파드를 유지
    matchLabels:
      app: my-nginx 
# 포드에 대한 설정 정보
  template: 
    metadata:
      name: my-nginx-pod
      labels:
      	# 파드의 라벨
        app: my-nginx
    # 포드 생성 스펙 
    spec: 
      containers:
        - name: nginx
          image: nginx
          ports:
            - containerPort: 80
```

* 명령어로 생성
```
$ kubectl create deployment my-nginx-deployment --image=nginx
```

* 확인
```
$ kubectl get deployments,replicasets,pods -o wide
```

## Pods
- 쿠버네티스의 기본 구성 요소
- 쿠버네티스의 객체 모델 중에 서 만들고 배포할 수 있는 가장 작은 단위
- 쿠버네티스 클러스터 내에서 애플리케이션을 배포하며 동작하는 프로세스
- 1개 이상의 컨테이너로 이루어진 컨테이너들의 집합
- 컨테이너 : 이미지가 메모리에 올라가 실행되는 상태 

* 파드 정의 시 주의사항
1. Image 이름
2. 포트는 이미지에서 정의한 어플리케이션이 실제 사용하는 포트를 지정해야함
3. 파드의 레이블은 컨트롤러의 셀렉터 / 서비스 셀렉터와 일치시켜야함

* Pods 주요 필드
∙ .spec.containers: 컨테이너 정의
∙ .spec.containers.image: 컨테이너에 사용할 이미지
∙ .spec.containers.name: 컨테이너 이름
∙ .spec.containers.ports: 노출할 포트 정의
∙ .spec.containers.ports.containerPort: 노출할 컨테이너 포트번호
∙ .spec.containers.ports.protocol: 노출할 컨테이너 포트의 프로토콜

* 오브젝트를 구성하는 메니페시트 작성시 필수 작성 요소
```
yml 로 작성

apiVersion: v1
kind: Pod
metadata:
  name: pod-test
spec:
 containers
    - name: myapp
      image: arisu1000/simple-container-app:latest
      ports:
      - containerPort: 8080
```

* 파드 생성
```
testapp-pod.yml

apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod
spec:
  containers:
  - image: c1t1d0s7/myweb
    name: testapp
    ports:
    - containerPort: 80
      protocol: TCP
```
```
kubectl create -f testapp-pod.yml
```
```
// 실행중인 파드 정의 확인
$ kubectl get pods testapp -o yaml 	// YAML 형태로 출력
$ kubectl get pods testapp -o json 	// JSON 형태로 출력

// 파드의 자세한 정보 확인
$ kubectl describe pods testapp

// 파드 로그 확인
$ kubectl logs testapp

 파드 포트 포워딩
$ kubectl port-forward testapp 80:80
```

## 파드의 구성 패턴
- 파드로 여러 컨테이너를 묶어 구성하고 실핼할 때 몇가지 패턴을 적용시킬 수 있다.
1. Sidecar : 메인컨테이너의 기능을 확장 또는 향상
2. Ambassador : 메인컨테이너의 네트워크기능 담당
3. Adapter : 메인컨테이너의 출력을 변환

### 사이드카 패턴(Sidecar)
- **사이드카 패턴**은 기본 애플리케이션 서비스 컨테이너는 원래 목적에 충실하도록 설정
- 나머지 공통 부가 기능은 사이드카 컨테이너에 넣어 사용

> k8s Sidecar Pattern
![Sidecar](https://d33wubrfki0l68.cloudfront.net/b7b7a33a62a27dead666a7c5ffc61cb89eeecf78/040b2/images/blog/2015-06-00-the-distributed-system-toolkit-patterns/sidecar-containers.png)

- 웹서버 컨테이너는 웹서버 역할만 하고 로그는 그저 파일로 남겨진다.
- 사이드카 역할의 로그 수집 컨테이너가 파일 시스템에 쌓이는 로그를 수집해 외부의 로그 수집 시스템으로 보낸다.
- 이러한 방법으로 구성시 **웹** 서버의 **컨테이너**를 **다른** 컨테이너로 **바꾸더라도** **로그** 수집 **컨테이너**는 **그대로** 사용가능
- 즉, **공통 역할**을 하는 **컨테이너**의 **재사용성**을 높일 수 있다.  

### 앰버서더 패턴(Ambassador)
- 앰버서더 패턴은 메인컨테이너의  **네트워크 연결 **을 전담하는  **프록시 컨테이너**를 두는 패턴.
- **메인 컨테이너**는 기능 **자체**에 **집중**할 수 있고 **네트워크 컨테이너**에서는 **네트워크 기능**에 **집중**할 수 있게 된다.
![Ambassador](https://user-images.githubusercontent.com/15958325/71399761-06c13100-2668-11ea-8f92-67b29984afa7.png)

## 어댑터 패턴(Adapter)
- 어댑터 패턴은 외부로 노출되는 정보를 표준화하는 어댑터 컨테이너를 사용한다.
![Adapter](https://user-images.githubusercontent.com/15958325/71400235-6bc95680-2669-11ea-86a0-3254e3a60583.png)

- 예로 **모니터링 pod**가 존재하고 **다른 pod**들의 **모니터링** 정보를 **수집**하는 경우 모니터링 pod에서 다른 **pod들**의 **cpu**사용량과 **ram**사용량 **정보만 뽑아서** 보고싶다고 할때, **메인컨테이너**의 출력은 **변환하지 않고** **어댑터컨테이너**만 수정하여 일관된 **정보를 모니터링 pod에게** 줄 수 있습니다.

## 파드 자원 할당
- request : 해당 파드를 생성할 때 노드 선택에 참조 즉, 최소 리소스(request) 이상 만큼 생성 + 해당 파드가 원할하게 동작
- limit : 동작 중인 파드에 대해 제한 (독점방지), 즉 파드 생성 후 자원 사용량 제한 + 다른 파드들이 원활하게 동작
```
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod
spec:
  containers:
  - image: c1t1d0s7/myweb
    name: testapp
    resources:
      requests:
        cpu: 1
	memory: 200M
      limit:
        cpu: 0.5
	memory: 1G
    ports:
    - containerPort: 80
      protocol: TCP
```

## Label
- K8s 클러스터에 모든 오브젝트에 Key:value 형태로 존재
- 리소스를 식별하고 속성을 확인하는데 사용
- Obj가 많아지면서 식별을 할 수있게 오브젝트가 어떤 성격, 역할을 가지는지 설명을 위함
- 레이블 셀렉터를 통해 오브젝트에 부여되어 있는 레이블을 식별하고 검색할 수 있음
	1. 일치성 기준(Equality-based) 요건
		∙ =
		∙ ==
		∙ != 
	2. 집합성 기준(Set-based) 요건
		∙ in
		∙ notin 

* labels 가 포함된 yaml 파일
```
testapp-pod-label.yml

apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod-label
  labels:
    env:dev
    tire:frontend

spec:
  containers:
  - image: c1t1d0s7/myweb
    name: testapp
    ports:
    - containerPort: 80
      protocol: TCP
```
```
// label 확인
$ kubectl get 오브젝트 --show-label

// 특정 레이블 키를 가진 파드 확인
$ kubectl get pods --show-labels -l env

// 특정 레이블 키=값을 가진 파드 확인
$ kubectl get pods --show-labels -l env=dev

// 기존 pod에 label 생성
$ kubectl label pods 기존파드 key=value

// 기존 pod label 수정
$ kubectl label pods 기존파드 key=value --overwrite
```

## Annotation
- 주석과 같은 역할.
- 오브젝트에 비-식별 메타데이터를 지정할 수 있음
- 레이블과 비슷하게 key/value 값을 가지지만 셀렉터를 가지고 있지 않다.

* 어노테이션을 포함한 yaml 파일
```
testapp-pod-anno.yml

apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod-anno
  annotations: 
    test: "abcd"
    test: "1234"
spec:
  containers:
  - image: c1t1d0s7/myweb
    name: testapp
    ports
    - containerPort: 80
      protocol: TCP
```

* 명령어를 통한 어노테이션 부여
```
$ kubectl annotate pods 파드명 "[Key]"="[Value]"
```

* 확인
```
// 명령어를 이용한 어노테이션
$ kubectl annotate 파드명 key="값"

// 어노테이션 확인
$ kubectl get pods 파드명 -o yaml
$ kubectl get pods testapp-pod-anno -o yaml | grep -i -A 3 'annotation'
$ kubectl get pods testapp-pod-anno -o json | grep -i -A 3 'annotation'
$ kubectl describe pods testapp-pod-anno -o yaml | grep -i -A 3 'annotation'
```

## Namespace
- 오브젝트를 논리적으로 구분할 수 있도록 만들어주는 논리적인 파티션
![화면 캡처 2022-06-21 182645](https://user-images.githubusercontent.com/57117748/174766417-ce36ccfc-bd42-4f91-9429-c23cc685259e.png)

1. default 
	- default namespace

2. kube-node-lease
	- 쿠버네티스 노드의 가용성을 체크하기 위한 네임스페이스
	- 해당 네임스페이스에는 하트비트를 위한 리스(Leases) 오브젝트가 있음

3. kube-public
	- 모든 사용자(인증받지 않은 사용자 포함)가 읽기 권한으로 접근할 수 있음
	- 관례적으로 만들어져 있지만 아무 리소스도 없고, 꼭 사용할 필요 없음

4. kube-system
	- 쿠버네티스 클러스터의 핵심 리소스 배치

5. ingress-nginx
	- 쿠버네티스 인그레스(Ingress) 리소스를 위한 Nginx 인그레스 컨트롤러 배치

6. rook-ceph
	- Rook Ceph 스토리지 관련 리소스 배치   

* Docker Namespace vs K8s Namespace
1. 도커에서 사용하는 네임스페이스
	- **리눅스**의 기능
	- **컨테이너의 리소스를 격리**해주기 위한 용도

2. 쿠버네티스에서의 네임스페이스
	- **쿠버네티스**에서 **관리**하는 **오브젝트** 중의 하나
	- 쿠버네티스에서 **오브젝트**를 **격리**하는 용도
	- 목적에 따라 사용하는 경우가 많다


* namespace 확인
```
// 네임스페이스 확인
$ kubectl get namespaces
$ kubectl get ns

// 특정 네임스페이스 목록값 확인
$ kubectl get all -n 네임스페이스명

// 특정 네임스페이스의 파드 확인
$ kubectl get pods -n 네임스페이스명

// 모든 네임스페이스의 파드 확인
$ kubectl get namespaces
```

* 네임스페이스 생성 yml
```
namespace.yml

apiVersion: v1
kind: Namespace
metadata:
  name: myns1
```

* 명령어 네임스페이스 생성
```
$ kubectl create namespace myns2
```

* 특정 네임스페이스 지정 파드 생성
- 파드 생성 시 명령어 기반
```
$ kubectl create -f xxxxxxx.yml -n myns1
```

- yaml 파일 기반
```
testapp-pod-ns.yml

apiVersion: v1
kind: Pod
metadata:
  name: xxxxxxx
  namespace: myns2
spec:
  containers:
  - image: c1t1d0s7/myweb
    name: testapp
    ports:
    - containerPort: 80
      protocol: TCP
```

* 오브젝트 이름 지정 삭제
```
$ kubectl delete pods 파드명 -n myns1
```

## 명령형 명령어 사용
- Deployment 생성
```
$ vim deployment.yml
```
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-deployment
spec:
  replicas: 3
  selector:
    # maintain test-deploy label
    matchLabels:
      app: test-deploy

# pods configration
  template:
    metadata:
      name: test-pods
      labels:
        app: test-deploy
    spec:
      containers:
        - name: test
          image: nginx
          ports:
            - containerPort: 80
...
```
* deployment 불러오기
```
$ kubectl create deployment myapp --image=ghcr.io/c1t1d0s7/go-myweb
```

- kubectl apply check
```
$ kubectl apply -f test-deployment
```
```
$ kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
test-deployment   3/3     3            3           5m33s
$ kubectl get replicaset
NAME                       DESIRED   CURRENT   READY   AGE
test-deployment-5d8ff5d5   3         3         3       5m40s
$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
test-deployment-5d8ff5d5-5wj2t   1/1     Running   0          5m46s
test-deployment-5d8ff5d5-ffjmv   1/1     Running   0          5m46s
test-deployment-5d8ff5d5-l7l2t   1/1     Running   0          5m46s


$ kubectl get deployments,replicasets,pods
.
.
.

```

- service 생성(service : 외부 네트워크와 연결)
```
$ kubectl expose deployment test-deployment --port=80 --protocol=TCP --target-port=80 --name deploy-svc --type=LoadBalancer

$ kubectl get service
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
deploy-svc   LoadBalancer   10.233.3.20   <pending>     80:32237/TCP   8s
kubernetes   ClusterIP      10.233.0.1    <none>        443/TCP        26h
```

- Pods Scale chagne
```
$ kubectl scale deployment test-deployment --replicas=1
deployment.apps/test-deployment scaled

$ kubectl get pods
NAME                             READY   STATUS    RESTARTS   AGE
test-deployment-5d8ff5d5-fh66t   1/1     Running   0          2m43s
```

- 확인하기
```
$ kubectl get all
NAME                                 READY   STATUS    RESTARTS   AGE
pod/test-deployment-5d8ff5d5-fh66t   1/1     Running   0          3m45s

NAME                 TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/deploy-svc   LoadBalancer   10.233.39.47   <pending>     80:30132/TCP   3m35s
service/kubernetes   ClusterIP      10.233.0.1     <none>        443/TCP        27h

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/test-deployment   1/1     1            1           3m45s

NAME                                       DESIRED   CURRENT   READY   AGE
replicaset.apps/test-deployment-5d8ff5d5   1         1         1       3m45s
```

## --dry-run
- 만약 어떠한 **tamplate**을 **작성**해야 하는데 해당 **yaml** 파일의 **구조**가 **기억**이 나지 **않을때**, 혹은 하나 하나 **요소**들을 **기입**하기 **번거러울때** 사용 할 수 있는 옵션이다
- `--dry-run=client -o yaml`

* 명령어 옵션 사용시 아래처럼 동작
```bash
kubectl create deployment test-dep --image=nginx --dry-run=client -o yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: test-dep
  name: test-dep
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-dep
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: test-dep
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```
![화면 캡처 2022-07-12 111243](https://user-images.githubusercontent.com/57117748/178399407-4a5875a3-feba-4907-a8df-803d280b4e02.png)
![화면 캡처 2022-07-12 111248](https://user-images.githubusercontent.com/57117748/178399413-c7a5fcd9-ff0f-4022-a08e-1b7dd8a5fa19.png)

## 키워드
- deployments : replicaSet과 Pod 정보를 정의하는 상위 오브젝트
- replicaSet : 포드의 개수을 유지 시켜주는 오브젝트 replicaSet=3 상태에서 3개의 파드중 하나를 삭제해도 3을 맞춰주기 위해 다시 1개 더 만든다
- pods: K8s의 오브젝트 단위 중 가장 작은 단위로 컨테이너들의 집합 클러스터 내에서 애플리케이션을 배포하며 동작하는 프로세스
- Label: 리소스를 식별하고 속성을 확인하기 위해 오브젝트에 Key:Value 형태로 성격 및 목적을 정의하는 항목, 레이블 셀렉터를 통해 오브젝트에 부여되어 있는 레이블을 식별한다 그러나 쿠버네티스 클러스터에 직접적 영향은 없다 
- Annotation : 오브젝트내의 주석과 같은 역할로 비 식별 메타데이터이다. 레이블과 비슷하게 Key:Value 값으로 이루어 지지만 셀렉터가 존재하지 않는다
- namespace : 오브젝트들을 논리적으로 나누기 위한 논리적 파티션으로 default, kube-node-lease, kube-public, kube-system, ingress-nginx, rook-ceph 으로 이루어져있다.
- default(namespace) : default namespace로 k8s 오브젝트를 만들시 기본으로 정의된다.
- kube-node-lease : 노드의 가용성을 체크하기 위한 네임스페이스 하트비트를 위한 리스(Leases) 오브젝트가 존재
- kube-public : 누구나 읽기 권한으로 접근할 수 있고 관례적으로 존재하지만 리소스를 차지 하지 않으며 사용하지 않을 수 있다.
- kube-system : k8s 클러스터 시스템 핵심 프로세스 리소스 모음 
