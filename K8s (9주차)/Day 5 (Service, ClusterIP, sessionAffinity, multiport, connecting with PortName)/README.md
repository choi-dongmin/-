# Service
* 서비스의 이해
	- 파드는 클러스터 외부의 요청이나 클러스터 내부의 다른 파드의 요청에 응답해야 하고, 파드가 다른 파드에서 제공하는 애플리케이션에 접근하기 위해서는 파드를 찾을 수 있어야 한다.
	- 기존의 시스템은 애플리케이션이 동작하는 시스템의 호스트 이름이나 정적 IP를 할당하여 해당 애플리케이션을 찾을 수 있지만, K8s 환경에서 그렇게 할 수 없다.
	- 그 이유는 파드는 일회성으로 동작하기 위해서 설계되었고 IP 주소가 동적으로 할당되어 미리 예측 할 수 없다
	- 이러한 문제를 해결하기 위해 파드는 단일 고정 IP를 통해서 서비스를 제공하여야 한다.

## Service
- 쿠버네티스 시스템에서 같은 애플리케이션을 실행하는 컨트롤러 파드 그룹에 '단일 네트워크 진입' 리소스

- 서비스 부여된 IP는 해당이 서비스가 종료될 때까지 변경되지 않음

- 클라이언트는 서비스가 제공하는 IP 및 포트를 통해 파드에 접근하게 되고 만약 kube-dns(coredns)가 적용되어 있다면 서비스의 이름에 기반한 고유한 FQDN이 부여

- 서비스는 레이블 셀렉터를 이용해, 서비스의 대상(백엔드) 파드를 설정하는데 서비스에 선택된 파드의 목록은 엔드포인트(Endpoint) 리소스로 관리

- 서비스 리소스는 여러 파드가 연결되어 있을 때 라운드 로빈(Round Robin) 방식의 부하 분산을 제공

## Service의 종류
1. ClusterIP
	- 클러스터 내부용 서비스

2. NodePort
	- NodePort + ClusterIP 
	- 모든 노드(호스트)에 외부 접근용 포트
	- 노드의 포트로 접근하면 서비스에 의해 파드로 리다이렉션
	- 파드를 실행하고 있지 않는 노드에도 포트가 할당되고, 접근 가능

3. LoadBalancer
	- LoadBalancer + NodePort + ClusterIP
	- NodePort의 확장 형태
	- 클러스터 외부의 로드 밸런서를 사용하여 외부에서 접근 가능하게 함
	- 외부 로드 밸런서로 접근하면 서비스를 통해 파드로 리다이렉션
	- 클라우드 공급업체(AWS, GCP 등)나 MetalLB에서 지원
4. ExternalName
	- 외부에서 접근하기 위한 종류 아님
	- 외부의 특정 FQDN에 대한 CNAME 매핑
	- 파드가 CNAME을 이용해 특정 FQDN과 통신하기 위함

# Cluster IP
- 서비스의 가장 기본이 되는 타입으로 Pod의 IP 주소를 EndPoint로 가져서 전달
- Endpoints: 서비스의 레이블 셀렉터에 의해 연결된(포워딩 할) 파드의 IP 목록을 엔드포인트 리소스로 관리
- 클러스터 내부에 만든 네트워크 망
- Default Service Type

* 명령어를 통한 서비스 구성
```
$ kubectl expose <CONTROLLER_TYPE> <CONTROLLER_NAME> [--type=<SVC_TYPE>] --name <SVC_NAME>
```

* yaml 파일을 이용한 방법
```
---
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-clusterip
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: testapp-rs
...
```

* 구성 항목
```
.spec.ports.port: 서비스 리소스의 포트
.spec.ports.targetPort: 파드(대상) 리소스의 포트
.spec.selector: 파드 레이블 셀렉터, 엔드포인트 리소스 생성
```


## Cluster IP 사용

1. replicas를 통해 노드에 파드를 생성한다.

