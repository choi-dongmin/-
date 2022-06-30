# Ingress
- Ingress 는 L7 LB  , application LB  , HTTP LB (Matallb 는 L4 LB)
- URL, Hostname을 기준으로 LoadBalancing을 진행
- 외부의 노출이 존재함

* Ingress 
![화면 캡처 2022-06-27 142711](https://user-images.githubusercontent.com/57117748/175907159-9a183d34-f45a-40ad-814a-40242f11141a.png)

## Ingress 사용
0. 사용하기전 Ingress 버전을 확인 
```
$ kubectl api-resources  | grep ^ingress
ingresses                         ing          extensions                     true         Ingress
ingressclasses                                 networking.k8s.io              false        IngressClass
ingresses                         ing          networking.k8s.io              true         Ingress

$ kubectl explain ing --api-version=extensions/v1beta1
$ kubectl explain ing --api-version=networking.k8s.io/v1beta1
```

1. Ingrass Manifest 작성
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myapp-ing
spec:
  rules:
  - host: myapp.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp-svc-np
          servicePort: 80
...
```

2. Service manifest 작성
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-np
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: myapp-http
    nodePort: 30000
  selector:
    app: myapp-rs
...
```

3. ReplicaSet manifest 작성
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-rs
  template:
    metadata:
      labels:
        app: myapp-rs
    spec:
      containers:
      - name: myapp
        image: ghcr.io/c1t1d0s7/go-myweb:alpine
        ports:
        - name: myapp-http
          containerPort: 8080
...
```

4. 실행 및 확인
```
$ kubectl create -f myapp-ing.yml -f myapp-rs.yml -f myapp-svc-np.yml
```
```
$ kubectl get all,ing
NAME                 READY   STATUS    RESTARTS   AGE
pod/myapp-rs-j92bh   1/1     Running   0          21s
pod/myapp-rs-l7mtn   1/1     Running   0          21s
pod/myapp-rs-ppgxg   1/1     Running   0          21s

NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE 
service/kubernetes     ClusterIP   10.233.0.1      <none>        443/TCP        21h
service/myapp-svc-np   NodePort    10.233.27.154   <none>        80:30000/TCP   21s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs   3         3         3       21s

NAME                           CLASS    HOSTS               ADDRESS                                     PORTS   AGE
ingress.extensions/myapp-ing   <none>   myapp.example.com   192.168.56.21,192.168.56.22,192.168.56.23   80      21s
```
```
$ curl http://192.168.56.21:30000
Hello World!
myapp-rs-l7mtn

$ curl http://192.168.56.22:30000
Hello World!
myapp-rs-l7mtn

$ curl http://192.168.56.23:30000
Hello World!
myapp-rs-l7mtn

# -v 옵션으로 더 자세한 정보
$ curl http://192.168.56.23:30000 -v
* Rebuilt URL to: http://192.168.56.23:30000/
*   Trying 192.168.56.23...
* TCP_NODELAY set
* Connected to 192.168.56.23 (192.168.56.23) port 30000 (#0)
> GET / HTTP/1.1
> Host: 192.168.56.23:30000
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Mon, 27 Jun 2022 13:20:56 GMT
< Content-Length: 28
< Content-Type: text/plain; charset=utf-8
<
Hello World!
myapp-rs-ppgxg
* Connection #0 to host 192.168.56.23 left intact
vagrant@kube-master1:~/ingress$ clear
```


## curl -v 옵션을 통한 자세한 정보 확인
```
$ curl 192.168.56.23 -v
* Rebuilt URL to: 192.168.56.23/
*   Trying 192.168.56.23...
* TCP_NODELAY set
* Connected to 192.168.56.23 (192.168.56.23) port 80 (#0)
> GET / HTTP/1.1
> Host: 192.168.56.23
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 404 Not Found
< Server: nginx/1.19.2
< Date: Mon, 27 Jun 2022 13:25:39 GMT
< Content-Type: text/html
< Content-Length: 153
< Connection: keep-alive
<
<html>
<head><title>404 Not Found</title></head>
<body>
<center><h1>404 Not Found</h1></center>
<hr><center>nginx/1.19.2</center>
</body>
</html>
* Connection #0 to host 192.168.56.23 left intact


> HTTP 요청
< HTTP 응답
404 error -  ingress LB 가 응답 (어디로 가야할지 모르겠음)
URL, hostname 이 맞아야 이에 rules 해당되는 서비스에 연결
```

## Hostname 으로 접근하는 방법
1. curl --resolve 이용
```
$ curl --resolve myapp.example.com:80:192.168.56.21 http://myapp.example.com
Hello World!
myapp-rs-ppgxg

$ curl --resolve myapp.example.com:80:192.168.56.22 http://myapp.example.com
Hello World!
myapp-rs-j92bh

$ curl --resolve myapp.example.com:80:192.168.56.23 http://myapp.example.com
Hello World!
```

2. /etc/hosts 파일에 Host 추가
- DNS 서버를 확인하기 전에 /etc/hosts 파일을 먼저 확인하기 때문에 추가한다면 hostname으로 사용 가능
```
$ cat /etc/hosts | tail -1
192.168.56.22   myapp.example.com

$ curl myapp.example.com
Hello World!
myapp-rs-j92bh
```

3. wildcard dns
- nip.io, xip.ip, sslip.io 등의 Wildcard DNS를 사용하는 방법
[sslip.io](https://sslip.io/)
[nip.io](https://nip.io/)

```
$ host www.1.1.1.1.nip.io
www.1.1.1.1.nip.io has address 1.1.1.1

$ host www.1-1-1-1-1.nip.io
www.1-1-1-1-1.nip.io has address 1.1.1.1

$ host abc.1-2-3-4.nip.io
abc.1-2-3-4.nip.io has address 1.2.3.4
```

# HTTP LB
- LB (ingress), service, rs(pod)
- ingress L7 LB  . http hostname이나 URL에 따라 service를 다르게 가능함.

1. PATH 에 따라서 서비스를 다르게 가능
2. hostname 에 따라서 서비스를 다르게 가능

## 다중 경로 (Paths)
- PATH에 따라서 서비스 다른 서비스를 연결하는 방법

* 다중 경로 HTTP LB 생성
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: myapp-ing-mpath
spec:
  rules:
  - host: www.192-168-56-21.nip.io
    http:
      paths:
      - path: /web1
        backend:
          serviceName: myapp-svc-ext-np01
          servicePort: 80
      - path: /web2
        backend:
          serviceName: myapp-svc-ext-np02
          servicePort: 80

```

* np01,02 2개 서비스 만들기
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-np01
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: myapp-http
    nodePort: 30001
  selector:
    app: myapp-rs
...
```
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-np02
spec:
  type: NodePort
  ports:
  - port: 80
    targetPort: myapp-http
    nodePort: 30002
  selector:
    dev: admin
...
```

* ReplicaSet 설정
```
---
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp-rs
  template:
    metadata:
      labels:
        app: myapp-rs
    spec:
      containers:
      - name: myapp
        image: ghcr.io/c1t1d0s7/go-myweb:alpine
        ports:
        - name: myapp-http
          containerPort: 8080
...
```

* 생성 및 확인을 위한 라벨 추가
```
$ kubectl create -f myapp-svc-np01.yml -f myapp-svc-np02.yml -f myapp-ing-multi-paths.yml -f myapp-rs.yml

$ kubectl label pods myapp-rs-zbn9p dev=admin
```

* 생성확인
```
$ kubectl get all,ing --show-labels
NAME                 READY   STATUS    RESTARTS   AGE   LABELS
pod/myapp-rs-g7vjj   1/1     Running   0          12m   app=myapp-rs
pod/myapp-rs-rszf8   1/1     Running   0          12m   app=myapp-rs
pod/myapp-rs-zbn9p   1/1     Running   0          12m   app=myapp-rs,dev=admin

NAME                     TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE   LABELS
service/kubernetes       ClusterIP   10.233.0.1     <none>        443/TCP        23h   component=apiserver,provider=kubernetes
service/myapp-svc-np01   NodePort    10.233.57.89   <none>        80:30001/TCP   12m   <none>
service/myapp-svc-np02   NodePort    10.233.11.93   <none>        80:30002/TCP   12m   <none>

NAME                       DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/myapp-rs   3         3         3       12m   <none>

NAME                                       CLASS    HOSTS                      ADDRESS                                     PORTS   AGE   LABELS
ingress.extensions/myapp-ing-multi-paths   <none>   www.192.168.56.21.nip.io   192.168.56.21,192.168.56.22,192.168.56.23   80      12m   <none>
```

* 연결확인
```
$ kubectl describe ingress myapp-ing-multi-paths
Name:             myapp-ing-multi-paths
Namespace:        default
Address:          192.168.56.21,192.168.56.22,192.168.56.23
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host                      Path  Backends
  ----                      ----  --------
  www.192.168.56.21.nip.io
                            /web1   myapp-svc-np01:80 (10.233.101.102:8080,10.233.103.99:8080,10.233.76.129:8080)
                            /web2   myapp-svc-np02:80 (10.233.101.102:8080)
Annotations:                <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
  Normal  CREATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
  Normal  CREATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
  Normal  UPDATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
  Normal  UPDATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
  Normal  UPDATE  12m   nginx-ingress-controller  Ingress default/myapp-ing-multi-paths
```

## hostname에 따라 서비스 다르게 하게
- path와 다르게 hostname에 따라 다르게 하는 서비스
```
FQDN/ PATH					서비스
web1.example.com			myapp-svc-np1
web2.example.com			myapp-svc-np2
```

* hostname에 따라 다르게 하는 LB 생성
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: myapp-ing-multi-host
spec:
  rules:
  - host: web1.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp-svc-ext-np01
          servicePort: 80
  - host: web2.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: myapp-svc-ext-np02
          servicePort: 80
```

* 생성 확인
```
$ kubectl get all,ing
NAME                 READY   STATUS    RESTARTS   AGE
pod/myapp-rs-g7vjj   1/1     Running   1          32m
pod/myapp-rs-rszf8   1/1     Running   1          32m
pod/myapp-rs-zbn9p   1/1     Running   1          32m

NAME                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes       ClusterIP   10.233.0.1      <none>        443/TCP        23h
service/myapp-svc-np01   NodePort    10.233.45.192   <none>        80:30001/TCP   41s
service/myapp-svc-np02   NodePort    10.233.17.17    <none>        80:30002/TCP   41s

NAME                       DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs   3         3         3       32m

NAME                                      CLASS    HOSTS                               ADDRESS                                     PORTS   AGE
ingress.extensions/myapp-ing-multi-host   <none>   web1.example.com,web2.example.com   192.168.56.21,192.168.56.22,192.168.56.23   80      41s
```

* 연결 확인
```
$ kubectl describe ingress myapp-ing-multi-host
Name:             myapp-ing-multi-host
Namespace:        default
Address:          192.168.56.21,192.168.56.22,192.168.56.23
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host              Path  Backends
  ----              ----  --------
  web1.example.com
                    /   myapp-svc-ext-np01:80 (<error: endpoints "myapp-svc-ext-np01" not found>)
  web2.example.com
                    /   myapp-svc-ext-np02:80 (<error: endpoints "myapp-svc-ext-np02" not found>)
Annotations:        <none>
Events:
  Type    Reason  Age   From                      Message
  ----    ------  ----  ----                      -------
  Normal  CREATE  100s  nginx-ingress-controller  Ingress default/myapp-ing-multi-host
  Normal  CREATE  100s  nginx-ingress-controller  Ingress default/myapp-ing-multi-host
  Normal  CREATE  99s   nginx-ingress-controller  Ingress default/myapp-ing-multi-host
  Normal  UPDATE  90s   nginx-ingress-controller  Ingress default/myapp-ing-multi-host
  Normal  UPDATE  89s   nginx-ingress-controller  Ingress default/myapp-ing-multi-host
  Normal  UPDATE  89s   nginx-ingress-controller  Ingress default/myapp-ing-multi-host
```

* /etc/hosts 파일 수정
```
$ cat /etc/hosts | tail -2
192.168.56.22   web2.example.com
192.168.56.23   web1.example.com
```

## 키워드

Metallb : K8s의 로드 밸런싱을 지원하는 addon, L4 계층에서 TCP/UDP를 이용해 로드 밸런싱하는 방법 

ingress : L7 계층에서 URL/Path에 따라 로드 밸런싱 하는 애드온 외부에서 ingress로 연결 하면 서비스로 로드 밸런싱 후 파드로 연결

다중 경로 연결 : ingress를 설정 할때 여러 경로를 설정해 각각 다른 서비스로 연결

다중 호스트 연결 :  ingress를 설정 할때 여러 URL를 설정해 각각 다른 서비스로 연결



