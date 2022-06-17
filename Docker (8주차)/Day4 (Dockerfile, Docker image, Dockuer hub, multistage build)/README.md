# Docker image 
- 컨테이너를 실행하는데 필요한 구성 요소를 정의
- 컨테이를 통한 생성(Commit) 혹은 Dockerfile 을 통한 생성

## Dockerfile
- 도커 이미지를 만들기 위한 구성 파일
- Dockerfile 을 정의 한 후에 docker build 명령어로 이미지를 만든다

## Dockerfile 구문 소개 
1. FROM
	- 기본이 되는 이미지를 지정한다.
	- 필수적인 구문
	- bulid 시에 FROM 항목만 작성하고 이름만 바꾸어 설정하면 Image ID 값이 원본이랑 같을 수 있다.
	- Dokerfile 이름으로 지정 했다면 bulid 시에 default 값으로 지정되고 다른 이름이면 build 시에 -f 옵션을 통해 경로 및 이름 지정

2. CMD 
	- 컨테이너 실행 시 동작할 항목(프로세스)
	- 기본적인 동작은 ENTRYPOINT와 같음
	- CMD 만 사용시 run 명령어에서 이미지 이름 뒤에 명령어 따로 지정 시 해당 명령어를 실행
	- CMD와 ENTRYPOINT 두 항목 모두 사용시 CMD는 ENTRYPOINT의 인자값이 되고 run을 에서 이미지 뒤에 명령어 사용 시 ENTRYPOINT의 인자값으로 전달(CMD 무시)   
3. ENTRYPOINT 
	- 컨테이너 실행 시 동작할 항목(프로세스)  
	- 기본적은 CMDd와 같다
	- ENTRYPOINT 만 사용시 run 명령어에서 이미지 이름 뒤에 명령어 따로 지정 시 그 값은 인자값으로 전달
	- CMD와 ENTRYPOINT 두 항목 모두 사용시 CMD는 ENTRYPOINT의 인자값이 되고 run을 에서 이미지 뒤에 명령어 사용 시 ENTRYPOINT의 인자값으로 전달(CMD 무시)

4. RUN 
	- 명령어 및 작업을 입력하면 실행
	- 작업을 실행 시키는 섹션
	- shell, exec 방식의 입력 방식 둘 다 사용 가능

5. COPY
	- 지정한 파일을 이미지 생성 시 이미지 안에 복사(로컬파일)
6. ADD
	- 지정한 파일을 이미지 생성 시 이미지 안에 복사(외부 혹은 로컬)

7. EXPOSE
	- 포트에 대한 내보내기 설정
8. ENV
	- 이미지에서 사용할 기본 변수들(이미지 / 컨테이너 안)
9. VOLUME 
	- 이미지에서 필요로 하는 볼륨 연결 위치

10. USER
	- 컨테이너 실행 시 작업할 사용자 지정
11. WORKDIR
	- 컨테이너 실행 시 작업할 디렉토리
12. SHELL
	- 컨터이너 실행 시 작업할 쉘

13. LABEL
	- 주석

## 멀티스테이지 방식

- 그냥 베이스 이미지에 직접 이미지을 만들시에 결과물에 개발 도구가 남아 제품환경에는 불필요한 리소스를 차지하게 된다
- 멀티스테이지 빌드란?
컨테이너 이미지를 만들면서 빌드 등에는 필요하지만, 최종 컨테이너 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어 기반 이미지를 만드는 방법

- 멀티스테이트 방식 Dockerfile 구성
```
# 1. Build Image
FROM golang:1.13 AS builder
 
# Install dependencies
WORKDIR /go/src/github.com/asashiho/dockertext-greet
RUN go get -d -v github.com/urfave/cli
 
# Build modules
COPY main.go .
RUN GOOS=linux go build -a -o greet .
 
# ------------------------------
# 제품환경 이미지
FROM busybox
WORKDIR /opt/greet/bin
 
# 개발환경 모드
COPY --from=builder /go/src/github.com/asashiho/dockertext-greet/ .
ENTRYPOINT ["./greet"]
```

