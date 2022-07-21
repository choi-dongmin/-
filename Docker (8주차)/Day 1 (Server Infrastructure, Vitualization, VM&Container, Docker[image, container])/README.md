# 도커

- Container 타입의 서버의 자동화 관리를 하기 위한 대표적인 도구 


## 도커를 이해하기 위한 기본 용어
1. 서버 : 서비스를 제공하는 시스템(컴퓨터)
2. 인프라 : 서버를 구성하는 환경. (서버 시스템, 네트워크, 스토리지)

- 서버의 구성 방식
	* 실제 시스템을 운용하는 형태(온프레미스)
	* 실제 운영체제에 직접 구성하는 방식
	* 가상머신을 이용하는 형태 
		- 서버 가상화
			- 가상머신 
		- 클라우드
			1. 퍼블릭
			2. 프라이빗(오픈스택 : IaaS 의 형태의 플랫폼을 쉽게 구축할 수 있게 서비스)
	* 컨테이너를 이용하는 형태
		- 컨테이너
			- K8s
* 과거에는 "폭포수 모델"을 기반으로 분석, 설계, 구현, 시험 및 운영의 단계를 차례로 거쳤지만 현재에는 클라우드를 통해 더 편리하게 작업 진행 가능

- 하드웨어 : 네트워크 장비, 스토리지 장비 등
- OS : 운영체제
- 미들웨어 : 운영체제 위에서 설치하는 소프트웨어

## Kernel vs Shall
1. Kernel : Shell 로 부터 전달된 명령어를 통해 그 명령을 하드웨어단에서 제어/운영을 하는 핵심
2. Shell : 사용자에게 명령어를 받아 kernel 로 전달하기 위한 입력 장치

# 가상화
- 물리적인 시스템 및 장치를 논리적인 개념으로 재구성 하는것.
- 논리적으로 여러 장치를 하나로 통합(LVM..) 혹은 하나의 장치를 여러개의 장치로 분할(가상머신)
* 에뮬레이션 : 형태가 다른 장치를 나의 장치에 맞게 maping 시키는 작업
- 가상화의 종류
	1. 시스템
	2. 스토리지
		- LVM, S3(Obj Storage), SDS(S/W Define Storage)
	3. 네트워크
		- VPN, vLAN(SDN : S/W : Defube Network, NFV : Network function virtualization)

## 시스템의 가상화