```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: testapp-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: testapp-rs
  template:
    metadata:
      labels:
        app: testapp-rs
    spec:
      containers:
      - name: testapp
        image: c1t1d0s7/myweb
        ports:
        - containerPort: 8080
...
```
```
$ watch -n1 kubectl get pods --show-labels

NAME                           READY   STATUS    RESTARTS   AGE   LABELS
pod/testapp-replicaset-8hxpx   1/1     Running   0          73s   app=testapp-rs
pod/testapp-replicaset-9cxh4   1/1     Running   0          73s   app=testapp-rs
pod/testapp-replicaset-ml8ps   1/1     Running   0          73s   app=testapp-rs
```

2. clusterIP 서비스 메니페스트 생성 및 확인

```
---
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-clusterip
spec:
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: testapp-rs
...
```
```
$ watch -n1 kubectl get services,endpoints,pods --show-labels

NAME                              Every 1.0s: kubectl get services,endpoints,pods --show-labels         kube-master1: Fri Jun 24 09:53:51 2022
NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     LABELS
service/kubernetes              ClusterIP   10.233.0.1      <none>        443/TCP   4h33m   component=apiserver,provider=kubernetes
service/testapp-svc-clusterip   ClusterIP   10.233.50.151   <none>        80/TCP    11m     <none>

NAME                              ENDPOINTS                                                  AGE     LABELS
endpoints/kubernetes              192.168.56.11:6443                                         4h33m   <none>
endpoints/testapp-svc-clusterip   10.233.101.76:8080,10.233.103.68:8080,10.233.76.103:8080   11m     <none>

NAME                           READY   STATUS    RESTARTS   AGE   LABELS
pod/testapp-replicaset-8hxpx   1/1     Running   0          20m   app=testapp-rs
pod/testapp-replicaset-9cxh4   1/1     Running   0          20m   app=testapp-rs
pod/testapp-replicaset-ml8ps   1/1     Running   0          20m   app=testapp-rs

```

3. 내부 네트워크에서 접속 확인
```
vagrant@kube-master1:~/test$ curl 10.233.50.151:80
Message: Hello World!
Hostname: testapp-replicaset-ml8ps
Platform: linux
Uptime: 31131
IP: 10.233.76.103
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 10.233.50.151:80
Message: Hello World!
Hostname: testapp-replicaset-9cxh4
Platform: linux
Uptime: 31180
IP: 10.233.103.68
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 10.233.50.151:80
Message: Hello World!
Hostname: testapp-replicaset-8hxpx
Platform: linux
Uptime: 31229
IP: 10.233.101.76
DNS: 169.254.25.10
vagrant@kube-master1:~/test$
```

## sessionAffinity
- 클라이언트 요청을 매번 똑같은 파드로 연결하고 싶은 경우나 또는 그렇게 해야 할 경우 사용하는 서비스 항목 
- None: (기본) 세션 어피니티 없음
- ClientIP: 클라이언트의 IP를 확인해 같은 파드로 연결됨 

* sessionAffinity 메니페스트 구성하기
```
---
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-affi
spec:
  sessionAffinity: ClientIP
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: testapp-rs
...
```
```
$ watch -n1 kubectl get services,endpoints,pods --show-labels

Every 1.0s: kubectl get services,endpoints,pods --show-labels         kube-master1: Fri Jun 24 15:25:03 2022
NAME                       TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   LABELS
service/kubernetes         ClusterIP   10.233.0.1      <none>        443/TCP   10h   component=apiserver,provider=kubernetes
service/testapp-svc-affi   ClusterIP   10.233.24.222   <none>        80/TCP    60s   <none>

NAME                         ENDPOINTS                                                  AGE   LABELS
endpoints/kubernetes         192.168.56.11:6443                                         10h   <none>
endpoints/testapp-svc-affi   10.233.101.76:8080,10.233.103.68:8080,10.233.76.103:8080   60s   <none>

NAME                           READY   STATUS    RESTARTS   AGE     LABELS
pod/testapp-replicaset-8hxpx   1/1     Running   0          5h51m   app=testapp-rs
pod/testapp-replicaset-9cxh4   1/1     Running   0          5h51m   app=testapp-rs
pod/testapp-replicaset-ml8ps   1/1     Running   0          5h51m   app=testapp-rs
```

