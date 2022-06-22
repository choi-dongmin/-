# Docker, Docker Compose, K8s

1. Dokcer : 하나의 컨테이너를 하나의 호스트에서 관리

2. Docker Compose : 단수의 호스트가 다수의 컨테이너 관리

3. K8s : 다수의 호스트가 다수의 컨테이너를 관리

# 도커에서 이미지의 이용 방식
1. Dockerfile 으로 이미지 생성
2. Docker Hub 등 저장소에 업로드
3. 저장소에서 이미지 다운로드
4. 컨테이너를 사용해 실행

## 업로드 한 이미지 수정
1. 업로드된 이미지를 다운로드
2. 해당 이미지를 이용해 Dockfile 등으로 수정
3. 다시 업로드

* 편의를 위해 자동화 된 빌드 방식을 활용 

# Docker Hub
- Docker Image 의 공유를 위한 커뮤니티 저장소
- 회원가입 후 진행

```
docker push ID/REPO:TAG
```
* Docker Hub 사용 
![Docker Share](https://github.com/choi-dongmin/K-digital_k8s/tree/main/Docker%20(8%EC%A3%BC%EC%B0%A8)/Day4%20(Dockerfile%2C%20Docker%20image%2C%20Dockuer%20hub%2C%20multistage%20build)#docker-image-share)


# Docker registry
- Private 환경의 container 공유 저장소
- 사내 혹은 개인적으로 만든 이미지를 올리고 관리
- registry 이미지를 사용한 구축
- Registry는 컨테이너이므로, 올리는 이미지들을 유지하려면 볼륨 연동 필요
- 5000번 포트 사용
```
docker push IP주소:PORT/IMAGENAME
```

## Docker registry 실습
1. Dockerfile 작성
- centos:7 이미지 사용
- container_user 라는 이름의 사용자 생성
- 해당 사용자로 접속 및 작업
- 실행프로세스 /bin/bash
```
```
FROM    centos:7

RUN     useradd container_user
USER    container_user
CMD     /bin/bash
```
```

2. 이미지 빌드 및 확인
```
# docker build -t centos:registry_practice .

# docker run -it --name registry centos:registry_practice
9dc429accbf305604b15940e5767cb32dfbcb967fcc714b2ea328f045bc1c6ac
[container_user@9dc429accbf3 /]$
```

3. 도커 레지스트리 구성
- registry 이미지 다운로드
```
# docker pull registry
Using default tag: latest
latest: Pulling from library/registry
2408cc74d12b: Pull complete
ea60b727a1ce: Pull complete
c87369050336: Pull complete
e69d20d3dd20: Pull complete
fc30d7061437: Pull complete
Digest: sha256:bedef0f1d248508fe0a16d2cacea1d2e68e899b2220e2258f1b604e1f327d475
Status: Downloaded newer image for registry:latest
````

- registry 이미지로 컨테이너 시작
```
# docker run -d --name registry5000 -p 5000:5000 registry:latest
```

4. 태그 수정 및 업로드
- 2번에서 만든 이미지 이름 수정 (tag)
```
# docker tag centos:registry_practice localhost:5000/registry_practice
```

- registry에 업로드 
```
# docker push localhost:5000/registry_practice
```

5. 로컬 이미지 삭제 및 다운로드 확인
- 현재 이미지 삭제 (rm)
```
# docker image stop localhost:5000/registry_practice:latest
# docker image rm localhost:5000/registry_practice:latest
```

- 도커 레지스트리에서 이미지 다운로드 (pull)
```
# docker pull localhost:5000/registry_practice
```

- 컨테이너로 실행 및 확인
```
docker run -it --name registry_after_pull localhost:5000/registry_practice
[container_user@c58b3d7a58f7 /]$
```

# Docker Compose
- 하나의 호스트가 다수의 컨테이너를 생성 관리하는 방식
- Docker Engine, Docker CLI 필요
- YAML 언어를 사용 (Docker-compose.yaml : 관리할 컨테이너 목록 및 서비스 항목에서 컨테이너 특성 나열 )

## Docker Compose install
![docs.docker.com](https://docs.docker.com/compose/install/compose-plugin/)

1. Compose CLI plugin download
```
$ DOCKER_CONFIG=${DOCKER_CONFIG:-$HOME/.docker}
$ mkdir -p $DOCKER_CONFIG/cli-plugins
$ curl -SL https://github.com/docker/compose/releases/download/v2.6.0/docker-compose-linux-x86_64 -o $DOCKER_CONFIG/cli-plugins/docker-compose
```

2.  permissions to the binary
```
$ chmod a+x /usr/local/bin/docker-compose
```

3. Test the installation
```
$ docker compose version
```

## Run Container with Docker-compse.yaml
1. pull the base images
```
# docker pull mysql:5.6
# docker pull wordpress:latest 
```

2. Docker-compse.yaml setting
```
vi Docker-compse.yaml
``` 
```
version: "3.7"

services:
  mysql:
    image: mysql:5.6
    volumes:
      - ./4_db:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
      MYSQL_USER: manager
      MYSQL_PASSWORD: 1234


  wordpress:
    depends_on:
      - mysql
    image: wordpress:latest
    volumes:
      - ./4_wordpress:/var/www/html
    ports:
      - "8080:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_NAME: wordpress
      WORDPRESS_DB_USER: manager
      WORDPRESS_DB_PASSWORD: 1234
```

3. docker-compose up
```
# docker-compose up 
```

4. Check and Test
![화면 캡처 2022-06-18 014902](https://user-images.githubusercontent.com/57117748/174344049-52dcc58c-65f5-42a2-9087-1c603c0cce5d.png)

* If you wnat to be able to provide interactive input after attaching
```
services:
  example:
    # ...
    tty: true 
    stdin_open: true
```

# Harbor Regisrty
- Docker를 기업 환경에서 사용하려면 반드시 필요한 것이 Private Docker Registry.
- Docker에서 제공하는 기초적인 형태의 리포지터리(repository)는 관리 측면과 보안 측면에서 여러 가지 문제점이 있어서 개선
- Harbor는 CNCF(Cloud Native Computing Foundation)에 등록된 컨테이너 레지스트리 프로젝트 중 유일하게 'Incubating' 수준에 해당하는 오픈소스 프로젝트

## Harbor 설치
1. 설치 파일 다운로드
```
# wget https://github.com/goharbor/harbor/releases/download v2.5.1/harbor-offline-installer-v2.5.1.tgz
```

2. 압축 해제
```
# tar xfz harbor-offline-installer-v2.5.1.tgz
``` 


3. 설정파일 수정
- HTTPS를 사용하지 않기 때문에 HTTPS 제거(사용하려면 유지해야 한다.)
```
# cd harbor/
# cp harbor.yml.tmpl harbor.yml     // harbor.yml.tmpl 파일 복사 후 수정
# vim harbor.yml   // 호스트 이름 설정
                   // https 관련 설정 제거 혹은 주석처리
                   // Harbor admin 비밀번호 확인 및 수정
```
* 호스트 이름 설정
![화면 캡처 2022-06-20 013843](https://user-images.githubusercontent.com/57117748/174491459-6d0efdaf-5ab4-4100-b737-6e44ab29a5ed.png)

* HTTPS 관련 설정 수정
![화면 캡처 2022-06-20 013856](https://user-images.githubusercontent.com/57117748/174491537-edc0e3e2-8bb6-4258-a224-2c9150d13fa8.png)

* Harbor admin 비밀번호 확인 및 수정
![화면 캡처 2022-06-20 013910](https://user-images.githubusercontent.com/57117748/174491510-129798ca-5f59-4ca4-a3a2-5f95af35f185.png)

4. HTTP를 통해 하버에 연결(HTTPS X)

- 도커 서비스 설정 변경 및 재시작
```
# vim /etc/docker/daemon.json   //새로생성
```
```
{
"insecure-registries" : ["IP주소"]
}
```

- 도커 서비스 재시작
```
# systemctl restart docker
```

5. harbor 내의 설치파일 실행
```
# cd harbor
# ./install.sh
```
* 설치 및 Harbor 이미지 확인
![화면 캡처 2022-06-20 015501](https://user-images.githubusercontent.com/57117748/174491945-d2009554-2fdf-401b-9480-5e6ca0c8b9a8.png)

## Harbor Registry 이용하기
0. Harbor에 접속하기
- 설치 시 harbor.yml 에서 확인 및 수정한 admin Password 활용
![화면 캡처 2022-06-20 020200](https://user-images.githubusercontent.com/57117748/174492380-b02b74a3-1e51-4979-bee2-f372e0029e35.jpg)

1. 이미지 tag 로 업로드 형식에 맞춰 변환 
- IP주소/project명/image명:tag
```
# docker image tag alpine:latest xxx.xxx.xxx.xxx/goorm/image:tag
```

2. login harbor & 
```
# docker logout 
# docker login IP주소 -u admin -p Harbor12345   // harbor.yml Harbor Passwd 수정시 해당 PW 입력
```

3. 업로드
- push 로 업로드
```
docker push xxx.xxx.xxx.xxx/goorm/alpine:version_1
```
![화면 캡처 2022-06-20 023654](https://user-images.githubusercontent.com/57117748/174493547-b03b1534-fde0-4aba-96da-29a2935f8468.png)

4. 다운로드
```
docker pull xxx.xxx.xxx.xxx9/goorm/alpine:version_1
```

## 키워드 
- Docker : 1개의 호스트에서 1의 컨테이너를 관리하고 컨테이너는 기본적을 하나의 프로세스를 실행하는 플랫폼.
- Docker Compose : 1개의 호스트에서 다수 컨테이너를 관리할 수 있는 플랫폼
- K8s : 다수의 호스트(노드)에서 다수의 컨테이너를 실행하고 관리할 수 있는 플랫폼
- Regisrty : 개인 혹은 지정된 사용자들이 사용할 수 있는 개인 저장공유공간
- Harbor : CNCF(Cloud Native Computing Foundation)에 등록된 컨테이너 레지스트리 프로젝트 중 유일하게 'Incubating' 수준에 해당하는 오픈소스 프로젝트