![VM vs Container](https://raw.githubusercontent.com/collabnix/dockerlabs/master/beginners/docker/images/vm-docker5.png)

1. 가상머신을 통한 가상화
	- 하이퍼바이저를 통해 운용하는 가상머신
	- 가상의 운영체제가 각각의 가상머신마다 다르게 존재
	- 리소스 낭비 및 오버헤드의 증가

2. 컨테이너 
	- 컨테이너 런타임(엔진)으로 운용하는 컨테이너
	- 하나의 운영체제 위에 각각의 어플리케이션마다 분리
	- 리소스 낭비 및 오버헤드의 감소
	- 배포 속도의 증가
```
90년대에 들어서 서버를 운용하는데 하드웨어가 App 보다 월등히 좋아지는 현상으로 리소스 낭비가 발생한다. 

그래서 다수의 App을 이용해 리소스의 낭비를 줄여보려 했으나 서로 충돌이 발생해 안정성이 떨어짐

그리하여 시스템을 각 App 마다 논리적으로 분할하기 시작(가상머신), 이러한 VM을 네트워크로 제공하는것이 클라우드 서비스

그러나 문제는 같은 운영체제를 사용하는 경우가 대다수인데 VM은 각각의 가상머신마다 각각의 운영체제를 설치하는 비효율 발생

결과적으로 컨테이너를 활용해 하나의 운영체제 위에서 엔진을 활용하여 App을 관리하기 시작
```

- 호스팅형 가상화 : OS를 먼저 시스템에 설치하고 하이퍼 바이저를 설치해 가상머신을 구동
- 하이퍼바이저 가상화 : OS를 설치할 필요없이 OS 역할을 할 수 있는 하이퍼 바이저를 설치

* 즉, 내가 사용하는 OS가 아닌 다른 OS에서 서버를 운용하고 싶다면 VM 아니라면 Container
* VM 과 Container 모두 하드웨어를 공유하지만 논리적으로 격리하는것처럼 설정

# Docker
- 리눅스 컨테이너를 관리하는 엔진(컨테이너 런타임)
- 컨테이너 런타임은 컨테이너를 생성/관리 역할을 수행
- App 을 사용함에 있어 Img를 사용해 Container 만듦
- 리소스에 대한 격리(프로세스, 파일시스템..)

## Image & Container
1. Image : App을 실행하는데 있어서 최소로 필요한 파일의 집합
2. Container : Image를 실행한 상태

- 실행파일 : 프로세스 = 이미지 : 컨테이너

## Docker의 기능
1. 이미지의 생성
2. 이미지의 공유
3. 컨테이너 실행

## Docker's Objects
1. Image
2. Container
3. Network
4. Volume

## Docker의 기반기술
- namespace(이름공간) : 프로세스의 고유한 정보를 기반으로 테이블을 구성해 각각 리소스를 격리하는 기술. 데이터에 이름을 붙여 격리를 하고 H/W에서 참조가 가능
- Cgroup(Control Group):리소스 관리를 위한 그룹
- 가상의 브릿지/ vNIC : 컨테이너에서 사용할 네트워크 환경을 제공하기 위한 기술
- 파일 시스템 : 컨테이너에서 사용할 데이터 관리를 위해

## Docker의 명령어 
- docker 관리대상[오브젝트] 명령어의 구성방식
```
docker 	image 		pull
		container
		network
		volume
```

## 도커 사용 가능한 사용자
1. root
2. sudo 명령어가 가능한 사용자
3. docker 그룹의 구성원

# Docker Install

> CentOS
- Setting Reposittory
```
$  sudo yum install -y yum-utils

$ sudo yum-config-manager \
> --add-repo \
> https://download.docker.com/linux/centos/docker-ce.repo
```

- Install Docker Engine
```
$ sudo yum install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- Docker start and enable
```
$ sudo systemctl --now enable docker
```

- Test 
```
$ sudo docker run hello-world
```

- add user
```
$ sudo usermod -aG docker [username]
```

- re-evaluated
```
newgrp docker 
```


> Ubuntu
- remove old version
```
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

- Install apt pkg
```
$ sudo apt-get update

$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

- GPG Key Add
```
$ sudo mkdir -p /etc/apt/keyrings
 
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

- Repository Setting
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

- Install Dokcer
```
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

- Docker start and enable
```
$ sudo systemctl --now enable docker
```

- Test 
```
$ sudo docker run hello-world
```

- add user
```
$ sudo usermod -aG docker [username]
```

- re-evaluated
```
newgrp docker 
```

# docker image commands
- 이미지 이름 (이미지명:태그) : 이미지명은 일반적으로 App의 종류, 태그는 버전의 경우가 다수
* Tag를 따로 지정하지 않는다면 Latest로 지정하기 때문에 특정 태그를 지정하는것을 권장 

- docker image 다운로드 (httpd)
```
$ docker image pull httpd

Using default tag: latest
latest: Pulling from library/httpd
Digest: sha256:63502bc48a2e239db26efed3e4cbb97e89a85e1032ebd9ea49269d8a28db5d12
Status: Image is up to date for httpd:latest
docker.io/library/httpd:latest
```

- 다운로드한 image의 목록을 확인 (주로 -a 옵션과 같이 사용)
```
$ docker image ls

REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
httpd         latest    acd59370d8fb   4 days ago     144MB
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB

```

- 로컬 이미지의 상세정보 확인
```
$ docker image inspect acd

[
    {
        "Id": "sha256:acd59370d8fb027ca05939a9e4d2e412f11b6a1581dd4e743f357eacadf55e5e",
        "RepoTags": [
            "httpd:latest"
        ],
        "RepoDigests": [
            "httpd@sha256:63502bc48a2e239db26efed3e4cbb97e89a85e1032ebd9ea49269d8a28db5d12"
        ],
.
.
.
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:ad6562704f3759fb50f0d3de5f80a38f65a85e709b77fd24491253990f30b6be",
                "sha256:4e2b986e3da36be7d4a0475d6d849b0a2be624edc885bb58744eb720d5ee3986",
                "sha256:4a48959ac654663898efa39ac7b9623c548aca1b32f1746617034f9ef372ea26",
                "sha256:857d0bc0663ff416f892a241d9822fe28a8d46560064f38d9aea2fc80e177e95",
                "sha256:2db1ee0a0f4faa784abc3e24d35cbef4d95042db7a0fdc6615da3753fee301c0"
            ]
        },
        "Metadata": {
            "LastTagTime": "0001-01-01T00:00:00Z"
        }
    }
]
```

- 로컬 이미지를 삭제
```
$ docker image rm acd