- bulid 실행
```	
# docker build -t greet .
```

- 확인 (개발환경 803m / 제품환경 6.1m)
``` 
# docker image
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
greet               latest              6a9a7567bffc        51 seconds ago      6.1MB
<none>              <none>              97ca0eb13472        56 seconds ago      837MB
busybox             latest              388056c9a683        3 weeks ago         1.23MB
golang              1.13                d6f3656320fe        8 months ago        803MB
```

## 이미지 생성 시 고려 사항
1. CMD / ENTRYPOINT
	- CMD 만 사용시 run 명령어에서 이미지 이름 뒤에 명령어 따로 지정 시 해당 명령어를 실행
	- ENTRYPOINT 만 사용시 run 명령어에서 이미지 이름 뒤에 명령어 따로 지정 시 그 값은 인자값으로 전달
	- CMD와 ENTRYPOINT 두 항목 모두 사용시 CMD는 ENTRYPOINT의 인자값이 되고 run을 에서 이미지 뒤에 명령어 사용 시 ENTRYPOINT의 인자값으로 전달(CMD 무시)

2. 이미지 / 컨테이너
	- 이미지 빌드 작업을 위해서 사용한 컨테이너는 자동 삭제 다운로드한 이미지는 계속 남아있다.

3. 멀티스테이지 빌드 방식
	- 베이스 이미지로 바로 작업 : 작업에 필요한 도구가 해당 이미지에 있어야 작업 가능 작업이 끝나도 결과물(이미지)에 해당 도구 계속 존재
	- 멀티스테이지 빌드 : 불필요한 도구를 제거하고 이미지 생성 가능
	- 로컬 호스트에 도구 직접 설치 : 필요한 도구를 직접 설치하고 결과물만으로 새 이미지 생성(경량화된 이미지 생성 가능 대신 로컬 시스템이 지저분해짐)