* 내부 네트워크에서 확인
```
vagrant@kube-master1:~/test$ curl 10.233.24.222:80
Message: Hello World!
Hostname: testapp-replicaset-ml8ps
Platform: linux
Uptime: 35123
IP: 10.233.76.103
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 10.233.24.222:80
Message: Hello World!
Hostname: testapp-replicaset-ml8ps
Platform: linux
Uptime: 35125
IP: 10.233.76.103
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 10.233.24.222:80
Message: Hello World!
Hostname: testapp-replicaset-ml8ps
Platform: linux
Uptime: 35125
IP: 10.233.76.103
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 10.233.24.222:80
Message: Hello World!
Hostname: testapp-replicaset-ml8ps
Platform: linux
Uptime: 35126
IP: 10.233.76.103
DNS: 169.254.25.10
```

## 서비스 다중 포트 연결 
- 파드의 애플리케이션은 여러 포트를 사용하여 서비스 가능하다.
- Ex) HTTP는 80번, HTTPS는 443번 다중 포트를 가진 파드 및 서비스를 구성
- 다중 포트를 설정할 때는 반드시 포트의 이름을 부여해야 함(.spec.ports.portname)

* 다중포트 서비스 사용하기
```
# testapp-svc-multiport.yml

apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-multiport
spec:
  ports:
  - name: testapp-http
    port: 80
    targetPort: 8080
  - name: testapp-https
    port: 443
    targetPort: 8443
  selector:
    app: testapp-rs

```

## 포트 이름 참조
- 파드나 파드를 생성하는 컨트롤러의 파드 템플릿의 컨테이너 포트에도 이름을 부여 가능
- 파드(컨테이너)의 포트 이름을 이용하여 서비스 생성 시 파드의 대상포트(targetPort)에 이름을 이용하여 연결
- 이름으로 참조하기 때문에 파드의 포트 이름만 변경되지 않았다면, 실제 파드의 포트 번호가 변경되더라도, 스펙의 변경 없이 계속해서 연결
```
---
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-named
spec:
  ports:
  - name: port-http
    port: 80
    targetPort: testapp-http
  - name: port-https
    port: 443
    targetPort: testapp-https
  selector:
    app: testapp-rs
...        
```

* ReplicaSet에서 해당 포트이름으로 연결하기
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: testapp-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: testapp-rs
  template:
    metadata:
      labels:
        app: testapp-rs
    spec:
      containers:
      - name: testapp
        image: c1t1d0s7/myweb
        ports:
        - name: testapp-http
          containerPort: 8080
...
```
```
$ watch -n1 kubectl get services,endpoints,pods --show-labels

Every 1.0s: kubectl get services,endpoints,pods --show-labels         kube-master1: Fri Jun 24 15:43:57 2022
NAME                        TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE     LABELS
service/kubernetes          ClusterIP   10.233.0.1    <none>        443/TCP          10h     component=apiserver,provider=kubernetes
service/testapp-svc-named   ClusterIP   10.233.47.5   <none>        80/TCP,443/TCP   5m22s   <none>

NAME                          ENDPOINTS            AGE     LABELS
endpoints/kubernetes          192.168.56.11:6443   10h     <none>
endpoints/testapp-svc-named   <none>               5m22s   <none>

NAME                           READY   STATUS    RESTARTS   AGE   LABELS
pod/testapp-replicaset-74jzq   1/1     Running   0          79s   app=testapp-rs
pod/testapp-replicaset-f926f   1/1     Running   0          79s   app=testapp-rs
pod/testapp-replicaset-gq6k4   1/1     Running   0          79s   app=testapp-rs
```
```
$ watch -n1 kubectl get services,endpoints,pods --show-labels

