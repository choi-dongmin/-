# SFTP
- sshd's High Module
- ssh 를 통해 안전하게 ftp에 업로드 할 수 있도록 구현
- 키를 교환하는 암호화 진행하며 신원확인용 스트링을 교환한다
- ssh port : 22 를 통해 sftp 포트로 연결 

![SFTP 스택](https://blog.kakaocdn.net/dn/kyAiV/btqxiMQtgyw/6rECWyaYxyD9JhtocsyWt0/img.png)

# wordpress 설치 및 실행
- wordpress 웹 제작 프레임워크 설치 및 실행

1. php7 설치가 가능한 remi 저장소 사전에 구축
```
# yum -y install epel-release
# wget http://rpms.remirepo.net/enterprise/remi-release-7.rpm --no-check-certificate
# rpm -Uvh remi-release-7.rpm
```



2. php7 설치
```
# yum install -y httpd

# yum install -y yum-utils		//  yum-config-manager 설치
# yum-config-manager --enable remi73	// php7.3 설치가능한 remi repo 활성화
# yum install -y php
```
-yum-config-manager 설치
![화면 캡처 2022-05-19 181310](https://user-images.githubusercontent.com/57117748/169258490-a9510e08-f03f-4eb7-b20d-87285c296909.png)

-php7.3 설치가능한 remi repo 활성화
![화면 캡처 2022-05-19 182303](https://user-images.githubusercontent.com/57117748/169260977-a90c5ab5-4201-4ba1-a862-77b05da81850.png)

- yum install -y php
![화면 캡처 2022-05-19 182945](https://user-images.githubusercontent.com/57117748/169261598-dc4a159f-b195-4191-be81-4191413fa249.png)


3. http 연결 확인 및 실행, 방화벽 해제
```
# echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php

# systemctl restart httpd
# systemctl enable httpd

# firewall-cmd --permanent --add-service=http
# firewall-cmd --permanent --add-service=https
# firewall-cmd --reload

```

3. DB 구성
```
# yum intall -y mariadb mariadb-server

# systemctl restart mariadb
# systemctl enable mariadb

# mysql_secure_installation	// mysql 초기 설정값 빠른 세팅

# yum install -y php-mysql	// php와 mysql 연동 패키지 설치
# systemctl restart httpd
```

- mysql_secure_installation
![화면 캡처 2022-05-19 184453](https://user-images.githubusercontent.com/57117748/169264545-60d0ba4e-b898-46ee-b227-755ca6bff243.png)

- yum install -y php-mysql
![화면 캡처 2022-05-19 184757](https://user-images.githubusercontent.com/57117748/169265109-610994c5-60a9-4ef8-8af0-d35a193451e4.png)

3. DB 계정 생성
```
# mysql -u root -p

---------------------------------------------------
Maria SQL

> CREATE DATABASE [DB명];		// DB 생성
> CREATE USER [계정명]@[IP] IDENTIFIED '[PW]';	// 계정 생성
> GRANT ALL ON [DB명].* TO [계정명]@[IP];	// 해당 DB의 *(모든 테이블) 관리 권한부여 
> FLUSH PRIVILEGES; 	// 현재 사용중인 MySQL의 캐시를 지우고 새로운 설정을 적용

> EXIT
```
![화면 캡처 2022-05-19 190630](https://user-images.githubusercontent.com/57117748/169268925-06c07eb2-32fb-46ca-8411-352ee66122b5.png)
![화면 캡처 2022-05-19 190643](https://user-images.githubusercontent.com/57117748/169268930-4491df8b-6692-495f-9f13-c442f7354126.png)
![화면 캡처 2022-05-19 190656](https://user-images.githubusercontent.com/57117748/169268939-7cbe1776-5789-454e-9c02-339ddf44f78b.png)
![화면 캡처 2022-05-19 190706](https://user-images.githubusercontent.com/57117748/169268943-2da0ee4f-7c15-45ed-ab55-82e8da631312.png)

4. 워드 프레스 설치
```
# cd /tmp && wget http://wordpress.org/latest.tar.gz	// cd /tmp 하고 http://wordpress.org 에서 latest.tar.gz 파일을 받기
# tar -xvzf latest.tar.gz -C /var/www/html	// 내려받은 파일 아카이브, 압축 풀기
# chown -R apache /var/www/html/wordpress	// 압축을 해제한 디렉토리의 소유자를 apache 로 변환
```

5. 워드프레스 세팅
- 타 네트워크에서 [서버 IP]/wordpress

![화면 캡처 2022-05-19 193834](https://user-images.githubusercontent.com/57117748/169281074-059d7628-4786-4f45-9ac0-3e775ab2c387.png)
![화면 캡처 2022-05-19 201536](https://user-images.githubusercontent.com/57117748/169281078-127756b3-640a-4c1d-bfb7-af79648b9c61.png)

# http 프로토콜
- Hypertext Transfer Protocol
- Web Service 의 핵심 프로토콜
- Request and Response 형태
- 비연결지향프로코콜 : 클라이언트가 서버에 요청을 보내면서 연결하고 서버가 응답을 보내면서 연결 해제

## http Request 

```
1. Method, URL, http version
2. Header
3. Blank
4. Body
```
- Method
	1. get : 지정url의 자료 요청
	2. post : 지정 url의 자료 요청
	3. head : http 헤더만 응답으로 달라
	4. delete : 요청하는 url의 자원을 삭제
	5. options : 서버 정보를 알려달라(사용가능한 메소드 정보 등)
	6. put : 요청하는 url 자원을 생성
	.
	.
	.


## http Response
```
1. http version, 상태코드(응답코드)
2. Header
3. Blank
4. Body
```

- 응답코드
	1. 200 : 정상작동
	2. 301 : 다른 URL로 다시 Rediretion
	3. 400 : 잘못된 요청
	4. 403 : Forbidden 접근금지
	5. 404 : Not Found 지정한 자원이 없음
	6. 500 : Internal Server Error 서버에서 오류가 발생
	7. 503 : Service Unavilable : 최대 세션 초과

## 쿠키 vs 세션
- 쿠키
	* http는 비연결지향적이다 그래서 요청을 이미 클라이어트가 보낸 식별 또는 연속적 요청을 추적하기위해 클라이언트에 저장하는 작은 데이터.
	1. Presistent Cookie / 지속쿠키
		- 저장공간에 저장되는 쿠키로 브라우저 또는 컴퓨터를 재식작해도 남아 있다.
	2. Session Cookies / 세션쿠키
		- 메모리에 저장하는 임시쿠키로 브라우저를 나갔다 오면 초기화 된다.

- 세션
	* 클라이언트가 아닌 서버에 저장되어 활용되는 데이터 조각 혹은 값
	* 클라이언트가 서버에 데이터 조각을 저장하고 부여받은 ID 즉 세션을 받고 Request를 하면 세션을 통해 서버가 해당 세션을 통해 매칭을 시킨다.

# NFS
- 리눅스, 유닉스 시스템끼리 저장공간을 공유하게 해주는 서비스
- 서버의 리소스, 스토리지를 클라이언트가 마치 자신의 리소스를 사용하는 것 처럼 하도록 제공하는 서비스
- 네트워크가 가능한곳이면 리눅스, 유닉스 운영체제간 NFS 를 사용하여 파일 시스템을 공유가 가능하다.

## DAS, NAS, SAN
1. DAS 
	- 하드디스크
	- PC 혹은 server에 케이블로 직접 연결해 사용하는 스토리지
	- 원거리에 있는 스토리지 연결 불가 혹은 최대 수량의 제한
	- 블록 단위의 I/O

2. SAN
	- 원격(광케이블, TCP/IP 등)으로 스토리지를 연결
	- 블록 단위의 I/O

3. NAS
	- 네트워크로 저장곤간을 연결 (Cloud)
	- 파일 단위로 읽고 쓰기


## NFS 공유 (NFS 서버 설정 및 공유)
1. 공유 디렉토리 설정 및 속성 설정   
```
# rpm -qa nfs-utils	// 설치 파일 확인
# vi /etc/exports 	// nfs 공유 디렉토리 속성 설정 파일
-----------------------------------------------------------------------------------------
vi

> /[폴더명] [공유하고자 하는 서버 IP] (rw,sync)	

* 속성 옵션
- rw : read and write
- sync : 여러 네트워크가 NFS 사용시 충돌을 방지
-----------------------------------------------------------------------------------------

# mkdir /[디렉토리명]		// 서버에서 공유될 디렉토리
# chmod 777 /[디렉토리명]	// 공유될 디렉토리 퍼미션 부여
# touch /[디렉토리명]/[파일명]		// 공유될 파일
```
![화면 캡처 2022-05-19 231449](https://user-images.githubusercontent.com/57117748/169315174-d46c5529-60de-4164-ac1d-6dc0141c9ea9.png)


2. nfs_server 실행 및 공유시작
```
# systemctl restart nfs-server
# systemctl enable nfs-server
# exportfs -v
```

![화면 캡처 2022-05-19 222554](https://user-images.githubusercontent.com/57117748/169304196-ac13152d-3aa2-4b25-9915-17152fff8222.png)

3. 방화벽 해제
```
# firewall-cmd --permanent --add-service mountd
# firewall-cmd --permanent --add-service rpc-bind
# firewall-cmd --permanent --add-service nfs
# firewall-cmd --reload
```
![화면 캡처 2022-05-19 231638](https://user-images.githubusercontent.com/57117748/169315633-dcbd10d8-f21d-4150-82d9-6dbb4e481a3c.png)

## NFS 사용 (클라이언트)
1. nfs 패키지 확인 및 공유 디렉토리 생성
```
# rpm -qa nfs-utils	// 설치 파일 확인
# mkdir /[클라이언트 공유 디렉토리]
```
![화면 캡처 2022-05-19 231815](https://user-images.githubusercontent.com/57117748/169316222-90c03daa-1856-42a4-8e56-ff684617d494.png)


2. 서버 공유 디렉토리 확인 및 마운트
```
# showmount -e [nfs 서버명]	// 해당 서버가 공유하고 있는 디렉토리 정보 확인
# mount -t nfs [서버IP]:/[서버 공유 디렉토리 경로] [클라이언트 공유 디렉토리 경로]
```
![화면 캡처 2022-05-19 232128](https://user-images.githubusercontent.com/57117748/169316957-8cc908eb-bfa9-4ab3-bb22-1ebfde9c14ee.png)


# 자동마운트

## Autofs
- 자동 마운트 서비스
- 자동으로 마운트하고 해당 파일 시스템을 일정 시간 사용하지 않으면 자동 언마운트
- 네트워크 파일 시스템, CDROM, USB  등등 마운트할때 사용

## AutoFs 파일 시스템과 맵(Map)
- Map : autofs 모듈이 동작하는데 있어 필요한 정보가 저장된 파일
- 3가지의 Map 존재 

1. master map
	- autoFs 파일 시스템에서 가장 기준이 되는 맵
	- /etc/auto.masater.d/[이름.autofs] 파일 형태로 저장
	- direct map 혹은 indirect map의 이름이나 포인터를 정의하는 역할

2. direct amp
	- /etc./[auto.이름] 의 형태로 관습적으로 설정 저장
	- 맵 내부의 마운트 포인터로 절대경로를 명시함

3. indirect map
	- /etc./[auto.이름] 의 형태로 관습적으로 설정 저장
	- 맵 내부의 마운트 포인터로 상대경로를 명시함

* direct, indirect 둘 중에 하나만 적용하면 됨
## direct amp 설정시키기

1. autofs 패키지 설치
```
# yum intall -y autofs 
```

2. master map 편집
```
# vi /etc/auto.master.d/[이름.autofs]

----------------------------------------------------------------------------
vi

> /-	/etc/[auto.이름]
----------------------------------------------------------------------------

// /-	/etc/[auto.이름] : 직접 맵 마운트를 사용하니 해당 경로의 파일로가서 확인해라
```

![화면 캡처 2022-05-19 233807](https://user-images.githubusercontent.com/57117748/169324426-b8283062-0174-4dad-a4f9-8c16cfb53bbb.png)


3. diract map 편집
```
# vi /etc/[auto.이름]
----------------------------------------------------------------------------
vi

> /nfs_share     -rw,sync     10.0.2.15:/share
----------------------------------------------------------------------------

// /nfs_share 이 경로에 마운트 할 것이다.
// -rw,sync 옵션으로 rw, sync 를 줄것이다.
// 10.0.2.15:/share nfs
```

![화면 캡처 2022-05-19 234112](https://user-images.githubusercontent.com/57117748/169324489-f7ef5618-f83a-4466-ad7e-774c16aad745.png)


4. 확인
```
# systemctl restart autofs.service
# systemctl enable autofs.service
# ls /[클라이언트 마운트 디렉터리]
```
![화면 캡처 2022-05-19 234617](https://user-images.githubusercontent.com/57117748/169324623-c49db462-f38f-448e-ae3a-637d0e43c40e.png)


## indirect amp 설정시키기
1. autofs 패키지 설치
```
# yum intall -y autofs 
```


2. master map 편집
```
# vi /etc/auto.master.d/[이름.autofs]

----------------------------------------------------------------------------
vi

> /indirect	/etc/[auto.이름]
----------------------------------------------------------------------------

// /indirect	/etc/[auto.이름] : 간접 맵 마운트를 사용하니 해당 경로의 파일로가서 확인해라
```
![화면 캡처 2022-05-19 235845](https://user-images.githubusercontent.com/57117748/169327492-6b51192e-fb9e-4a2b-83ef-47318747e01d.png)


3. indiract map 편집
```
# vi /etc/[auto.이름]
----------------------------------------------------------------------------
vi

> share     -rw,sync     10.0.2.15:/share
----------------------------------------------------------------------------

// /indirect/share 이 경로에 마운트 할 것이다.
// -rw,sync 옵션으로 rw, sync 를 줄것이다.
// 10.0.2.15:/share nfs
```
![화면 캡처 2022-05-19 235614](https://user-images.githubusercontent.com/57117748/169326919-5e103cb9-2589-4137-8101-9133ba4c9f43.png)


4. 경로에 파일 만들기 및 실행
```
# mkdir /indirect
# mkdir /indirect/share
# system restart autofs
```
![화면 캡처 2022-05-20 000014](https://user-images.githubusercontent.com/57117748/169327761-37e43e37-4454-44d6-892c-96c0cc5484a9.png)

# SSL 통신의 과정 (PKI)
![PKI](https://postfiles.pstatic.net/MjAyMjAzMTBfNjgg/MDAxNjQ2OTAxNjk2Mzk3.J2B7IO46B-abFN6iF8O7-V2zHBmfygoFU9p2wakJIA0g.KxB6_98laklhOpGqaj5KJOjFx7JXYB9Dpk_TtMp735Ag.PNG.escare1/SSL%ED%86%B5%EC%8B%A0%EB%B0%A9%EC%8B%9D.png?type=w966)

1. 클라이언트가 서버에게 'client hello'를 보낸다
	- 클라이언트의 브라우저에서 지원되는 암호화 방식
	- client hello :지원가능한 암호방식, 키교환방식, 서명방식, 압축방식 등을 서버에 알림)

2. 서버가 클라이언트에게 'server hello'를 '2번' 보낸다
	- server hello :지원가능한 암호방식, 키교환방식, 서명방식, 압축방식 등을 클라이언트에 알림)
	- 선택적으로 인증서 요구

3. 클라이언트는 자신의 비밀키를 서버의 공개키로 암호화 후 전송
	- 선택적으로 클라이언트 인증서 발송

4. 상호간 완료의 메세지를 주고받음

5. 비밀키를 통해 상호 암호화 통신