Untagged: httpd:latest
Untagged: httpd@sha256:63502bc48a2e239db26efed3e4cbb97e89a85e1032ebd9ea49269d8a28db5d12
Deleted: sha256:acd59370d8fb027ca05939a9e4d2e412f11b6a1581dd4e743f357eacadf55e5e
Deleted: sha256:69512b7faaf8ba7d52eee6ed511192aa2a5e7871562d1d691c3d428705e173ec
Deleted: sha256:f60cdf12473ac01aa6e574e59f50afe9a2d4b341682d5e7fb08b6cc892955bc6
Deleted: sha256:b6ac17bbf3c4d8d63c1a085e9affa7a6c3f6e765f821837ff3d3e7fc0c0706e2
Deleted: sha256:3439b4fb9b4289be9427c14f2840d97e3573dc8d609584d9ceb2ad8bbd3ae9dc
Deleted: sha256:ad6562704f3759fb50f0d3de5f80a38f65a85e709b77fd24491253990f30b6be

$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB
```

- 그 외 명령어들
```
$ docker image history : 이미지의 history 를 정렬
$ docker image prune : 사용하지 않는 이미지 제거
$ docker image push : 내가 가진 Image를 업로드(docker login 이후)
```


# docker container commands
![화면 캡처 2022-06-13 153008](https://user-images.githubusercontent.com/57117748/173330419-4cc38d41-392a-438d-88c1-c5825f802dcb.png)

- docker container create : 컨테이너 생성
	* -it 옵션으로 입출력이 필요한(쉘을 실행하는) 컨테이너에게 사용한다 !중요!
	* --rm : 종료시 자동삭제 / --name : 컨테이너 이름 설정

- docker container start : 생성한 컨테이너를 실행
	* 입출력이 필요한 컨테이너에게 -ai 를 이용해 연결한다.(create에서 -it 필수)

- docker container run : create + start
	* -it : 입출력이 필요한 경우 (자동연결)
	* -d : 연결이 불필요한경우 연결하지 않음
	* --rm, --name
	* -p : 포트포워딩 설정 (-p 8080:80 nginx)
	* --restart [--name, cid] : 해당 컨테이너를 실행이 안될시 재시도
	* --add-host : container 생성 시 네트워크 호스트 추가 설정
	* --dns : container 생성 시 DNS 설정
	* --expose : 포트번호를 직접 연결 할 수 있다
	* --hostname : 호스트이름 지정

- docker container ls : 도커의 컨테이너 목록확인
	* docker container ls = docker ps

- 컨테이너 생성 및 실행 시 주의사항
	* 이미지의 종류(실행 프로세스)에 따라서 옵션이 다르게 적용
	* 쉘을 실행하는 이미지의 경우 create : -it / start : -ai / run : -it  not -d 
	* 서비스(Demon)을 실행하는 이미지의 경우 create: x / start : x / run : -d
	* 일회성(출력만) 실행하는 이미지의 경우 create: x / start -ai / run : x

- docker container logs [--name, cid]
	* 해당 컨테이너의 로그를 출력

- docker container attach [name]
	* shell 을 사용하는 시작 되어있는 container shell에 연결

- docker container --add-host : container 생성 시 네트워크 호스트 추가 설정

- docker container --dns : container 생성 시 DNS 설정



## Tips
- 모든 컨테이너를 삭제(실무에서는 절대 금지)
```
$ docker container rm -f $(docker ps -aq)`
```