Every 1.0s: kubectl get services,endpoints,pods --show-labels         kube-master1: Fri Jun 24 16:02:11 2022
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE   LABELS
service/kubernetes          ClusterIP   10.233.0.1      <none>        443/TCP   10h   component=apiserver,provider=kubernetes
service/testapp-svc-named   ClusterIP   10.233.42.213   <none>        80/TCP    22s   <none>

NAME                          ENDPOINTS                                                  AGE   LABELS
endpoints/kubernetes          192.168.56.11:6443                                         10h   <none>
endpoints/testapp-svc-named   10.233.101.81:8080,10.233.103.73:8080,10.233.76.108:8080   22s   <none>

NAME                           READY   STATUS    RESTARTS   AGE   LABELS
pod/testapp-replicaset-2bpqw   1/1     Running   0          13s   app=testapp-rs
pod/testapp-replicaset-d2w8h   1/1     Running   0          13s   app=testapp-rs
pod/testapp-replicaset-w4g2z   1/1     Running   0          13s   app=testapp-rs

```

* Edit 을 이용한 수정 
```
$ kubectl edit replicasets.apps testapp-replicaset
```
```
.
.
.
    spec:
      containers:
      - image: c1t1d0s7/myweb
        imagePullPolicy: Always
        name: testapp
        ports:
        - containerPort: 8081  		// 8080 -> 8081 변경
          name: testapp-http
          protocol: TCP
        resources: {}
.
.
.
```

* 수정 후 replicaset, service delete 후 다시시작
```
$ kubectl delete service testapp-svc-named
$ kubectl delete replicaset.apps testapp-replicaset

$ kubectl create -f testapp-svc-named.yml
$ kubectl create -f testapp-replicaset.yaml
```
![화면 캡처 2022-06-24 151859](https://user-images.githubusercontent.com/57117748/175575202-9ea1446a-7f66-4d08-af91-c052c80b3596.png)

# 서비스 탐색
- K8s Cluster의 Service는 파드에서 직접 탐색하고 접근이 가능하다.

1. 환경 변수를 이용한 탐색 방법
2. DNS를 이용한 탐색 방법

## 환경변수를 이용한 서비스 탐색
- K8s Cluster 는 파드가 시작시 현재 존재하는 서비스를 파드내의 쉘 환경변수 설정시킴
- 서비스 이름만 알고 있다면 적절한 환경 변수를 참조해 접근하고자 하는 서비스의 IP와 포트를 확인가능

* 하나의 서비스에 설정되는 여러 쉘 환경 변수
```
- [SVC_NAME]_SERVICE_HOST=[SVC_IP]
- [SVC_NAME]_SERVICE_PORT=[SVC_PORT]
- [SVC_NAME]_PORT=[PROTOCOL]://[SVC_IP]:[SVC_PORT]
- [SVC_NAME]_PORT_[PORT_NUMBER]_[PROTOCOL]=[PROTOCOL]://[SVC_IP]:[SVC_PORT]
- [SVC_NAME]_PORT_[PORT_NUMBER]_[PROTOCOL]_ADDR=[SVC_IP]
- [SVC_NAME]_PORT_[PORT_NUMBER]_[PROTOCOL]_PORT=[SVC_PORT]
- [SVC_NAME]_PORT_[PORT_NUMBER]_[PROTOCOL]_PROTO=[PROTOCOL]
```

* 파드의 쉘 내에서 env 명령어를 통해 확인 가능
```
$ env
```

## DNS를 이용하는 서비스 탐색
- 쿠버네티스 클러스터는 시스템 내부에서 사용할 수 있는 DNS(CoreDNS) 서버가 동작중이다.
- DNS 관련 서비스인 service/coredns 서비스가 있으며 10.233.xxx.xxxx IP로 접근할 수 있고 표준 DNS 서버인 53번 포트를 이용하고 있다.
- K8s 1.15v 이전은 파드가 직접적으로 coredns 서비스에 DNS 쿼리/응답을 받었음으로 대규모 환경에서 병목현상을 만들어졌다.
- K8s 1.15v 부터 노드 로컬 DNS 캐시(NodeLocal DNSCache) 기능이 도입
- NodeLocal DNSCache : 각 노드에 DNS 캐시 기능을 가지고 있는 파드를 데몬셋으로 배치해 DNS 쿼리 및 응답을 coredns가 대신하게 되는 구조

* DNS 관련 리소스 확인
```
$ kubectl get all -n kube-system -l k8s-app=kube-dns
```

* NodeLocal DNSCache 작동방식
```
Pod <--> NodeLocal DNSCache(169.254.25.10) <--> iptables <--> coredns(10.233.0.3)
```

* FQDN을 이용한 탐색 
```
bash-5.1# curl http://testapp-svc

