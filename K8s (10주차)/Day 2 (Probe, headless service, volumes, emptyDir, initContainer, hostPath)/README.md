# Probe
1. LivenessProbe
	- 불완전한 파드를 재시작하기 위해 
	- 컨테이너가 켜져있는 자체를 확인
	- liveness Probe

2. ReadinessProbe
	 - 컨테이너가 요청을 처리할 준비가 되었는지 확인 즉, 서비스를 실행할 준비가 되었는지 확인
	 - 진단에 실패하면 엔드포인트(Endpoint) 컨트롤러는 파드의 IP 주소를 엔드포인트에서 제거

3. StartupProbe 
	- 맨처음 application 시작할 때 확인


## Probe 확인방법
1. HTTP GET 
	- 특정 경로에 HTTP Get 요청
	- HTTP 응답코드 200 - 399 확인
2. TCP
	- 특정 TCP port 연결 시도(TCP syn, syn+ack, ack : 3 ways hands shaking)
3. Exec 
	- 컨테이너 내부에 바이너리(명령)을 실행하고 종료 코드로 확인

## readiness Probe 사용
1. svc-readiness manifest 작성
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-readiness
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app : myapp-rs-readiness
...
```

2. rs-readiness manifast 작성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-readiness
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-rs-readiness
  template:
    metadata:
      labels:
        app: myapp-rs-readiness
    spec:
      containers: 
      - name: myapp
        image: ghcr.io/c1t1d0s7/go-myweb:alpine
        readinessProbe:
          exec:
            command:
            - ls
            - /var/ready
        ports:
        - containerPort: 8080
...
```

3. 실행
```
$ kubectl create -f myapp-svc-readiness.yml -f myapp-rs-readiness.yaml
```

4. 확인
- readiness로 확인한 결과 아직 서비스 준비가 되지 않아 READY 0 인 상태이고 그로인해 endpoint 가 할당 되지 않음
```
$ kubectl get all,endpoints
NAME                           READY   STATUS    RESTARTS   AGE
pod/myapp-rs-readiness-jw2gm   0/1     Running   0          83s
pod/myapp-rs-readiness-m8k4g   0/1     Running   0          83s
pod/myapp-rs-readiness-stckn   0/1     Running   0          83s

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes            ClusterIP   10.233.0.1      <none>        443/TCP   89m
service/myapp-svc-readiness   ClusterIP   10.233.35.121   <none>        80/TCP    83s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-readiness   3         3         0       83s

NAME                            ENDPOINTS            AGE
endpoints/kubernetes            192.168.56.11:6443   90m
endpoints/myapp-svc-readiness                        2m58s 
```

- Unhealthy 부분을 보면 exec probe 확인을 위해에서 명령한 ls를 실행 할 수 없는데 체크하려는 파일이 존재하지 않기 때문이다.
```
$ kubectl describe pod myapp-rs-readiness-jw2gm | tail -5

  Normal   Scheduled  6m25s                 default-scheduler  Successfully assigned default/myapp-rs-readiness-jw2gm to kube-node1
  Normal   Pulled     6m24s                 kubelet            Container image "ghcr.io/c1t1d0s7/go-myweb:alpine" already present on machine
  Normal   Created    6m23s                 kubelet            Created container myapp
  Normal   Started    6m23s                 kubelet            Started container myapp
  Warning  Unhealthy  76s (x31 over 6m16s)  kubelet            Readiness probe failed: ls: /var/ready: No such file or directory
```

4. 수정 및 확인
```
# 파드에게 exec 명령이 실행 되도록 파일 생성
$ kubectl exec myapp-rs-readiness-jw2gm -- mkdir /var/ready


# 해당 파드 READY 상태 및 endpoints 확인 가능
$ kubectl get all,endpoints
NAME                           READY   STATUS    RESTARTS   AGE
pod/myapp-rs-readiness-jw2gm   1/1     Running   0          10m
pod/myapp-rs-readiness-m8k4g   0/1     Running   0          10m
pod/myapp-rs-readiness-stckn   0/1     Running   0          10m

NAME                          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/kubernetes            ClusterIP   10.233.0.1      <none>        443/TCP   98m
service/myapp-svc-readiness   ClusterIP   10.233.35.121   <none>        80/TCP    10m

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-readiness   3         3         1       10m

NAME                            ENDPOINTS             AGE
endpoints/kubernetes            192.168.56.11:6443    98m
endpoints/myapp-svc-readiness   10.233.101.116:8080   10m
```