- 쉘을 실행한 컨테이너에서 종료하지 않고 빠져나오기
```
Ctrl + p + q
```

# 실습
1. centos:7 / alpine / httpd:latest / mysql:5.6
```
$ docker image pull centos:7
$ docker image pull alpine
$ docker image pull httpd:latest
$ docker image pull mysql:5.6

$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
httpd         latest    acd59370d8fb   4 days ago     144MB
alpine        latest    e66264b98777   2 weeks ago    5.53MB
mysql         5.6       dd3b2a5dcb48   5 months ago   303MB
hello-world   latest    feb5d9fea6a5   8 months ago   13.3kB
centos        7         eeb6ee3f44bd   9 months ago   204MB
```

* inspect 을 사용해 CMD 섹션을 보면 쉘을 하는지 알 수 있다.


2. 컨테이너 실행 

- httpd 이미지로 web 이라는 이름으로 동작하는 컨테이너를 만들기 (create - > start)
```
$ docker container create --name web httpd
89fdb2d41376dc0f2b030ecb4e121f90a0b79046fa718c3605fbbfeae67c6fc1

$ docker container start web
web

$ docker ps
CONTAINER ID   IMAGE     COMMAND              CREATED          STATUS         PORTS     NAMES
89fdb2d41376   httpd     "httpd-foreground"   56 seconds ago   Up 7 seconds   80/tcp    web
```

- centos:7 이미지로 os 라는 이름으로 동작하는 컨테이너를 만들기 (create - > start)
```
# docker container create -it --name os centos:7
0c86e80c47983eed976ecd6ea98b81fe53a67609c1a1c16e9ec525b2f5d6a23e

# docker container start -ai os
[root@0c86e80c4798 /]#
```
- 위 두 동작을 run을 이용하여 web2,os2 컨테이너 만들기
```
# docker container run -d --name web2 httpd
4158ff877d0475c306fc75ad3122755ddbd0720749ad17584f25d1138aacfb89

# docker ps
CONTAINER ID   IMAGE     COMMAND              CREATED          STATUS          PORTS     NAMES
4158ff877d04   httpd     "httpd-foreground"   10 seconds ago   Up 9 seconds    80/tcp    web2
89fdb2d41376   httpd     "httpd-foreground"   19 minutes ago   Up 19 minutes   80/tcp    web

# docker container run -it --name os2 centos:7
[root@f06caf7679ce /]# 

[root@f06caf7679ce /]# [root@localhost ~]# docker container ps
CONTAINER ID   IMAGE      COMMAND              CREATED              STATUS              PORTS     NAMES
f06caf7679ce   centos:7   "/bin/bash"          About a minute ago   Up About a minute             os2
4158ff877d04   httpd      "httpd-foreground"   2 minutes ago        Up 2 minutes        80/tcp    web2
89fdb2d41376   httpd      "httpd-foreground"   21 minutes ago       Up 21 minutes       80/tcp    web

# docker container stop os2
os2

# docker container ps
CONTAINER ID   IMAGE     COMMAND              CREATED          STATUS          PORTS     NAMES
4158ff877d04   httpd     "httpd-foreground"   2 minutes ago    Up 2 minutes    80/tcp    web2
89fdb2d41376   httpd     "httpd-foreground"   22 minutes ago   Up 21 minutes   80/tcp    web
```