bash-5.1# curl http://testapp-svc.default

bash-5.1# curl http://testapp-svc.default.svc

bash-5.1# curl http://testapp-svc.default.svc.cluster.local
```
● mynapp-svc: 서비스의 이름
● default: 서비스 리소스의 네임스페이스 이름
● svc: 서비스 그 자체를 의미함
● cluster.local: 쿠버네티스 클러스터 도메인

FQDN의 주소 형식
<서비스 이름>.<네임스페이스>.<리소스 종류>.<클러스터 도메인>

# NodePort 
- 쿠버네티스 모든 노드(호스트)에 '외부 접근용 포트를 할당'
- 노드의 포트를 사용하여 외부에서 접근 가능
- 노드의 포트로 접근하면 서비스에 의해 파드로 리다이렉션
- 파드를 실행하고 있지 않는 노드에도 포트가 할당
- 기본적으로 30000-32767 포트 사용

## NodePort 사용
- 각 노드의 IP:지정포트로 접속 시 서비스 포트인 80번 포트로 리다이렉션 되며, 이는 다시 내부 파드의 8080번 포트로 리다이렉션

* NodePort 메니페스트 만들고 실행하기
```
---
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-nodeport
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30000
  selector:
    app: testapp-rs
...
```
```
$ kubectl get service --watch
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes             ClusterIP   10.233.0.1     <none>        443/TCP        32h
testapp-svc-nodeport   NodePort    10.233.23.2    <none>        80:30000/TCP   0s
```

* 노드IP:[30000:지정포트]로 각 노드에서 파드 확인하기
```
$ kubectl get nodes -o wide

NAME           STATUS   ROLES    AGE    VERSION    INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
kube-master1   Ready    master   5d6h   v1.18.10   192.168.56.11   <none>        Ubuntu 18.04.6 LTS   4.15.0-188-generic   docker://19.3.12
kube-node1     Ready    <none>   5d6h   v1.18.10   192.168.56.21   <none>        Ubuntu 18.04.6 LTS   4.15.0-188-generic   docker://19.3.12
kube-node2     Ready    <none>   5d6h   v1.18.10   192.168.56.22   <none>        Ubuntu 18.04.6 LTS   4.15.0-188-generic   docker://19.3.12
kube-node3     Ready    <none>   5d6h   v1.18.10   192.168.56.23   <none>        Ubuntu 18.04.6 LTS   4.15.0-188-generic   docker://19.3.12
```
```
vagrant@kube-master1:~/test$ curl 192.168.56.11:30000
Message: Hello World!
Hostname: testapp-replicaset-nnsd9
Platform: linux
Uptime: 2679
IP: 10.233.76.114
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 192.168.56.21:30000
Message: Hello World!
Hostname: testapp-replicaset-wt4hv
Platform: linux
Uptime: 2727
IP: 10.233.103.80
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 192.168.56.22:30000
Message: Hello World!
Hostname: testapp-replicaset-nnsd9
Platform: linux
Uptime: 2685
IP: 10.233.76.114
DNS: 169.254.25.10

