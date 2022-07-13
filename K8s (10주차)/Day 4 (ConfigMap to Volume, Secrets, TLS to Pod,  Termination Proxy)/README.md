# annotation(애너테이션)
- 추가 정보를 저장 (주석과 유사)
- 고유한 속성을 정의 (인그레스 리다이렉트 / 롤아웃기록 등)

# ConfigMap
- 별도의 데이터 저장을 위한 오브젝트
- 개발, 배포, 운영사이에 더 쉬운 환경 변수를 찾기 위해 사용
- 같은 이미지를 이용해서 설정이 다른 컨테이너를 실행하려고 사용
- 컨피그맵을 설정하고 해당 파드에서 그 값을 사용(전체 또는 일부 값을 호출해서 사용)
- 환경변수로 호출 : 변수의 값으로 설정
- volume 형태로 마운트 : 파일

```
도커		ENTRYPOINT		CMD
쿠버네티스	command		args
```

## env 변수로 어플리케이션으로 값전달
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod-env
spec:
  containers:
  - image: ghcr.io/c1t1d0s7/go-myweb:alpine
    name: myapp
    env:
    - name: MESSAGE
      value: "Customized Hello World!"
    ports:
    - containerPort: 8080
      protocol: TCP
```

- 8080 포트 사용 확인
```
$ netstat -antp | grep 8080
```

- 8080 포트로 포트 포워딩
```
$ kubectl port-forward myapp-pod-env 8080:8080
```

- 새로운 터미널에서 env 환경변수 확인
```
$ curl http://localhost:8080
Customized Hello World!
myapp-pod-env
``` 

## ConfigMap 생성 방법
1. 명령어 이용하여 생성
```
$ kubectl create configmap  <이름>  [--from-file=source]  [--from-listeral=key1=vaule1]

									--form-literal=key1=value1  옵션으로 파일을 참조
									--from-file=source
									--from-file=source=key=source
```
```
$ kubectl get configmap
$ kubectl describe configmap [이름]
```

2. object 파일을 이용
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod-cm
spec:
  containers:
  - image: ghcr.io/c1t1d0s7/go-myweb:alpine
    name: myapp
    env:
    - name: MESSAGE
      valueFrom:
        configMapKeyRef:
          name: myapp-message      # configmap name
          key: message                       # key 또는 file명
    args:
    - $(MESSAGE)
    ports:
    - containerPort: 8080
      protocol: TCP

```

- cnofigMap 생성
```
$ kubectl create configmap myapp-message --from-file=configmap/
$ kugectl get configmap
```

- configmap pod 생성 
```
$ kubectl create -f myapp-pod-cm.yaml
$ kubectl get pod
```

- 포트 포워딩
```
$ kubectl port-forward myapp-pod-cm 8080:8080
```

- 새로운 터미널에서 확인
```
$ curl http://localhost:8080
Hello World by ConfigMap
myapp-pod-cm
```
# configmap으로 볼륨(File) 사용하기

- 리눅스의 특성상 폴더 및 파일에 마운트를 진행 할때 이미 어떠한 데이터 및 파일이 있다면 마운트 된 후 기존에 존재하던 파일은 접근할 수 없다.
- 이것을 이용해 nginx conf 파일을 configMap 으로 이용해 /etc/nginx/conf.d/ 에 마운트하여 설정한다

1. configMap를 만들어줄 web 구성요소 즉, nginx.conf 파일을 만들어 준다.
-/etc/nginx.conf 기본 설정 파일
```
$ vi nginx-gzip.conf
```
```
server {
    listen              80;
    server_name         myapp.example.com;
    gzip on;
    gzip_types text/plain application/xml;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

2. configMap으로 만들기 
```
kubectl create configmap nginx-gzip-config --from-file=nginx-gzip.conf
```

3. configMap 확인
```
$ kubectl get configmap
NAME               DATA   AGE
nginx-tls-config   1      3h19m
 

$ kubectl describe configmap nginx-tls-config
Name:         nginx-tls-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx-tls.conf:
----
server {
    listen              80;
    listen              443 ssl;
    server_name         myapp.example.com;
    ssl_certificate     /etc/nginx/ssl/tls.crt;
    ssl_certificate_key /etc/nginx/ssl/tls.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}


Events:  <none>
```

4. configMap와 연결할 파드를 작성 
- -/etc/nginx/conf.d 추가 설정 디렉토리
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-compress
spec:
  containers:
  - image: nginx
    name: nginx-compress
    volumeMounts:					  # 리눅스 특성으로 마운트 하면 그전 파일은 접근 불가
    - name: nginx-compress-config
      mountPath: /etc/nginx/conf.d/
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: nginx-compress-config
    configMap:
      name: nginx-gzip-config         # configmap 이름 (pod 생성시 있어야 함)
```

5. 파드 생성
```
$ kubectl create -f nginx-pod-compress.yml
```

6. 확인
```
$ kubectl describe pod nginx-pod-compress
```