# Headless 서비스
- 기본적으로 파드 그룹을 서비스의 엔드포인트에 등록하고, LB를 이용한다
- 하지만 늘 LB가 필요로 하는것은 아니다.
- 로드 밸런서가 필요하지 않을때 즉, 개별의 파드에 접근이 필요한 경우 'Headless' 서비스를 사용
- 쿠버네티스 서비스 생성 시 .spec.clusterIP 필드 값을 None으로 설정하면 클러스터 IP가 없는 서비스
- 헤드리스 서비스에 셀렉터(.spec.selector) 필드를 설정하면 쿠버네티스 API로 확인할 수 있는 엔드포인트(endpoint)가 만들어집니다. 서비스와 연결된 파드를 직접 가리키는 DNA A 만들어진다
- 즉 ClusterIP 가 아닌 Pod 각각의 IP를 사용해 엔드포인트와 연결한다.
```
VM   haproxy(LB)-------web--------WAS-----------DB1 (W, R) 쓰기가능 master
							    				DB2 (RO)읽기전용 salve
```

## Headless 사용하기
1. svc-headless manifest 작성
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-headless
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: myapp-rs-headless
...
```

2. rs-headless manifest 작성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-headless
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-rs-headless
  template:
    metadata:
      labels:
        app: myapp-rs-headless
    spec:
      containers:
      - name: myapp
        image: ghcr.io/c1t1d0s7/go-myweb:alpine
        ports:
        - containerPort: 8080
...
```

3. 생성
```
$ kubectl create -f myapp-svc-headless.yaml -f myapp-rs-headless.yaml
```

4. 확인
- ClusterIP는 None이지만 endponts에 각각 파드들이 연결되어 있다.
```
$ kubectl get all,endpoints
NAME                          READY   STATUS    RESTARTS   AGE
pod/myapp-rs-headless-2ww6x   1/1     Running   0          12s
pod/myapp-rs-headless-sk27x   1/1     Running   0          12s
pod/myapp-rs-headless-tq2sh   1/1     Running   0          12s

NAME                         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes           ClusterIP   10.233.0.1   <none>        443/TCP   4h30m
service/myapp-svc-headless   ClusterIP   None         <none>        80/TCP    53s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-headless   3         3         3       12s

NAME                           ENDPOINTS                                                    AGE
endpoints/kubernetes           192.168.56.11:6443                                           4h30m
endpoints/myapp-svc-headless   10.233.101.117:8080,10.233.103.112:8080,10.233.76.145:8080   53s
```

5. 추가확인
```
$kubectl run nettool -it --image=c1t1d0s7/network-multitool  --rm=true bash

# nslookup myapp-svc-headless
Server:         169.254.25.10
Address:        169.254.25.10#53

Name:   myapp-svc-headless.default.svc.cluster.local
Address: 10.233.76.145
Name:   myapp-svc-headless.default.svc.cluster.local
Address: 10.233.101.117
Name:   myapp-svc-headless.default.svc.cluster.local
Address: 10.233.103.112

# host myapp-svc-headless
myapp-svc-headless.default.svc.cluster.local has address 10.233.103.112
myapp-svc-headless.default.svc.cluster.local has address 10.233.101.117
myapp-svc-headless.default.svc.cluster.local has address 10.233.76.145

# dig myapp-svc-headless.default.svc.cluster.local
; <<>> DiG 9.14.8 <<>> myapp-svc-headless.default.svc.cluster.local
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7044
;; flags: qr aa rd; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 1
;; WARNING: recursion requested but not available

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
; COOKIE: c7ebb87921cebbfd (echoed)
;; QUESTION SECTION:
;myapp-svc-headless.default.svc.cluster.local. IN A

;; ANSWER SECTION:
myapp-svc-headless.default.svc.cluster.local. 5 IN A 10.233.103.112
myapp-svc-headless.default.svc.cluster.local. 5 IN A 10.233.101.117
myapp-svc-headless.default.svc.cluster.local. 5 IN A 10.233.76.145

;; Query time: 2 msec
;; SERVER: 169.254.25.10#53(169.254.25.10)
;; WHEN: Tue Jun 28 13:10:16 UTC 2022
;; MSG SIZE  rcvd: 265
```