vagrant@kube-master1:~/test$ curl 192.168.56.23:30000
Message: Hello World!
Hostname: testapp-replicaset-mh7m4
Platform: linux
Uptime: 2775
IP: 10.233.101.88
DNS: 169.254.25.10
```

## LoadBalancer 사용
- 클라우드 인프라에서 LoadBalancer 서비스를 생성하면 클라우드 인프라의 로드 밸런서를 자동으로 프로비저닝 하게 되며, 이 로드 밸런서를 통해 서비스와 파드에 접근

환경설정

Kubernetes 로드밸런서 서비스를 위한 MetalLB 설치

- 디렉토리 변경
```
$ cd ~/kubespray
```

- Kubernetes 클러스터 설정 변경
```
$ vi inventory/mycluster/group_vars/k8s-cluster/k8s-cluster.yml

$ kube_proxy_strict_arp: true
```

- 로드밸런서에 할당할 IP 범위 설정
```
$ vi contrib/metallb/roles/provision/defaults/main.yml

ip_range: "192.168.56.200-192.168.56.220"
```

- MetalLB 설치
```
ansible-playbook -i inventory/mycluster/inventory.ini contrib/metallb/metallb.yml -b
```


- MetalLB 네임스페이스 확인
```
$ kubectl get ns
```

LoadBalancer 서비스 생성
```
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: testapp-rs
```
LoadBalancer 서비스 확인
```
kubectl get service testapp-svc-lb
```
로드 밸런서의 외부 IP가 192.168.56.200으로 정상적으로 할당된 것을 확인할 수 있음

클라우드의 관리형 쿠버네티스 서비스 및 MetalLB를 사용하지 않는 경우, 외부 로드 밸런서가 프로비저닝 되지 않기 때문에 외부 IP(로드 밸런서 IP)가 할당되지 않고 <pending> 상태로 보고

실제 외부 로드 밸런서를 사용할 수 있는 경우, 외부의 로드 밸런서가 생성되고 IP가 할당되기까지 시간이 필요

로드 밸런서 서비스의 IP로 접근
```
curl http://192.168.56.200
```

## ExternalName 서비스 생성
- 외부에서 접근하기 위한 서비스 종류가 아닌 내부 파트가 외부의 특정 FQDN에 쉽게 접근하기 위한 서비스
- 즉, 쿠버네티스 클러스터의 coredns 서비스가 특정 FQDN에 대한 CNAME(서비스의 FQDN)을 제공함에 따라 해당 CNAME을 이용하여 쉽게 통신할 수 있음

* ExternalName 메니페스토 만들기 
```
apiVersion: v1
kind: Service
metadata:
  name: testapp-svc-extname
spec:
  type: ExternalName
  externalName: www.google.com
```

* 파드로 접근
```
﻿$ kubectl run <Pod_Name> -it --image=centos:7
```

* 해당 파드에서 확인
```
$ curl <ExternalName>.<namespaceName>.svc.cluster.local

```

## 키워드
- Service : 쿠버네티스 시스템에서 같은 애플리케이션을 실행하는 컨트롤러 파드 그룹에 '단일 네트워크 진입' 리소스로 고유한 도메인 이름을 부여(FQDN), 로드 밸런서의 역할 수행 

- ClusterIP : 클러스터 내부에서만 포드에 접근이 가능한 서비스.

- sessionAffinity : ClusterIP를 실행 할때 분산접속이 아닌 하나의 세션 즉 고정된 파드에서 계속 작업 할 수 있도록 하는 옵션

- 파드 포트 다중연결 : 하나의 파드에서 여러 포트를 연결 시키는것 

- 파드 이름 참조 : 컨트롤러의 파드 템플릿의 컨테이너 포트에 이름을 부여해 서비스에서 해당 포트 이름으로 연결할 수 있는 방법

- NodePort : 연결된 노드들에게 외부 접근용 포트를 할당 노드의 포트로 접근하면 서비스에 의해 파드로 리다이렉션

- LoadBalancer :  클라우드 인프라에서 LoadBalancer 서비스를 생성하면 클라우드 인프라의 로드 밸런서를 자동으로 프로비저닝 하게 되며 로드 밸런싱의 역할을 함

- ExternalName :  내부 파트가 외부의 특정 FQDN에 쉽게 접근하기 위한 서비스