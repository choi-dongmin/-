# 파드의 생명주기와 프로브
- 파드는 정의된 순서대로 진행하는 생명주기를 가진다
```
Pending -> Running -> Succeeded or Failed
```
- K8s의 구성요소의 kubelet은 컨테이너 상태를 추적하고 오류상태라면 정상으로 만들기 위해서 재시작할 수도 있다.
- 파드가 최초 생성될 때 한번만 스케줄링 되며, 노드에 할당되면 파드는 중지되거나 종료될 때까지 항상 같은 노드에서 실행

## 파드의 생명주기
- 파드는 영구적인 워크로드 리소스가 아닌 '임시의 워크로드 리소스'이다
- K8s 는 파드를 일회용으로 간주하며, 상위 레벨의 추상화 리소스인 컨트롤러 리소스를 이용해 파드를 관리한다.

* 파드의 단계
![Pod LifeCycle](https://static.packt-cdn.com/products/9781788834759/graphics/215ab89b-ed5d-4858-b4d7-b358adb6d0d6.png)
1. Pending
	- 파드가 클러스터에서 승인되었지만, 실행되고 있지 않음
	- 파드가 스케줄링되기 전
	- 이미지 풀링
2. Running 
	- 파드가 노드에 할당됨
	- 하나 이상의 컨테이너가 실행 중
	- 시작 또는 재시작 중
3. Succeeded
	- 모든 컨테이너가 정상적(0) 종료
	- 재시작되지 않음
4. Failed
	- 모든 컨테이너가 종료
	- 하나 이상의 컨테이너가 실패(Not 0)로 종료
5.Unknown
	- 알 수 없음
	- 일반적으로는 노드와의 통신 오류

* 파드 단계 확인
```
$ kubectl get pods 	// STATUS FILED에서 확인 가능
```

# 컨테이너 상태
- 스케줄러가 파드를 특정 노드에 할당시, kubelet은 컨테이너 런타임을 이용해 컨테이너를 생성하고 kubelet은 컨테이너 상태를 주기적으로 모니터링한다.

- Waiting
	- 컨테이너가 Running 또는 Terminated 상태가 아닌 상태
	- 컨테이너 시작 전 필요한 작업 실행중(이미지 풀링, 스토리지 연결 등)
	- Reason 필드에 이유가 표시
- Running
	- 실행 중
	- 해당 상태가 된 시각 표시
- Terminated
	- 컨테이너 실행이 완료됨
	- 실패
	- Reason 필드에 이유가 표시
	- 해당 상태가 된 시각 표시

* 컨테이너 상태 확인
```
kubectl describe pod  // Containers 항목
```

## 컨테이너 재시작 정책
- 모든 컨테이너는 컨테이너의 오류 발생 시 재시작을 하기 위한 재시작 정책을 가진다
- 파드의 컨테이너 재시작은 최대 300초로 제한된 지수 백오프(exponential back-off) 지연(10초, 20초, 40초, 80초 ...) 순차적 재시작
- 10분 동안 이상 없이 동작하면 지수 백오프 지연 시간을 초기화
- .spec.restartPolicy 재시작 정책
	1. Always : Default 값, 종료/실패 시 항상 재시작
	2. Onfailure : 실패 시 재시작(정상 종료 시 재시작하지 않음)
	3. Never: 재시작 하지 않음

* 예시 
```
---
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod
spec:
  restartPolicy: Always  => spec에서 재시작 정책을 설정할 수 있음
  containers:
  - image: ghcr.io/c1t1d0s7/go-myweb
    name: testapp
    ports:
    - containerPort: 8080
      protocol: TCP
...
```

## 컨테이너 프로브
- 컨테이너 프로브는 kubelet이 컨테이너를 주기적으로 진단하는 프로브 핸들러를 호출 
- 프로브의 핸들러는 세 가지 메커니즘을 가지고 컨테이너의 상태를 진단
- 프로브 핸들러는 3가지 상태 중 하나를 가짐
	1. Success: 진단 통과
	2. Failure: 진단에 실패
	3. Unknown: 진단 자체가 실패

## 프로브 종류
1. livenessProbe (활성 프로브)
	- 컨테이너가 동작 중인지 그 자체를 확인
	- 진단에 실패하면 재시작 정책을 적용
	- livenessProbe를 선언하지 않으면, 기본 상태는 Success

2. readinessProbe(준비 상태 프로브)
	- 컨테이너가 요청을 처리할 준비가 되었는지 확인
	- 프로그램이 요청을 받을 준비가 되었는지를 검사(컨테이너가 서비스가 가능한 상태인지를 체크)
	- 진단에 실패하면 엔드포인트(Endpoint) 컨트롤러는 파드의 IP 주소를 엔드포인트에서 제거
	- readinessProbe를 선언하지 않으면, 기본 상태는 Success

3. startupProbe
	- 컨테이너 내의 애플리케이션이 시작되었는지 확인
	- startupProbe가 선언되었을 경우, 진단을 통과하기 전까지 다른 프로브를 활성화하지 않음 

## 프로브 핸들러의 세 가지 컨테이너 진단 매커니즘
- HTTPGetAction
	* 가장 많이 사용하는 probe 방식으로 HTTP GET을 이용한다
	* 지정된 URL로 HTTP GET 요청을 보내서 리턴값 확인(200 ~ 300 정상)
- TCPSocketAction
	* 특정 TCP 포트 연결
	* 지정된 tcp 포트로 연결할 수 있는지 없는지 확인
	* 지정된 포트에 TCP 연결을 시도하여, 연결이 성공하면, 컨테이너가 정상인것으로 판단
- ExecAction
	* 컨테이너 내의 지정된 바이너리를 실행
	* 명령의 종료 코드가 0인지 확인

## Health Check Practice
1. livenessProbe Normal
- HTTP GET 프로브를 사용하였으며, 경로는 / 포트는 8080이다. 즉, 결론적으로 200 코드 응답이 돌아올 것으로 예상
```
---
apiVersion: v1
kind: Pod
metadata:
  name : testapp-pod-liveness
spec:
  containers:
    - name: testapp
      image: ghcr.io/c1t1d0s7/go-myweb
      ports:
      - containerPort: 8080
      # HTTP GET 프로브를 사용, 경로는 /, 포트는 8080
      livenessProbe:
        httpGet:		
          path: /health
          port: 8080
...
```
```
kubectl create -f testapp-pod-liveness.yaml
```
```
kubectl get pods --watch
```
```
# 정상 작동
$ kubectl get pods --watch
testapp-pod-liveness   0/1     Pending   0          0s
testapp-pod-liveness   0/1     Pending   0          0s
testapp-pod-liveness   0/1     ContainerCreating   0          0s
testapp-pod-liveness   1/1     Running             0          4s
```
```
# 정상 작동 확인
$ watch -n1 kubectl get pods

Every 1.0s: kubectl get pods                                          kube-master1: Wed Jun 22 09:46:02 2022
NAME                   READY   STATUS    RESTARTS   AGE
testapp-pod-liveness   1/1     Running   0          8m34s
```


2. livenessProbe 404
- RESTARTS(재시작) 필드가 0에서 양수로 변경된 것을 확인가능
- 라이브니스 프로브에 의해 컨테이너가 건강한 상태가 아님을 감지하고 재시작을 시도
- CrashLoopBackOff 상태인 것을 확인할 수 있으며, 이는 지수 백오프 지연을 나타냄
```
---
apiVersion: v1
kind: Pod
metadata:
  name : testapp-pod-liveness-404
spec:
  containers:
    - name: testapp
      image: ghcr.io/c1t1d0s7/go-myweb
      ports:
      - containerPort: 8080
      livenessProbe:
        httpGet:
          path: /health?code=404
          port: 8080
...
```
```
# 작동 확인
$ kubectl get pods --watch

testapp-pod-liveness       0/1     Pending   0          0s
testapp-pod-liveness       0/1     Pending   0          0s
testapp-pod-liveness       0/1     ContainerCreating   0          0s
testapp-pod-liveness       1/1     Running             0          4s
testapp-pod-liveness-404   0/1     Pending             0          0s
testapp-pod-liveness-404   0/1     Pending             0          0s
testapp-pod-liveness-404   0/1     ContainerCreating   0          0s
testapp-pod-liveness-404   1/1     Running             0          4s
testapp-pod-liveness-404   1/1     Running             1          27s
testapp-pod-liveness-404   1/1     Running             2          56s
testapp-pod-liveness-404   1/1     Running             3          86s
testapp-pod-liveness-404   1/1     Running             4          117s
testapp-pod-liveness-404   0/1     CrashLoopBackOff    4          2m25s
```
```
# 작동 확인
$ watch -n1 kubectl get pods

Every 1.0s: kubectl get pods                                          kube-master1: Wed Jun 22 09:54:33 2022
NAME                       READY   STATUS             RESTARTS   AGE
testapp-pod-liveness       1/1     Running            0          17m
testapp-pod-liveness-404   0/1     CrashLoopBackOff   5          3m47s
```

3. startupProbe 404
- 파드의 상태가 Running 상태이지만 준비(READY 0/1)가 되지 않고 있음
- 스타트업 프로브에 실패한 것으로 확인되어 라이브니스 프로브는 활성화되지 않음
```
---
apiVersion: v1
kind: Pod
metadata:
  name: testapp-pod-startup-404
spec:
  containers:
  - name: testapp
    image: ghcr.io/c1t1d0s7/go-myweb
    ports:
    - containerPort: 8080
    startupProbe:
      httpGet:
        path: /health?code=404
        port: 8080
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
...
```
```
$ kubectl get pods --watch

NAME                      READY   STATUS    RESTARTS   AGE
testapp-pod-startup-404   0/1     Running   0          58s
testapp-pod-startup-404   0/1     Running   1          74s

```
```
$ watch -n1 kubectl get pods

Every 1.0s: kubectl get pods                                          kube-master1: Thu Jun 23 02:50:55 2022
NAMEy 1.0s: kubectl get poREADY   STATUS    RESTARTS   AGE
testapp-pod-startup-404   0/1     Running   2          2m44s
```

## containers의 Liveness 필드의 의미
- livenessProbe의 경우 초기 지연 시간(initialDelaySeconds)을 설정하는 것이 중요한데, 파드(컨테이너)가 실행되고 난 이후 애플리케이션에 따라 애플리케이션이 정상 동작하는 데 시간이 걸릴 수 있기 때문이다
- 간혹 초기 지연 시간을 설정하지 않아 계속적인 재시작을 초래할 수 있음
1. delay
  	- 프로브를 초기화하기 전 지연
  	- 기본값: 0초
  	- .spec.containers.*Probe.initialDelaySeconds
2. timeout
	- 프로브 타임아웃
  	- 기본값 1초
  	- .spec.containers.*Probe.timeoutSeconds
3. period
  	- 프로브 주기
  	- 기본값 10초
  	- .spec.containers.*Probe.periodSeconds
4. success
  	- 진단 성공 임계값
  	- 기본값 1
  	- .spec.containers.*Probe.successThreshold
5. failure
  	- 진단 실패 임계값
  	- 기본값 3 (3번 연속 진단 실패 시 최종 실패로 진단)
  	- .spec.containers.*Probe.failureThreshold


# 컨트롤러 
- 쿠버네티스에서 어플리케이션을 구동하기 위해서는 파드를 생성해야 하다.
- 쿠버네티스 컨트롤러는 파드를 올바르게 동작하게 하기위한 컨트롤러이고 특정 상태는 컨트롤러의 동작 방식과 정의 상태에 따라 다르다 
- 실제 쿠버네티스를 통해서 서비스를 구축할 때 개별 파드 오브젝트를 직접 컨트롤하는 방식은 사실상 사용되지 않음
- K8s는 컨테이너 가상화의 특징을 활용해 유연하고 확정성있는 서비스를 구성할 수 있도록 컨트롤러 오브젝트를 제공한다.
- 컨트롤러 오브젝트는 '파드의 상태 유지'의 방식 사용

# replicaSet 컨트롤러
- replicaSet 컨트롤러는 파드가 특정 개수만큼 복제되고 동작하는것을 보장한다.
- 노드의 문제가 발생했거나 파드가 문제가 생겨 원하는 수의 파드가 동작하지 않는다면 자동으로 스케줄러에 의해 새로운 다시 새로운 파드를 생성해 원하는 수의 파드를 유지한다

* 원하는 수의 복제본 보다 더 많은 복제본이 생길수 있는 경우
	- 수동으로 동일한 형식을 파드를 생성
	- 기존의 파드의 유형을 변경
	- 원하는 수의 파드 복제본 수를 줄인 경우

즉, 실행 중인 파드의 목록을 주기적으로 모니터링하고, 원하는 수의 파드 수와 항상 일치

## replicaSet 구성요소
- 파드를 지정하는 레이블 'selector'
- 새로운 파드의 복제본을 만들기 위한 파드 'template'
- 복제본의 수(기본값 : 1)  

## replicaSet이 제공하는 기능
- 원하는 복제본 수의 파드가 없는 경우 파드 템플릿을 이용하여 파드를 생성
- 노드에 장애가 발생하면 장애가 발생한 노드에서 실행 중이던 파드를 다른 노드에 복제본을 생성
- 수동이나 자동으로 파드를 수평적으로 스케일링할 수 있음

## API 그룹과 버전 정보 확인
```
$ kubectl api-resources
$ kubectl api-versions 
```

## replicaSet 사용하기
```
vi testapp-replica.yml
```
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: testapp-replica
spec:
  # 복제본의 수 정의
  replicas: 3
  # 셀렉터 정의
  selector:
  	# 레이블이 같은 파드를 찾아서 개수 유지
    matchLabels: 
      app: testapp-replica
  # 개수 유지가 안된다면 파드를 그 수만큼 생성
  template: 
    metadata:
      # 새로 생설될 파드의 레이블
      labels:
        app: testapp-replica
    spec:
      containers:
      - name: testapp
        image: c1t1d0s7/myweb
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
```
![화면 캡처 2022-06-22 230913](https://user-images.githubusercontent.com/57117748/175050369-addde139-27d1-4b4c-b2d0-0eb4c06e019f.png)
![화면 캡처 2022-06-22 161141](https://user-images.githubusercontent.com/57117748/175050634-452d28cc-c998-48fa-9064-f1c805a1fe13.png)

* 파드 확인
```
$ kubectl describe replicaset/apps [podname]
$ kubectl get pods --show-labels
```


* 레이블 변경
```
$ kubectl label pods [podname] [Key]=[Value]

$ kubectl label pods [podname] [Key]=[Value] --overwrite
```

* scale out
```
$ kubectl scale replicaset [controlername] --replicas=[int:수정값]
``` 

* 오브젝트를 이용한 scale out
```
$ kubectl edit replicasets.apps [controlername]
```
![화면 캡처 2022-06-22 232248](https://user-images.githubusercontent.com/57117748/175053278-5f5120f8-34b7-4dc8-8c3e-23f1c7fec136.png)

```
.
.
.
spec:
  replicas: 5  	// 해당값 변경
  selector:
.
.
.
.
```
![화면 캡처 2022-06-22 161348](https://user-images.githubusercontent.com/57117748/175054304-1fb2897b-7610-4d3a-8b7d-8e59d7596c36.png)

## replicaSet 과 replication Container 비교
- 쿠버네티스 초기 파드의 복제본을 생성하고, 파드의 개수를 유지하는 기능을 제공하는 컨트롤러로 '레플리케이션 컨트롤러'를 사용
- 현재는 레플리케이션 컨트롤러 대신에 레플리카셋 컨트롤러를 사용하기 시작
- 레플라카셋의 기본 목적은 레플리케이션 컨트롤러와 같지만, 레플리카셋은 레이블 셀렉터의 "집합성 기준" 레이블 셀렉터를 지원
```
1. 일치성 기준 : 레이블 셀렉터는 "키=값" 모두 일치해야 (replication Container, replicaSet)
.spec.selector.matchLabels: 일치성 기준 레이블 셀렉터 지정

2. 집합성 기준 : 레이블 셀렉터는 "키=값" 모두 일치하거나, "키" 만 일치하는 것도 지원 (replicaSet)
.spec.selector.matchExpressions: 집합성 기준 레이블 셀렉터 지정
```

## 레이블 매칭 확장 연산자(matchExpressions)
1. In : 레이블의 키가 존재하며, 값이 대상 중 하나와 일치함

2. NotIn : 레이블의 키가 존재하며, 값이 대상 모두와 불일치

3. Exists : 레이블의 키가 존재함 (값은 무관)

4. DoesNotExist : 레이블의 키가 존재하지 않아야 함

Exists와 DoesNotExist는 레이블의 키값만 매칭 즉, values 항목을 설정하지 않는다.


## matchExpressions 사용
* testapp-pod1 을 생성하는데 labels 에 app: testapp-rs-exp, env: dev 입력 
![화면 캡처 2022-06-22 162202](https://user-images.githubusercontent.com/57117748/175060031-9dfd5dc0-22da-4318-ab57-1435d8ff2d2e.png)

* 같은 방식으로 testapp-pod2 생성
![화면 캡처 2022-06-22 162135](https://user-images.githubusercontent.com/57117748/175060142-5b245f43-753c-4dab-bcef-ae778cde2690.png)


```
vi testapp-rs-exp.yml
```
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: testapp-rs-exp
spec:
  replicas: 4
  selector:
    matchExpressions:
    - key: app
      operator: In  		// label의 Key 해당 키가 존재 Value가 하나 이상 일치 
      values:	
      - testapp-rs
    - key: env
      operator: Exists 		// label의 Key 해당 키가 존재
  template:
    metadata:
      labels:
        app: testapp-rs
        env: op
    spec:
      containers:
      - name: testapp
        image: c1t1d0s7/myweb
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
```
```
$ kubectl get pods --watch
```
```
$ watch -n1 kubectl get pods

Every 1.0s: kubectl get pods                                          kube-master1: Wed Jun 22 14:25:28 2022
NAME                   READY   STATUS    RESTARTS   AGE
testapp-pod1 		   1/1     Running   0          10m22s
testapp-pod2   		   1/1     Running   0          12m22s
testapp-rs-exp-6nmwn   	   1/1     Running   0          3m22s
testapp-rs-exp-hlq4r       1/1     Running   0          3m22s
```
![화면 캡처 2022-06-22 163818](https://user-images.githubusercontent.com/57117748/175059332-ae8574fc-6806-4168-adb7-5b06bdfb9b66.png)

## 키워드 

- 파드 생명주기 : 파드는 정의된 순서대로 생명주기(Pending -> Running -> Succeeded) 파드는 영구적이 아닌 임시로 사용하기 위한 워크로드 리소스로 파드가 생성되면 파드를 식별하기 위한 고유의 UID가 할당되고, 종료될 때까지 특정 노드에서만 실행

- 파드의 단계 : Pending, Running, Succeeded, Failed, Unknown 

- Pending : 클러스터에서 승인을 받은 실행되지 않는 상태 파드가 스케줄링 되기전 작업 등 을 진행하는 상태

- 컨테이너 상태 : Waiting, Running, Terminated 의 상태를 가진다. 스케줄러가 파드를 노드에 할당하면, kubelet은 컨테이너 런타임을 이용하여 컨테이너를 생성, 주기적인 모니터링 진행

- 프로브 : Kubelet이 컨테이너를 주기적으로 진단하는 프로브 핸들러를 호출하는 상태 3가지의 매커니즘으로 상태 판단 HTTPGetAction, TCPSocketAction, ExecAction 

- 프로브의 종류 : 컨테이너가 자체가 동작 중인지 확인 확인하는 livenessProbe, 컨테이너가 요청을 받아 처리할 준비가 되었는지 확인하는 readinessProbe, 컨테이션 내의 애플리케이션이 시작 되었는지 확인하는 startupProbe

- 컨트롤러 : 파드를 생성할 때 가상화의 특징을 활용하여 유연하고 확정성 있는 서비스를 구성할 수 있는 컨트롤러 오브젝트를 뜻함 컨트롤러는 지정된 상태를 유지를 지향

- replicasSet : 파드가 특정 개수만큼 복제되고 동작하는 것을 보장하는 컨트롤러. 실행 중인 파드를 주기적으로 모니터링해, 유지하려는 파드 수와 일치시킨다.

- Label Selector : 레이블을 통해 파드를 식별할수 있게 한다, replicasSet에서 파드를 유지 하려 할때 레이블 셀렉터로 식별하여 해당 개수 만큼 유지 시키려 한다. 

- Pods Templete : 레이블 셀렉터를 통해 식별한 값이 replicasSet에서 유지하려는 값보다 적을때 Pod 를 생성, 복제 하기위한 항목
