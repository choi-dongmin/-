# SELinux
- 보안에 취약한 리눅스를 보호하기 위해 탄생
- /etc/sysconfig/selinux 해당 파일에서 확인 및 수정이 가능

## SELinux 의 옵션
1. 강제 (Enforcing)
	- 시스템 보안에 영향을 미치는 기능이 확인이 된다면 그 기능의 작동을 정지한다.\

2. 차단 (Permissive)
	- 시스템 보안에 영향을 미치는 기능이 확이되면 허용하지만 로그에 기록으로 남긴다.

3. 허용 (Disabled)
	- SELinux 가 비활성 상태

![화면 캡처 2022-05-18 102700](https://user-images.githubusercontent.com/57117748/169002553-0a1f43fc-de33-4689-9c8c-2a1d56a4ca4f.png)


## SELinux 명령어

- 상태 전환
```
setenforce 0	// 허용 상태로 전환
setenforce 1	// 강제 상태로 전환
* Disabled 상태는 명령어가 없음

getenforce 	// 현재 SELinux 상태 확인 
```

![화면 캡처 2022-05-18 103248](https://user-images.githubusercontent.com/57117748/169002667-1f4c2a19-b59b-4d37-aa9b-12a8f1879838.png)


# DB
- Database 의 약자로 여러 사람들에 의해 공유되고 사용될 목적으로 통합하여 관리되는 데이터의 집합

## DB 용어정리
```
- 데이터 : 하나 하나의 자료들
- 테이블 : 데이터를 표 형식으로 만든 테이블
- DBMS : 데이터를 관리하는 소프트웨어 혹은 솔루션 (MYSQL, Maria....)
- row : 테이블의 행, 가로줄에 해당하며 레코드라고도 한다.
- column : 테이블의 열, 세로줄에 해당
- Primary Key : 레코드를 식별하는 유일한 값을 갖고 있는 비어있지 않은 필드
- Foreigne Key : 다른 테이블의 Primary Key 와 대응되는 column
- SQL(Structured Query Language) : 구조화된 질의 언어라는 뜻으로 DB에서 정보를 얻고 조작하고 갱신하는 등의 역할을 하는 언어
```

## RDBMS
- 관계형 데이터 베이스
- 데이터의 크기가 거대해짐에 따라 나타난 기술 상호간 관계가 있는 데이터를 기준으로 하나의 데이터로 볼 수 있도록 한 기술
- 즉, 어떠한 데이터가 상호호환 된다면 해당 데이터를 기준으로 여러 컬럼을 하나의 데이터로 나타내는것.
- MYSQL, MariaDB, OracleDB 

## NOSQL
- 관계형 데이터 베이스를 지원하지 않는 SQL로 빠른 데이터 처리가 가능하지만 안정서에서 RDBMS 보다 떨어진다.
- 그러나 빅데이터의 등장으로 다시 각광을 받고 있다.
- MongoDB

## Install MariaDB for Linux

1. 설치 및 확인
- yum 을 통해 mariadb, mariadb-server 를 설치
```
# yum -y install mariadb mariadb-server
```
- Install
![화면 캡처 2022-05-18 181411](https://user-images.githubusercontent.com/57117748/169004469-c4416601-f878-4784-90c5-7aa14449c312.png)
![화면 캡처 2022-05-18 181809](https://user-images.githubusercontent.com/57117748/169004636-68a3b135-aaae-4c3e-999e-826e5d004b48.png)

- Check
![화면 캡처 2022-05-18 182058](https://user-images.githubusercontent.com/57117748/169005210-a04e9bfd-2812-4096-83ca-b174b892ecdc.png)

2. 방화벽 해제
- firewall-cmd 을 통해 방화벽을 해제 이떄 mariadb 가 아닌 mysql 로 입력
```
# firewall-cmd --permanent --add-service=mysql
# firewall-cmd --reload
```
![화면 캡처 2022-05-18 182246](https://user-images.githubusercontent.com/57117748/169005577-91ae8a6b-ede4-45b5-b11e-2ac27693f61e.png)

# Maria SQL 사용

## root 계정에 Password 설정
- mysqladmin 을 사용해 계정설정
```
# mysqladmin -u root password '[NewPW]';
```
![화면 캡처 2022-05-18 114731_LI](https://user-images.githubusercontent.com/57117748/169010492-5aae4329-5bdf-40d0-99b8-d5f87eaeef5a.jpg)


## DATABASE 사용
- SHOW : DB 목록 확인
```
> SHOW DATABASES ;	// DB 목록 출력
```
![화면 캡처 2022-05-18 185028](https://user-images.githubusercontent.com/57117748/169011486-1d95215b-14a0-415f-b2f8-af37d0195d9e.png)


- CREATE : DB 생성
```
> CREATE DATABASE [DB이름];
```
![화면 캡처 2022-05-18 185404](https://user-images.githubusercontent.com/57117748/169012275-8317b97a-2883-4819-b7cb-da54add829b9.png)

- DROP : DB 삭제
```
> DROP DATABASE [DB이름];
```
![화면 캡처 2022-05-18 185650](https://user-images.githubusercontent.com/57117748/169012800-8d2cbe49-c9c2-44fa-8082-d2dd157ced52.png)

- USE : 해당 DB 진입
```
> USE [DB이름];
```
![화면 캡처 2022-05-18 115255](https://user-images.githubusercontent.com/57117748/169012899-794e378b-4add-4481-a1fc-176897210b9b.png)

## Table 관리
- CREATE TABLE : 테이블 생성
```
> CREATE TALBE [테이블명] ([DataType]..);
--------------------------------------------

> CREATE TABLE (
	> id VARCHAR(12) NOT NULL PRIMARY KEY,
	> name VARCHAR(5),
	> age INT,
	> address VARCHAR(5)
); 

SQL 데이터 종류 --------------------------------
1. VARCHAR(n) : n개의 가변적인 문자열
2. CHAR(n)	: n개로 고정된 문자만 받는다
3. NOT NULL : NULL 값은 올 수 없다
4. PRIMARY KEY : 주요 키 컬럼
5. Int : 정수
6. Float : 실수
7. Date : 날짜
8. Time : 시간
```
![화면 캡처 2022-05-18 214258](https://user-images.githubusercontent.com/57117748/169041232-7b719556-8b0a-4a5e-bc4c-070239f61bb9.png)



- SHOW TABLES : 테이블 목록확인
```
> SHOW TABLES;
```
![화면 캡처 2022-05-18 214334](https://user-images.githubusercontent.com/57117748/169041360-9835da2c-e3b4-46e9-ace4-a0e1984491e7.png)


- ALTER : 테이블에 컬럼 추가
```
> ALTER TABLE [TableName] ADD COLUMN [NewColume] [DataType]
```
![화면 캡처 2022-05-18 215348](https://user-images.githubusercontent.com/57117748/169043163-260ed258-6fd0-48db-ac03-6609681fe0b5.png)


- EXPLAIN, DESCRIBE, DESC : 테이블 컬럼 정보
```
> DESC [TableName]
```
![화면 캡처 2022-05-18 215317](https://user-images.githubusercontent.com/57117748/169043058-dc6bed16-ee5d-4af3-91af-b3825c5a2b3b.png)


- DROP TABLE : 테이블 삭제
```
> DROP TABLE [테이블명]
```
![화면 캡처 2022-05-18 214642](https://user-images.githubusercontent.com/57117748/169041885-5a4340bc-ea7b-43b2-bf27-15e9f21e5f32.png)

## 데이터 CRUD
- INSERT : 테이블 컬럼에 데이터를 입력
```
> INSERT INTO [테이블명] VALUES ('Choi','DM','29','Suwon','M')
```
![화면 캡처 2022-05-18 220050](https://user-images.githubusercontent.com/57117748/169044563-3e3ba02c-1f70-4d64-8385-b02b02e9c872.png)


- SELECT : 테이블의 데이터 읽기
```
SELECT * FROM member;

* 전부
WHERE 을 통해 조건 부여 가능
```

- UPDATE : 데이터를 수정
```
> UPDATE [테이블명] SET [컬럼명='수정값'] WHERE [조건값]
```
![화면 캡처 2022-05-18 221940](https://user-images.githubusercontent.com/57117748/169048223-6583ff15-2bb6-439d-a3df-7d32e30f1e7f.png)



- DELETE : 데이터 삭제 
```
> DELETE FROM [테이블명] WHERE [조건값];
```
![화면 캡처 2022-05-18 222309](https://user-images.githubusercontent.com/57117748/169048975-115847d2-1c01-4247-b2a8-e2b56d0f470b.png)


## 수퍼계정 만들기
- 다른 IP에서 SQL서버에 접근할 수 있는 수퍼계정 생성 및 권한 부여

1. USE를 통해 DATABASE mysql로 접근한다.
2. SELECT user,host FROM user WHERE user NOT LIKE '' // user, host 가 null 값이 아닌 값만 출력 (현재는 root만 출력)
3. GRANT ALL PRIVILEGES ON *.* TO [계정명]@'[X.X.X.%]' IDENTIFIED BYYE 'PW' ;
	- 해당 X.X.X.0~255 아이피 모두 접근 이 계정으로 접근 가능 

![화면 캡처 2022-05-18 125157_LI](https://user-images.githubusercontent.com/57117748/169051849-e23bbf4c-cc9f-406a-be0f-8d81afc211cc.jpg)


# LAMP
- HTML.PHP 등이 실행 할 수 있는 환경을 제공하는 웹 서버 프로그램
- Linux, MYSQL, Apach, PHP
- 과거 60%의 점유율을  현재 gnix 가 따라잡고 있다.
- 오픈소스이며 다양한 플렛폼에서 유연하게 동작한다.

## AMP 구현
1. install AMP
```
# yum -y install maraidb, mariadb-server php php-mysqlnd 
```
![화면 캡처 2022-05-18 225548](https://user-images.githubusercontent.com/57117748/169058214-7cff43f8-e771-4f2b-a01c-6755ce9530e8.png)

2. httpd, mariadb start and enable setting
```
# systemctl restart mariadb
# systemctl restart httpd

# systemctl enable mariadb
# systemctl enable httpd
```
![화면 캡처 2022-05-18 232325](https://user-images.githubusercontent.com/57117748/169064834-6d9e65eb-f27f-4a8c-9e00-c5e518fe3414.png)

3. firewall add service
```
# firewall-cmd --permanent --add-service=http
# firewall-cmd --permanent --add-service=https
# firewall-cmd --reload
```
![화면 캡처 2022-05-18 232721](https://user-images.githubusercontent.com/57117748/169065729-5c57882f-46f4-4059-9e7c-a92ecc7794d9.png)

4. var/www/html 파일추가
```
# cd /var/www/html
# touch phpinfo.php
# vi phpinfo.php
-------------------------------
vi

<?php phpinfo(); ?>
```
![화면 캡처 2022-05-18 144240](https://user-images.githubusercontent.com/57117748/169066527-f806e2d5-4f71-4e21-a6ce-a42efd649b94.png)
![화면 캡처 2022-05-18 144251](https://user-images.githubusercontent.com/57117748/169066582-d441a523-258d-4359-b803-59262ce6d5d0.png)

5. 외부에서 서버연결 확인하기 
![화면 캡처 2022-05-18 144421](https://user-images.githubusercontent.com/57117748/169066735-70b50349-4233-4f8b-89a0-8c4bd923e660.png)

## 주요 속성 파일

1. /var/www		// 웹서버 통합 디렉토리

2. /var/www/html 	// 기본 웹 콘텐츠 모음 디렉토리

3. /etc/httpd/conf/httpd.conf // 웹 서버 기본 속성 설정 파일

4. /var/log/httpd/error_log		// 사이트에서 발생한 에러 이벤트 발생시 기록

5 /var/log/httpd/access_log		// 사이트에 접근 이벤트 발생시 기록

# XE
- xpressengine.com 
- 우리나라에서 만든 오픈소스 웹 사이트 제작 프레임워크

## XE 설치 및 실행
- xpressengine.com 에서 설치파일 xe.zip 다운로드

1. XE 베이스 패키지 설치
```
# yum -y install php-gd.x86_64
```
![화면 캡처 2022-05-18 152547](https://user-images.githubusercontent.com/57117748/169071538-34dbad3e-49d5-458b-b9c2-b1718d55f60e.png)


2. /etc/httpd/conf/httpd.conf 수정
```
# vi /etc/httpd/conf/httpd.conf
-----------------------------------------------------------
vi

151   AllowOverride All 	// .htaccess 파일을 를 읽어 올 수 있다.

```
![화면 캡처 2022-05-18 152929](https://user-images.githubusercontent.com/57117748/169072752-8d127d25-142e-4f90-86da-0bfc48f5e689.png)


3. xe.zip /var/www/html 에 옮기고 압축풀기
```
# cd ~/Downloads/
# mv xe.zip /var/www/html
unzip xe.zip
```

4. xe 디렉토리에 707 퍼미션 주기
```
chmod 707 xe
systemctl restart httpd
```
![화면 캡처 2022-05-19 000859](https://user-images.githubusercontent.com/57117748/169075262-831394fe-86bd-4286-8cba-f0df8cce282a.png)
![화면 캡처 2022-05-19 001050](https://user-images.githubusercontent.com/57117748/169075548-0f4a3571-df6f-49c5-b835-b2b2d76c7ab7.png)


5. xe 에서 사용 할 수퍼계정 생성
	- Maria SQL 접속 후 mysql DB를 사용
	- 수퍼 계정 생성
```
GRANT ALL PRIVILEGES ON [DB명].* TO [계정명]@localhost IDENTIFIED BY '[PW]]';
```
![화면 캡처 2022-05-19 001556](https://user-images.githubusercontent.com/57117748/169077666-de5ce204-b624-4c50-affa-1d17377a475f.png)


6. xe 에서 사용 할 DB 생성
	- 이전 만든 수퍼 [계정명] 으로 Maria SQL 실행
	- CREATE DATABASE [DB명]
![화면 캡처 2022-05-19 002102](https://user-images.githubusercontent.com/57117748/169079009-53c4ffc4-3e02-4957-bd0c-2e51657ae7ac.png)
![화면 캡처 2022-05-19 002241_LI](https://user-images.githubusercontent.com/57117748/169080456-0ff08289-33f7-4726-aff5-dd5d53c145d2.jpg)

7. http//: 서버IP/xe/ 를 통해 나머지 설치 진행
![화면 캡처 2022-05-19 002749](https://user-images.githubusercontent.com/57117748/169082569-e96899d1-3546-48ec-9ba1-80c24be530a2.png)
![화면 캡처 2022-05-18 155421](https://user-images.githubusercontent.com/57117748/169082581-3ab95823-a3d5-40aa-af21-c733a4a1c829.png)
![화면 캡처 2022-05-18 155518](https://user-images.githubusercontent.com/57117748/169082589-b42262ea-d491-47be-bf88-ad8e18169ef6.png)
![화면 캡처 2022-05-19 003223](https://user-images.githubusercontent.com/57117748/169082596-7dc61f3a-b40d-4a92-8b35-dedbdad43519.png)


# FTP 서비스
- FTP 파일 전송 프로토콜

## Active, Passive Mod

1. Active 모드 
![Active mod](https://img.extrememanual.net/2015/12/ftp_theory_diagram_active.jpg)

1. 클라인언트가 자신의 임의포트 5150와 함께 서버 FTP : 21 포트로 접속한다.
2. 클라이언트는 5151(임의포트)를 통해 데이터를 받는다는 신호를 서버에게 보낸다.
3. 서버는 5151 포트로 접속해 데이터를 전송한다고 응답.
4. 클라이언트가 응답하고 데이터 받는다.

* 서버가 클라이언트의 해당 임의포트로 접속해 데이터를 보낸다.

2. Passive 모드
![Passive mod](https://img.extrememanual.net/2015/12/ftp_theory_diagram_passive.jpg)

1. 클라이언트가 서버에 접속
2. 서버에서 데이터 전송을 위한 포트를 클라이언트에게 전송
3. 클라이언트는 데이터 전송을 위한 포트에 데이터 전송
4. 서버의 응답과 함께 클라이언트는 서버측의 포트로 데이터를 넘겨받음

- 실제적으로 ftp 가 21번 포트는 단순히 명령어를 위한 포트, 데이터 전송은 다른곳에서 이루어짐

* 즉, 서버측에서 20번 포트를 사용해 클라이언트에게 접속하면 active, 클라이언트가 서버에서 알려준 데이터 포트로 접근하면 passive

## vsftp
- ftp 서비스중 대표적인 서비스로 RedHatOS 에서 주로 사용한다.
- /var/ftp/pub : 기본 공유 디렉토리
- /etc/vsftpd/vsftpd.conf : vsftpd 설정 파일
- /etc/vsftpd/ftpusers : 사용불가 계정설정 파일

## vsftpd 설정 파일

```
/etc/vsftpd/vsftpd.conf

etc/vsftpd/vsftpd.conf
anonymous_enable :익명사용자 접속허용
local_enable : 로컬 계정 접속 허용
write_enable : 로컬 계정이 접속한후에 디렉터리 만들기, 삭제, 생성 등 쓰기가 가능하게 허용
anon_upload_enable : 익명사용자 업로드 허용
anon_mkdir_write_enable : 익명사용자 디렉터리; 생성 허가 여부
dirlist_enable : 접속한 디렉토리 파일리스트 보기 허용
download_enable :다운로드 허용
listen_port : ftp 서비스 포트번호
```
## vsftp 서버 구축

1. vsftp 설치 및 실행, 방화벽 해제
```
# yum -y install vsftpd

# systemctl restart vsftpd
# systemctl enable vsftpd

# firewall-cmd --permenent --add-service=ftp
# firewall-cmd --reload
```

2. 익명 계정 업로드 허용
- etc/vsftpd/vsftpd.conf 수정을 통해 익명 계정 업로드 허용
- default 는 업로드 허용

```
# vi /etc/vsftpd/vsftpd.conf
------------------------------------------------------------
anonymous_enable=yes :익명사용자 접속허용
anon_upload_enable=yes : 익명사용자 업로드 허용
anon_mkdir_write_enable=yes : 익명사용자 디렉터리; 생성 허가
download_enable=yes :다운로드 허용
```

3. /var/ftp/pub 디렉토리 소유자 ftp로 변경
	- chown 명령어 사용
```
# chown ftp:ftp /var/ftp/pub
```
![화면 캡처 2022-05-19 011038](https://user-images.githubusercontent.com/57117748/169090709-e08b3205-df47-41a3-835f-eb10dc0a6592.png)