- alpine 이미지로 a1 이라는 이름의 컨테이너를 만들기
```
# docker container run -it --name a1 alpine
/ # 
```

- alpine conatiner inspect alpine 을 통해 CMD(명령 경로) 섹션을 확인
```
# docker container inspect a1
.
.
            "Cmd": [
                "/bin/sh"
            ],
.
.
```

- hello-world 이미지로 컨테이너 만들기 단, 실행 후 삭제
```
# docker container run -it --rm hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/


# docker ps -a
CONTAINER ID   IMAGE      COMMAND              CREATED          STATUS                            PORTS     NAMES
36279cddb05e   alpine     "/bin/sh"            2 minutes ago    Exited (137) About a minute ago             a1
f06caf7679ce   centos:7   "/bin/bash"          7 minutes ago    Exited (137) 5 minutes ago                  os2
4158ff877d04   httpd      "httpd-foreground"   8 minutes ago    Up 8 minutes                      80/tcp    web2
0c86e80c4798   centos:7   "/bin/bash"          24 minutes ago   Exited (0) 9 minutes ago                    os
89fdb2d41376   httpd      "httpd-foreground"   28 minutes ago   Up 27 minutes                     80/tcp    web
```



3. 컨테이너 종료
- 동작 중인 컨테이너들을 모두 종료
```
# docker container stop $(docker ps -aq)
36279cddb05e
f06caf7679ce
4158ff877d04
0c86e80c4798
89fdb2d41376

# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

- 종료된 컨테이너들을 모두 삭제
```
# docker container rm -f $(docker ps -aq)
36279cddb05e
f06caf7679ce
4158ff877d04
0c86e80c4798
89fdb2d41376
```

- 로컬 이미지들 중 hello-world 이미지를 삭제
```
# docker image rm 
alpine:latest       centos:7            hello-world:latest  httpd:latest        mysql:5.6

# docker image rm hello-world:latest
Untagged: hello-world:latest
Untagged: hello-world@sha256:80f31da1ac7b312ba29d65080fddf797dd76acfb870e677f390d5acba9741b17
Deleted: sha256:feb5d9fea6a5e9606aa995e879d862b825965ba48de054caab5ef356dc6b3412
Deleted: sha256:e07ee1baac5fae6a26f30cabfe54a36d3402f96afda318fe0a96cec4ca393359

# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
httpd        latest    acd59370d8fb   4 days ago     144MB
alpine       latest    e66264b98777   2 weeks ago    5.53MB
mysql        5.6       dd3b2a5dcb48   5 months ago   303MB
centos       7         eeb6ee3f44bd   9 months ago   204MB
```

## 키워드

- 가상화 : 물리적인 시스템 및 장치를 논리적인 개념으로 재구성 하는것.

- 컨테이너 : 시스템의 가상화 방식 중 하나로 사용자의 OS와 같은 OS를 공유하면서 어플리케이션을 통해 격리시켜주는 방식으로 오버헤드와 리소스가 감소한다.

- 가상머신 : 시스템의 가상화 방식 중 하나로 애플리케이션을 격리시킨다 그러나 운영체제도 같이 격리적으로 설치하면서 컨테이너보다 무겁고 리소스가 더 크다.

- 도커 : 컨테이너 런타임 종류 중 하나이다로 리눅스 버전의 컨테이너들을 관리 한다. 리눅스가 같은 기본 기능들을 활용한다.(namespace, 가상브릿지/vNIC, cgroup, 계층형 파일 시스템)

- Image : 컨테이너를 실행하기 위한 최소의 실행파일들의 아카이브

- Container : App을 기준으로 분리하는 하나의 가상화 시스템   
