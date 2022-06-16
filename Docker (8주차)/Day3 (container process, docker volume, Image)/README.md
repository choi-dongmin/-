# Container Process
- 기본적으로 컨테이너는 하나의 프로세스가 동작
- 기본 프로세스는 이미지에서 설정
- 컨테이너 안의 프로세스가 종료되면 컨테이너도 종료

## Container Commands
- docker container run
```
# docker container run ....[이미지]	// 이미지에서 정한 프로세스를 실행
# docker container run ....[이미지] [명령어]	//이미지에서 정한 프로세스를 실행하지 않고 지정한 다른 프로세스를 실행
```

- docker container exec
```
# docker container exec .... [컨테이너] [명령어]	// 동작 중인 컨테이너에 추가 프로세스를 실행, -it 옵션으로 쉘을 실행가능
```
![화면 캡처 2022-06-15 113037](https://user-images.githubusercontent.com/57117748/173794261-5cc49b38-88ac-48b0-8e01-87ef49f7f70a.png)
![화면 캡처 2022-06-15 113617](https://user-images.githubusercontent.com/57117748/173794289-014afff5-6f68-4144-8e58-5be353242754.png)

- docker container attach
```
# docker container attach [컨테이너]	// 동작 중인 컨테이너에 연결, 연결 시 입출력 작업은 쉘이 동작 중인 경우만 가능
```

- docker container top 
```
# docker container top [컨테이너]	// 컨테이너 안에서 실행 중인 프로세스 확인 (ps)
```

- docker container rename
```
# docker container remname [컨테이너] [새로운 컨테이너 명]		// rename
```

- docker container kill
```
# docker container kill [컨테이너]	// 강제종료
```

- docker container cp
```
# docker container cp [파일명] [컨테이너 파일 경로]	// 호스트에 있는 파일 컨테이너에게 복사 혹은 그 반대
```
![화면 캡처 2022-06-15 121443](https://user-images.githubusercontent.com/57117748/173796428-db1c3559-0cf5-4270-9f4e-16cfc2f81215.png)

- docker container diff
```
# docker container diff [컨테이너]	// 컨테이너의 초기 상태와 현재를 비교
```
![화면 캡처 2022-06-15 121952](https://user-images.githubusercontent.com/57117748/173796449-587b2b23-9cb9-40ab-96e7-8961c98db343.png)

# Docker Volume
- 컨테이너의 데이터는 영구저장 되지 않음
- 컨테이너의 데이터를 저장하는 방식 'Volume'
- docker volume create 명령으로 사용
- docker volume ls 를 통해 확인 가능
```
docker volume create [볼륨명]
```
## 볼륨과 컨테이너의 연결 방식
1. 도커 볼륨 연결
- docket container run 시에 -v 옵션으로 사용
```
# docket container run -v [볼륨명]:[컨테이너 데이터 저장 경로]
```

2.기존의 디렉토리 이용하기
- 사용자의 시스템상의 디렉토리 경로를 사용
- 경로를 지정함으로 디렉토리 연결 
- 경로에 파일 없을시 생성
```
# docket container run -v [디렉토리 경로]:[컨테이너 데이터 저장 경로]
```
3. 이미지에서 설정해 둔 경우 자동 연결


## 다중 볼륨 연결
1. 한 컨테이너에서 여러 볼륨을 연결
```
docker -v /on/my/host/1:/on/the/container/1 \
       -v /on/my/host/2:/on/the/container/2
```
2. 이미지에서 지정한 볼륨 연결
- 이미지에서 지정한 볼륨과 연결 가능
- 해당 컨테이너의 다른 볼륨도 지정 가능
```
docker -v ~/mysql_vol:/var/lib/mysql
```
# Docker Image
- 컨테이너를 실행하기 위한 파일(데이터의 집합)

## 이미지 생성
- 현재 있는 컨테이너를 설정, 포드 등 그대로 이미지화
- 파일시스템 구조 변경 가능

1. docker container commit 명령
- docker container commit [옵션][컨테이너명][새로운 이미지 명] 사용
- 백업의 용도
```
# docker container commit c1 newimg:latest
sha256:d0808b8877a382d4932fd4a499d503904d93655930b46e8cd38bbc9465fb5880

# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
newimg       latest    d0808b8877a3   8 seconds ago   204MB
httpd        latest    acd59370d8fb   6 days ago      144MB
alpine       latest    e66264b98777   3 weeks ago     5.53MB
mysql        5.6       dd3b2a5dcb48   5 months ago    303MB
centos       7         eeb6ee3f44bd   9 months ago    204MB
```

2. export / import
```
# docker container export c1 -o img.tar	// 컨테이너 이미지를 아카이브 시키기
# docker image import img.tar img:latest	// 아카이브된 이미지 컨테이너 만들기
```

3. save / load
- 이미지들을 아카이빙(save)하고 다시 로드(load)
```
# docker image ls
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
httpd        latest    acd59370d8fb   6 days ago     144MB
alpine       latest    e66264b98777   3 weeks ago    5.53MB
mysql        5.6       dd3b2a5dcb48   5 months ago   303MB
centos       7         eeb6ee3f44bd   9 months ago   204MB

docker image save httpd:latest alpine:latest mysql:5.6 centos:7 -o save.tar	// save

# docker image rm -f $(docker image ls -aq)
Untagged: httpd:latest
Untagged: httpd@sha256:63502bc48a2e239db26efed3e4cbb97e89a85e1032ebd9ea49269d8a28db5d12
Deleted: sha256:acd59370d8fb027ca05939a9e4d2e412f11b6a1581dd4e743f357eacadf55e5e
Deleted: sha256:69512b7faaf8ba7d52eee6ed511192aa2a5e7871562d1d691c3d428705e173ec
.
.
.

# docker image ls -a
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

# docker image load -i save.tar	// save 한 파일을 다시 로드

# docker image ls -a
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
httpd        latest    acd59370d8fb   6 days ago     144MB
alpine       latest    e66264b98777   3 weeks ago    5.53MB
mysql        5.6       dd3b2a5dcb48   5 months ago   303MB
centos       7         eeb6ee3f44bd   9 months ago   204MB
```

## 실습 
1. 컨테이너 실행 관리
- 컨테이너 생성 및 실행
```
# docker container run -d --name h1 httpd:latest	// h2 httpd 

# docker container run -it --name h2 httpd:latest /bin/bash	// h2 httpd baashShell
```

2. 컨테이너 파일 수정
- h2 컨테이너에 연결
```
# docker container attach h2 // shell 이 있기 때문에 attach
# echo 'hello world' > /usr/local/apache2/htdoc/index.html 	// vi 가 없기때문에 echo

// 만약 exit 한다면 쉘이 실행중이던 프로세스이기 때문에 컨테이너 자체가 종료 된다
```

- h1 컨테이너에 추가 실행
```
# docker container exec -it h1 bash
# echo 'hello world' > /var/html/www/index.html 	// vi 가 없기때문에 echo

// exit 로 나와도 추가 실행이기 때문에 상관 없음
```

- 호스트에서 index.html 파일을 작성하고 h1에 복사
```
# echo 'hello there' > ./index.html 	// 호스트 시스템에 index.html 파일 만들기

# docker container cp index.html h1:/usr/local/apache2/htdoc/index.html 	// 호스트 시스템 파일 컨테이너에 복사

# docker container exec h1 cat h1:/usr/local/apache2/htdoc/index.html
hello there
```

3. 컨테이너 프로세스 관리
- h1 컨테이너에 쉘을 실행하고 Ctrl + p + q 로 탈출
```
[root@localhost ~]# docker container exec -it h1 bash
root@ceb792b065f8:/usr/local/apache2# read escape sequence
```

- h1 컨테이너의 프로세스 정보 확인
```
[root@localhost ~]# docker container top h1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                2643                2624                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2670                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2671                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2672                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
root                3359                2624                0                   11:03               pts/0               00:00:00            bash 
```

- 불필요한 프로세스(쉘)를 강제종료
```
[root@localhost ~]# kill -9 3359
[root@localhost ~]# docker container top h1
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                2643                2624                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2670                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2671                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
33                  2672                2643                0                   10:50               ?                   00:00:00            httpd -DFOREGROUND
```

4. 컨테이너 생성 및 이미지 생성
- 이미지 백업

- 컨테이너 생성 및 이미지 생성

- 이미지 복원(load -i )

5. 볼륨 생성
- 볼륨 생성
```
# docker volume create contents
```

- 컨테이너 생성 시 content 볼륨을 /mnt 디렉토리에 연결
```
# docker container run -it -v contents:/mnt --name creator centos:7 
```

- 호스트의 사용자 홈디렉토리에서 index.html 파일 작성 해당 파일을 2번 컨테이너의 mnt 디렉토리로 복사
```
# echo "I'm host" > ./index

# docker container cp index.html creator:/mnt/	// 호스트 index.html 복사

# docker container exec creator cat /mnt/index.html

# ls /var/lib/docker/volumes/contents/_data/	// 볼륨의 데이터 저장경로
```

- 컨테이너 생성 시 컨텐츠 디렉토리와(/usr/local/apache2/htdoc/index.html) content 볼륨 연결
```
# docker container run --itd --name web_volumes -v contents:/usr/local/apache2/htdocs/ --network host httpd:latest 		// creator /mnt = contents volume = web_volumes data path

# curl localhost
```

## 키워드 
- Container Process : 기본적으로 컨테이너는 하나의 프로세스 헤당 프로세스는 이미지에서 설정하며 프로세스 종료시 컨테이도 종료된다.

- docker container exec : 현재 동작중인 프로세스에 추가 명령을 실행한다(보통 shell이 없는 프로세스에서)

- docker container attach : 현재 동작중인 프로세스에 연결한다(보통 shell이 있는 프로세스에서)

- docker container cp : 호스트에 있는 파일 컨테이너에게 복사 혹은 그 반대

- Docker Volume : 컨테이너의 데이터는 영구 저장되지 않는다, 저장 하기 위한 컨테이너 데이터 저장 공간이다. 컨테이너 디렉토리와 연결 하고 또는 호스트 컴퓨터의 디렉토리에 연결한다.

- image backup : 컨테이너를 commit 명령어로 이미지화 시키고 save 명령어로 아카이빙해 백업한다 백업한 아카이빙 파일은 load로 불러준다.