6. 파드 접속 확인
```
$ curl 10.233.103.112:8080
Hello World!
myapp-rs-headless-2ww6x

$ curl 10.233.101.117:8080
Hello World!
myapp-rs-headless-tq2sh

$ curl 10.233.76.145:8080
Hello World!
myapp-rs-headless-sk27x
```

# Volume
- 파드(컨테이너)의 데이터는 별도로 저장/관리 하지 않는다.
- 기본적으로 컨테이너는 프로세스를 완료하면 컨테이너를 종료하는 일회적인 프로세스로 간주되기 때문에 데이터를 저장할 공간이 필요하다.
- 컨테이너 삭제 후에도 보관되는 저장소 'Volume'

* 컨테이너의 장/단점
- 장 : 컨테이너 문제 발생 시 자유롭게 새로운 재생성 가능
- 단 : 저장한 데이터 유실

- CNI : Container Network Interface	
- CSI : Container Storage Interface 
- CRI : Container Runtime Interface

* 쿠버네티스에서 지원하는 볼륨의 종류이다.
1. emptyDir (node 로컬 사용)
  - 임시로 데이터를 저장하는 빈 볼륨, , RAMDisk가능, 동일한 파드내의 컨테이너 간에 데이터를 공유, 경로 지정 불가

2. gitRepo  
  - initContainer, 빈 컨테이너에 gitrepo의 데이터를 clone한다, git에 업로드후 파드로 데이터 클론 할 수 있다

3. hostPath (node 로컬 사용)
  - 노드 로컬내에서 데이터를 저장하는 볼륨 그러나 emptyDir와 다르게 경로 지정이 가능하다.

4. 네트워크 스토리지 볼륨
  - 공유의 개념을 가지고 있음. 다른 호스트의 pod를 가능함. 하나의 pod에 동시에 접근가능함(동시쓰기는 안됨)
  - cephfs,iscsi, nfs, rbd...

5. 클라우드 스토리지 볼륨
   - 다른 pod와 공유가능
   - awsElasticBlockStore, azureDisk, azureFile, gcePersistentDisk, awsEBS  

6. 정적/동정 프로비저닝 볼륨
  - persistentVolumeClaim

7. 특수 유형 볼륨
  - configMap, secret


## emptyDir
- 하나의 '노드 내'에서 여러 Pod가 emptyDir를 공유
- 볼륨이지만 데이터를 영구저장하지 않음(파드가 실행중에만 실행)
- 파드의 컨테이너들이 서로 데이터 공유를 위해 사용
- 디스크 / 메모리를 이용 (고속 엔진 스토리지)
- 임시로 사용하기 위한 용도로 이용

1. svc manifest 작성
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-fortune
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp-rs-fortune
...
```

2. rs manifest 작성 이떄 2개의 컨테이너를 하나의 파드에서 생성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-fortune
spec:
  replicas: 1
  selector:
    matchLabels:
      app: myapp-rs-fortune
  template:
    metadata:
      labels:
        app: myapp-rs-fortune
    spec:
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: web-fortune  # mount with web-fortune volume
          mountPath: /usr/share/nginx/html  # mount path of web-server
          readOnly: true
        ports:
        - containerPort: 80
      - name: html-generator
        image: ghcr.io/c1t1d0s7/fortune
        volumeMounts:
        - name: web-fortune
          mountPath: /var/htdocs
      volumes:
      - name: web-fortune  #make emptyDir
        emptyDir: {}  # notting inside
...
```