## LAYERS
- 이미지가 구성되기 위해 어떤 파일로 이루어져 있는가
- 그 이미지가 생성될 때 작업한 횟수
- image inspect 을 통해 확인 가능
- FROM, RUN 2번 동작한 이미지
![화면 캡처 2022-06-16 123642](https://user-images.githubusercontent.com/57117748/174043894-6f63e52a-9569-46df-97a6-928d0d82fc25.png)


# Docker Image Share
- docker push 를 통해 docker hub에 Image 를 공유 가능
* 업로드 시 지켜야 할 주의 사항
	1. 저장소(레포지토리)에 대한 인증
	2. 이미지 이름 지정

0. docker hub login or signin
- hub.docker.com에 회원가입

1. 도커 로그인
# docker login
```
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: cdm5212
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```
2. tag 생성
- docker tag 로 이미지 업로드를 위해 '아이디/repo이름:설명' 로 원본 이미지를 재정의 후 복사
- '아이디/repo이름:설명' 으로 해야지 자신의 repo에 업로드 가능
```
# docker image tag alpine:latest cdm5212/goorm-k8s-docker:alpine

# docker image ls
REPOSITORY                 TAG       IMAGE ID       CREATED        SIZE
alpine                     custom    71cae665f421   2 hours ago    5.53MB
httpd                      latest    acd59370d8fb   7 days ago     144MB
cdm5212/goorm-k8s-docker   alpine    e66264b98777   3 weeks ago    5.53MB
alpine                     latest    e66264b98777   3 weeks ago    5.53MB
mysql                      5.6       dd3b2a5dcb48   5 months ago   303MB
centos                     7         eeb6ee3f44bd   9 months ago   204MB
```

3. docker image upload
```
# docker push cdm5212/goorm-k8s-docker:alpine
The push refers to repository [docker.io/cdm5212/goorm-k8s-docker]
24302eb7d908: Mounted from library/alpine
alpine: digest: sha256:4ff3ca91275773af45cb4b0834e12b7eb47d1c18f770a0b151381cd227f4c253
size: 528
```

## 실습 
- centos7 기반의 Apache Webserver 구성
- Index.html 파일 호스트에서 복사해 넣기
- 포트는 8080
- img_user 사용자 생성
- /var/www/html 여기에 볼륨연결

1. Dockefile 만들기
- 각 이미지마다 각각의 디렉토리를 만들고 그 안에 도커 파일을 만드는것을 권장
```
# vi ~/Docker_imgs/test/Dockerfile
```
```
FROM    centos:7

RUN     yum -y install httpd
COPY    index.html /var/www/html/index.html

VOLUME  /var/www/html
EXPOSE  8080:80
ENV     NAME=img_user

RUN     useradd img_user
USER    img_user
WORKDIR /tmp
RUN    touch fileA

USER    root
CMD     httpd -D FOREGROUND
```
* USER를 img_user를 만들고 기본 사용자로 지정하면서 권한 거부가 생김으로 마지막에 root 사용자로 변경 

2. index.html 설정
```
# echo 'Im host' > ./index.html
```

3. image bulid
```
# docker build -t test:test .	// Dockerfile 이 있는 경로에서 실행
```

4. image 작업 동작 확인
```
# docker image ls -a 	// 이미지 생성 확인

# docker image inspect test:test	// 몇번의 동작이 있었나 확인(RUN, COPY, VOLUME, RUN, RUN)

[root@localhost Docker_img]# docker image inspect test:test | grep -A5 'Layers'
            "Layers": [
                "sha256:174f5685490326fc0a1c0f5570b8663732189b327007e47ff13d2ca59673db02",
                "sha256:e54320f5d01fb3ae88065a630889150c64d85dc5e77c8dd74c0b00ba2dfd410a",
                "sha256:0341068dd8ae0028e1cab6e0cc37986877bc0521b4f89ca3b3ceddb54674b6b7",
                "sha256:df491ea30ae2403d42c6a70ea7f35b0fb163cd00a7e8b51ba8f45328b448cb44",
                "sha256:d269bd31f4d3d376af877fb4b8d023577efbf8d16198f66e2b716c08ad64c271"
```

5. docker run 으로 컨테이너 생성 및 실행
```
# docker run -itd --network host --name test_web test:test
330df659931a62b4f9a683434f9f16da0622a91394bcd265b8bdbb9a7bc483b5

# curl localhost
Im host
```

6. docker container inspect 으로 확인
```
# docker inspcet test_web
[
    {
        "Id": "330df659931a62b4f9a683434f9f16da0622a91394bcd265b8bdbb9a7bc483b5",
        "Created": "2022-06-16T08:21:04.284411068Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "httpd -D FOREGROUND"
        ],
.
.
.
            {
                "Type": "volume",
                "Name": "9952e86d22ad3a3ca84cd1a7d3b927280fb32fcb996897121ed6fa1a36a8eed2",
                "Source": "/var/lib/docker/volumes/9952e86d22ad3a3ca84cd1a7d3b927280fb32fcb996897121ed6fa1a36a8eed2/_data",
                "Destination": "/var/www/html",
                "Driver": "local",
                "Mode": "",
                "RW": true,
                "Propagation": ""
            }
.
.
.
            "Cmd": [
                "/bin/sh",
                "-c",
                "httpd -D FOREGROUND"
            ],
```


## 키워드
- Dockerfile : 컨테이너를 실행할 이미지를 직접 만들 때 사용한다
- multistage build : 제품 이미지에는 필요 없는 환경을 제거할 수 있도록 단계를 나누어 기반 이미지를 만드는 방법
- FROM : 기반 이미지를 지정하는 항목
- CMD : 컨테이너를 실행할 때 동작할 항목. ENTRYPOINT 항목과 동시에 사용되면 그곳의 인자값으로 전달된다
- ENTRYPOINT : 컨테이너를 실행할 때 동작할 항목. CMD 항목과 동시에 사용되면 CMD가 인자값으로 전달되며 run 구문 사용시 이미지 뒤 명령/작업을 입력한다면 그값이 인자값으로 전달