7. 포트포워딩
```
$kubectl port-forward nginx-pod-compress 8080:80
```

8. 확인
- 다른 터미널에서
```
curl http://localhost:8080 -v
```

# Secret
- ConfigMap과 같은 키/값 저장소
- ConifgMap은 단순 데이터, 민감하지 않은 설정 데이터를 저장 및 참조 용도
- Secret 패스워드, 인증서, 키, 토큰 등 소량의 민감한 데이터를 저장한다.
- BASE64 를 통한 인코딩
- 개별 오브젝트당 1MiB로 제한
- [시크릿](https://kubernetes.io/ko/docs/concepts/configuration/secret/)
## secret의 데이터의 타입

-secret에 직접 저장할 수 있는 데이터의 타입
	∙ generic: (기본) 키-값 형식의 임의 데이터
	∙ docker-registry: 도커 저장소 인증 정보
	∙ tls: TLS 키 및 인증서

- secret에 직접 저장할 수 없는 데이터의 타입
	∙ service-account-token: 서비스 계정 토큰

- kubectl get secrets에 표시되는 데이터 타입
	∙ Opaque (=generic)
	∙ kubernetes.io/dockerconfigjson
	∙ kubernetes.io/tls
	∙ kubernetes.io/service-account-token

## Base64 encoding, decodeing

- Base64를 통한 인코딩
```
$ echo "q1w2e3r4" | base64
cTF3MmUzcjQK
```

- Base64를 통한 디코딩
```
echo "cTF3MmUzcjQK" | base64 --decode
```

## secrets 생성
1. 명령을 통한 생성
```
$ kubectl create secret <TYPE> <NAME> [OPTIONS]
``` 

2. 일반 시크릿 생성 방법
```
$ kubectl create secret generic <NAME> [--from-file=[key=]source] [--from-literal=key1=value1]
```

3. 도커 저장소 인증 정보용 시크릿 생성 방법
```
$ kubectl create secret docker-registry <NAME> \
> --docker-username=user --docker-password=password \
> --docker-email=email [--docker-server=string]
```

4. TLS 키/인증서용 시크릿 생성 방법
```
$ kubectl create secrets tls <NAME> --cert=cert-file --key=key-file
```

## base64 암호화 secret 만들기

1. 시크릿에 저장할 파일을 생성
- 시크릿에 저장할 암호화된 파일 만들기
```
$ echo -n "admin" > id.txt
$ echo -n "q1w2e3r4" > pwd.txt
```

- 시크릿 만들기
```
$ kubectl create secret generic my-secret --from-file=id.txt --from-file=pwd.txt // 파일을 이용한 시크릿 만들기
```

- 확인
```
$ kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-n7fm9   kubernetes.io/service-account-token   3      10d
my-secret             Opaque                                2      47s

$ kubectl get secrets my-secret –o yaml
apiVersion: v1
data:
  id.text: YWRtaW4=
  pwd.text: cTF3MmUzcjQ=
kind: Secret
metadata:
.
.
.
```

2. 오브젝트파일을 이용해서 시크릿 정의하기
```
apiVersion: v1
kind: Secret
metadata:
  name: user-pass-yaml
type: Opaque
data:
  username: YWRtaW4K
  password: cTF3MmUzcjQK
```

# secret을 이용한 Nginx https 웹서비스 구축
- nginx 설정파일 conf파일
- 자체서명인증서 cert, key 

1. 인증서 생성
```
$ openssl req -x509 -nodes -newkey rsa:2048 \
> -keyout tls.key -out tls.crt -days 365 \
> -subj "/CN=myapp.example.com"
```

2. 인증서를 secret 생성
```
$ kubectl create secret tls nginx-tls-secret \
> --cert=./tls.crt \
> --key=./tls.key
```

3. secret 확인
```
$ kubectl get secrets
default-token-n7fm9   kubernetes.io/service-account-token   3      10d
nginx-tls-secret      kubernetes.io/tls                     2      2m52s


$ kubectl describe secrets nginx-tls-secret
Name:         nginx-tls-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  kubernetes.io/tls

Data
====
tls.crt:  1135 bytes
tls.key:  1704 bytes


$ kubectl get secrets nginx-tls-secret -o yaml 	// output된 파일을 yaml 파일 형태로 본다
$ kubectl get secrets nginx-tls-secret -o json 	// output된 파일을 json 파일 형태로 본다
```

4. nginx conf 구성파일 생성
```
server {
    listen              80;
    listen              443 ssl;
    server_name         myapp.example.com;
    ssl_certificate     /etc/nginx/ssl/tls.crt;
    ssl_certificate_key /etc/nginx/ssl/tls.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}
```

5. nginx conf구성파일을 configmap으로 생성
```
$ kubectl create configmap nginx-tls-config --from-file=conf/nginx-tls.conf
```

6. 확인
```
$ kubectl get configmap
NAME               DATA   AGE
nginx-tls-config   1      42s

$ kubectl describe configmap nginx-tls-config
Name:         nginx-tls-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
nginx-tls.conf:
----
server {
    listen              80;
    listen              443 ssl;
    server_name         myapp.example.com;
    ssl_certificate     /etc/nginx/ssl/tls.crt;
    ssl_certificate_key /etc/nginx/ssl/tls.key;
    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
    }
}

Events:  <none>
```

7. nginx 파드 생성 (secret, configmap 사용)
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-https
spec:
  containers:
  - image: nginx
    name: nginx-https
    volumeMounts:
    - name: nginx-tls-config
      mountPath: /etc/nginx/conf.d
    - name: https-cert
      mountPath: /etc/nginx/ssl
      readOnly: true
    ports:
    - containerPort: 80
      protocol: TCP
    - containerPort: 443
      protocol: TCP
  volumes:
  - name: nginx-tls-config
    configMap:
      name: nginx-tls-config  // 리눅스 특성을 이용한 nginx-tls-config을 볼륨으로 /etc/nginx/conf.d 경로에 nginx-tls.conf를 마운트
  - name: https-cert
    secret:
      secretName: nginx-tls-secret 	// configMap에게 인증서가 Secret 명시 
```

8. 실행
```
$ kubectl create -f nginx-pod-https.yaml
```

9. 포트 확인 및 포트 포워딩
```
$ netstat -antp | grep :8443  // 해당 포트 사용하고 있는지 확인
$ kubectl port-forward  nginx-pod-https 8443:443  //포트 포워딩
```

10. 새로운 터미널에서 접속 확인
```
$ curl https://localhost:8443 -k -v

* Rebuilt URL to: https://localhost:8443/
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 8443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: /etc/ssl/certs/ca-certificates.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Unknown (8):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Client hello (1):
* TLSv1.3 (OUT), TLS Unknown, Certificate Status (22):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* ALPN, server accepted to use http/1.1
* Server certificate:
*  subject: CN=myapp.example.com
*  start date: Jun 30 14:40:07 2022 GMT
*  expire date: Jun 30 14:40:07 2023 GMT
*  issuer: CN=myapp.example.com
*  SSL certificate verify result: self signed certificate (18), continuing anyway.
* TLSv1.3 (OUT), TLS Unknown, Unknown (23):
> GET / HTTP/1.1
> Host: localhost:8443
> User-Agent: curl/7.58.0
> Accept: */*
>
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Certificate Status (22):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS Unknown, Unknown (23):
< HTTP/1.1 200 OK
< Server: nginx/1.23.0
< Date: Thu, 30 Jun 2022 14:45:26 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Tue, 21 Jun 2022 14:25:37 GMT
< Connection: keep-alive
< ETag: "62b1d4e1-267"
< Accept-Ranges: bytes
<
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host localhost left intact
```

# TLS Termination Proxy
- 최근에는 Web Pod가 아닌 LB에서 인증서를 관리하도록 한다 
- 공격을 받을시 servre-client 사이 LB가 중간에 TLS 세션을 끊어서 대처가능
- TLS 종료 프록시(Termination Proxy)

* 그전 까지 pod에 설치하였지만 LB에 TLS 인증서 관리하도록 한다. 
![화면 캡처 2022-06-30 234948](https://user-images.githubusercontent.com/57117748/176708197-3ac04dbc-109f-438a-8df4-6562bda3c4c9.png)

## TLS 종료 프록시(Termination Proxy) 사용
- 로드밸런서는 ingress를 통해 구축

1. 서비스, 리플리카셋 생성 (TLS 종료 프록시)
- 서비스 메니페스트
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
    targetPort: 8080
    nodePort: 30000
  selector:
    app: myapp-rs
...
```

- 레플리카세트 메니페스트
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
        - containerPort: 8080
...
```

- 실행
```
kubectl create -f myapp-rs.yml –f myapp-svc-np.yml
```

2. 인증서 및 secret 생성
- 인증서 만들기
```
openssl req  -x509 -nodes -newkey rsa:2048 -keyout tls.key \
-out tls.crt -days 365 \
-subj "/CN=myapp.example.com"
```

- secret 생성
```
kubectl create secret tls myapp-tls-secret \
--cert=./tls.crt \
--key=./tls.key 

```

- ingress controller 생성
```
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: myapp-ing-tls-term
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: myapp-tls-secret
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

- ingress controller 실행
```
kubectl create –f myapp-ing-tls-term.yaml
```

- 확인
```
//  backenc service (nortport) 를 인식함
$ kubectl get ingresses.networking.k8s.io
NAME                 CLASS    HOSTS               ADDRESS                                     PORTS     AGE
myapp-ing-tls-term   <none>   myapp.example.com   192.168.56.21,192.168.56.22,192.168.56.23   80, 443 

# curl --resolve myapp.example.com:443:192.168.56.21 -k https://myapp.example.com
```