3. 생성 
```
$ kubectl create -f myapp-svc-fortune.yaml -f myapp-rs-fortune.yml
```

4. 확인
```
$ kubectl get all,endpoints
NAME                         READY   STATUS    RESTARTS   AGE
pod/myapp-rs-fortune-xbrcm   2/2     Running   0          38s

NAME                        TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
service/kubernetes          ClusterIP      10.233.0.1     <none>           443/TCP        5h56m
service/myapp-svc-fortune   LoadBalancer   10.233.2.121   192.168.56.200   80:32201/TCP   38s

NAME                               DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-fortune   1         1         1       38s

NAME                          ENDPOINTS            AGE
endpoints/kubernetes          192.168.56.11:6443   5h56m
endpoints/myapp-svc-fortune   10.233.76.149:80     38s
```

5. 스토리지 공유 확인
![화면 캡처 2022-06-28 161758](https://user-images.githubusercontent.com/57117748/176207084-9d011e36-767c-47c7-88d3-14b78b0ba9bf.png)

## infraContainer
- 하나의 파드에 여러 컨테이너가 작동중이더라도 하나의 Ip를 받아 App을 실행하는데 혼용되지 않도록 한다.
- 항상 작동

## initContainer
- gitRepo 사용이 중지 되면서 gitrepo를 대체하기 위한 방법 초기화 컨테이너(initContainer)
- 파드 생성 시, 1번 실행 후 종료
- 파드에서 실직적인 목적(어플리케이션)을 위한 초기 구성에 필요한 작업을 진행
What is gitRepo?
- 기본적으로 EmptyDir와 같은 기능이지만 git 저장소를 동기화 할 수 있다는 차이가 있었다. 
- git 레포지토리에 있는 데이터를 복제해 파드에게 제공하는 형태


1. initContainer manifest 작성
```
---
apiVersion: v1
kind: Pod
metadata:
  name: init-pod
spec:
  initContainers:
    - name: gitpull
      image: alpine/git
      args:
        - clone # birng codes form git
        - -b
        - v2.18.1
        - https://github.com/kubernetes-sigs/kubespray.git
        - /repo  # match mountPath of volumeMounts
      volumeMounts:
        - name: gitrepo
          mountPath: /repo
  containers:
    - name: gituse
      image: busybox
      args:  # keep running tail -f /dev/null
        - tail
        - -f
        - /dev/null
      volumeMounts:
        - name: gitrepo
          mountPath: /kube # copy /kube to mountPath
  volumes:
    - name: gitrepo
      emptyDir: {}
...
```

2. 실행 및 확인
```
$ kubectl create -f init-pod.yml
pod/init-pod created

$ kubectl get pod
NAME                     READY   STATUS     RESTARTS   AGE
init-pod                 0/1     Init:0/1   0          9s

$ kubectl get pod
NAME                     READY   STATUS    RESTARTS   AGE
init-pod                 1/1     Running   0          19s
```

3. 과정 확인
```
$ kubectl get pod --watch
init-pod                 0/1     Pending     	  0          0s
init-pod                 0/1     Pending    	  0          0s
init-pod                 0/1     Init:0/1    	  0          0s
init-pod                 0/1     Init:0/1    	  0          5s
init-pod                 0/1     PodInitializing  0          12s
init-pod                 1/1     Running          0          16s
```

4. pod 접속을 통한 git Clone 확인
```
$ kubectl exec -it init-pod -- sh
/ # ls
bin   dev   etc   home  kube  proc  root  sys   tmp   usr   var

/ # cd kube/
/kube # ls
CNAME                      _config.yml                legacy_groups.yml          reset.yml
CONTRIBUTING.md            ansible.cfg                library                    roles
Dockerfile                 ansible_version.yml        logo                       scale.yml
LICENSE                    cluster.yml                mitogen.yml                scripts
Makefile                   code-of-conduct.md         recover-control-plane.yml  setup.cfg
OWNERS                     contrib                    remove-node.yml            setup.py
OWNERS_ALIASES             docs                       requirements-2.10.txt      test-infra
README.md                  extra_playbooks            requirements-2.11.txt      tests
RELEASE.md                 facts.yml                  requirements-2.9.txt       upgrade-cluster.yml
SECURITY_CONTACTS          index.html                 requirements-2.9.yml
Vagrantfile                inventory                  requirements.txt
```

* 과정을 살펴보면
```
initContainer 생성 -> volume 연결되어 git clone 코드를 저장 -> 종료 
-> busybox 실행 -> Mount

* inticontianer는 상당히 많이 사용하는 기법 
```

# hostPath
- 쿠버네티스 클러스터 노드(호스트)의 파일 시스템을 제공 즉, 노드에 데이터 저장. **노드 내** 파일공유 지원하며 **경로 지정**이 가능한 볼륨
- 노드에 데이터를 저장하기 때문에 파드가 재시작해도 가능 (같은 노드에서 재시작 할 때에만)
![화면 캡처 2022-06-28 143900](https://user-images.githubusercontent.com/57117748/176227492-04e1dbd1-f3ec-44a1-9864-1db57fd5a5ef.png)

1. svc manifest 작성
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-hp
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp-rs-hp
...
```

2. hostPath manifest 작성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-hp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp-rs-hp
  template:
    metadata:
      labels:
        app: myapp-rs-hp
    spec:
      # nodeName: kube-node1 경로지정 x
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: web-content
        hostPath:
          type: Directory
          path: /web_contents
...
```

3. 생성 
```
$ kubectl create -f myapp-rs-hp.yaml -f myapp-svc-hp.yaml
```

4. 확인
- node 2,3에 volume이 없기 때문에 계속해서 ContainerCreating 상태
```
$ kubectl get all -o wide
NAME                    READY   STATUS              RESTARTS   AGE   IP       NODE         NOMINATED NODE   READINESS GATES
pod/myapp-rs-hp-4nrh4   0/1     ContainerCreating   0          9s    <none>   kube-node2   <none>           <none>
pod/myapp-rs-hp-mzr6w   0/1     ContainerCreating   0          9s    <none>   kube-node3   <none>           <none>

NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE   SELECTOR
service/kubernetes     ClusterIP      10.233.0.1    <none>           443/TCP        13m   <none>
service/myapp-svc-hp   LoadBalancer   10.233.6.67   192.168.56.200   80:31407/TCP   9s    app=myapp-rs-hp

NAME                          DESIRED   CURRENT   READY   AGE   CONTAINERS   IMAGES         SELECTOR
replicaset.apps/myapp-rs-hp   2         2         0       9s    web-server   nginx:alpine   app=myapp-rs-hp
```

5. 자세한 정보 확인
- Unable to attach or mount volumes 즉, 볼륨에 연결 할수 없다
```
$ kubectl describe pod myapp-rs-hp-4nrh4 | tail -5
  Type     Reason       Age                   From               Message
  ----     ------       ----                  ----               -------
  Normal   Scheduled    5m13s                 default-scheduler  Successfully assigned default/myapp-rs-hp-4nrh4 to kube-node2
  Warning  FailedMount  64s (x10 over 5m14s)  kubelet            MountVolume.SetUp failed for volume "web-content" : hostPath type check failed: /web_contents is not a directory
  Warning  FailedMount  55s (x2 over 3m11s)   kubelet            Unable to attach or mount volumes: unmounted volumes=[web-content], unattached volumes=[web-content default-token-n7fm9]: timed out waiting for the condition
```

6. node1 에서 볼륨과 연결 될 /web_contents 디렉토리 생성
```
vagrant@kube-node1:/$ sudo mkdir /web_contents

node1 $ echo "hello hosPath Volumes" | sudo tee /web_contents/index.html
```

7. 기존 rs 삭제
```
$ kubectl delete replicasets.apps myapp-rs-hp
```

8. hostPath가 적용된(nodeName) hostPath manifest 작성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-hp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp-rs-hp
  template:
    metadata:
      labels:
        app: myapp-rs-hp
    spec:
      nodeName: kube-node1
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: web-content
        hostPath:
          type: Directory
          path: /web_contents
...
``` 

9. 실행 및 확인(Running 상태)
```
$ kubectl create -f myapp-rs-hp.yaml

$ kubectl get all
NAME                    READY   STATUS    RESTARTS   AGE
pod/myapp-rs-hp-r9kj8   1/1     Running   0          12s
pod/myapp-rs-hp-zxc82   1/1     Running   0          12s

NAME                   TYPE           CLUSTER-IP    EXTERNAL-IP      PORT(S)        AGE
service/kubernetes     ClusterIP      10.233.0.1    <none>           443/TCP        27m
service/myapp-svc-hp   LoadBalancer   10.233.6.67   192.168.56.200   80:31407/TCP   14m

NAME                          DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-hp   2         2         2       12s 
```

10. 상세 정보 확인
```
$ kubectl describe pod myapp-rs-hp-r9kj8  | tail -5
  Type    Reason   Age    From     Message
  ----    ------   ----   ----     -------
  Normal  Pulled   3m33s  kubelet  Container image "nginx:alpine" already present on machine
  Normal  Created  3m32s  kubelet  Created container web-server
  Normal  Started  3m32s  kubelet  Started container web-server
```

11. 파드에서 볼륨 공유 확인
```
$ kubectl exec myapp-rs-hp-r9kj8 -- cat /usr/share/nginx/html/index.html
hello hosPath Volumes

$ kubectl exec myapp-rs-hp-zxc82 -- cat /usr/share/nginx/html/index.html
hello hosPath Volumes
```

## 키워드
- Probe Health Check : probe를 이용해 컨테이너의 상태를 확인하는 방법으로 livenessProbe, readinessProbe, startupProbe 가 존재한다. 순서대로 컨테이너 활성화 확인, 서비스 준비 확인, 애플리케이션 시작 확인이다.

- livenessProbe : 파드가 비활성화 되었을시 다시 시작 즉, 컨테이너가 시작되지 않았을시에 재실행 시키는 프로브

- readinessProbe : 파드가 활성화 되어 서비스를 준비할 시작이 되었는지 확인하는 프로브

- startupProbe : 처음 컨테이너를 실행시 사용하는 프로브로 애플리케이션 시작을 확인

- HTTPGet : 지정된 URL로 HTTPGet 신호를 보내 확인하는 방법

- TPC Socket : 해당 컨테이너의 TCP 포트가 활성화 되어 있는지 확인

- Exec : 컨테이너의 지정된 바이너리(명령)을 실행하여 확인

- Headless Service : ClusterIP가 None으로 지정되어 있는 상태로 ClusterIP를 통해 LB를 사용하는 것이 아니라 직접 원하는 파드의 IP로 접속 할 수 있는 방법

- volume : 파드는 별도로 데이터를 저장/관리 하지 않아 컨테이너가 삭제되면 데이터도 함께 삭제된다. 볼륨은 컨테이너가 삭제된 이후에도 보관되는 장소, 무작위 볼륨 생성 위치

- emptyDir : 비어있는 컨테이너를 임시로(컨테이너 삭제시 없어짐) 데이터를 저장하여 오직 같은 노드에 있는 파드에 데이터를 공유

- initContainer : git의 있는 데이터를 클론하여 빈 컨테이너에 저장하게 하고 마운트 하지만 한번만 실행 가능

- hostPath : 지정한 위치(노드)에 볼륨생성이 가능. 해당 노드의 지정된 디렉토리를 볼륨으로 사용 즉, 노드와 경로를 지정하여 볼륨을 만들 수 있다 단, 노드를 2개를 지정 불가
