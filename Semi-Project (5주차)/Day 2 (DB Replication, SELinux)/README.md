# DB
- SQL : 관계형 데이터 베이스를 사용하기위한 문법
- DB : 데이터를 체계적으로 저장/관리/유지하기 위한 서비스.

## DB 설정 순서
1. 패키지 설치 (mariadb-server)
2. 설정파일 수정 (/etc/my.cnf)
3. 서비스 활성화 (mariadb)
4. 서비스 기본 보안 설정 (mysql_secure-installation 명령어 사용)
5. 방화벽 설정 (원격 접근이 필요한 경우만 설정)

## Mariadb-server 패키지 설치

0. 네트워크 설정
```
# nmcli con add type ethernet ifname [enp0s3 : 장치명] con-name [nat : 네트워크 이름] ipv4.addresses [10.0.2.10/24 : 네트워크 IP 주소] ipv4.gateway [10.0.2.1 : 네트워크 게이트 주소] ipv4.dns [8.8.8.8 : DNS 주소] ipv4.method manual
# nmcli con up [nat : 네트워크 이름]
# hostnamectl set-hostname [master.goorm.com : 호스트 이름]
# bash
[root@master ~]#
```

1. 패키지 설치
```
# yum -y install mariadb-server
# systemctl restart mariadb
# systemctl enable mariadb

# firewall-cmd --add-service mysql --permanent
# firewall-cmd --reload
```

## 표준 SQL 구문
1. SELECT : 데이터의 선택/확인
2. DML
	- INSERT : 데이터 삽입(행 추가)
	- UPDATE : 데이터 변경(값 변경)
	- DELETE : 데이터 삭제(행 삭제)
3. DDL
	- CREATE : 테이블(객체)를 생성
	- ALTER : 테이블의 구조 / 객체의 특성을 변경
	- DROP : 테이블(객체)를 삭제
4. DCL
	- GRANT : 권한 추가
	- REVOKE : 권한 제거
5. TCL
	COMMIT : 변경사항을 적용
	ROLLBACK : 이전 상태로 되돌리기

# DB의 이중화
- 필요성 : 서비스 제공을 원활하게 하기 위해 장애 발생에 대한 대비
- 특징 : 마스터와 슬레이브 (마스터는 쓰기 사능 / 슬레이브는 읽기 전용)

## 백업 및 복원
- 필요성 : 데이터 베이스에 저장한 데이터를 안전하게 유지하기 위헤
- 방식별 특징
	1. 물리적 백업
		- 속도 빠름 / 제약 사항 같은 시스템(서비스)
	2. 논리적 백업
		- 속도 느림	/ 관계없음

## 구성방법
```
1. 기본 서비스가 구성된 2개 이상의 시스템을 준비

2. 마스터 역할을 시스템에서 설정
	2-1. 시스템을 구분할 ID 지정(설정파일)
	2-2. 사용자 설정(생성/권한)
	2-3. 방화벽 설정(MySQL)
	2-4. 데이터 베이스 백업

3. 슬레이브 역할을 시스템에서 설정
	3-1. 시스템을 구분할 ID 지정(설정파일)
	3-2. 마스터 지정 및 슬레이브 상태로 설정
	3-3. 마스터 백업 데이터를 이용해 복원
``` 

## MariaDB의 COMMIT 방식
- default : auto-commit (오류가 발생하면 복구가 힘듬)
- 활성/비활성 설정 시에는 세션(일시적)설정과 설정파일(영구)설정이 있음

- 현재 설정 값 확인 및 일시적 변경
```
> show variables like 'autocommit';	// 현재 commit 설정 값 확인
> MariaDB [(none)]> set autocommit = off;
```
![화면 캡처 2022-05-24 180616](https://user-images.githubusercontent.com/57117748/169994675-92b78bfe-76bf-4902-985e-2a8b86a291b6.png)

- 설정 파일을 통한 영구적 변경
```
vi /etc/my.cnf

-------------------------------------------------------------
[mysqld]
autocommit=off	// 비활성화 시에는 COMMIT / ROLLBACK 구문 사용 가능
```
![화면 캡처 2022-05-24 144430](https://user-images.githubusercontent.com/57117748/169995718-5adc870e-53eb-45f0-8054-11b837eb74e9.png)

## 이중화 설정 
0. DB 및 테이블 생성 및 데이터 추가 autocommit 해제
```
> create database [examdb : DB명];	
> create table [test_tab : 테이블명] (ID int, NAME varchar(5), LOCATE varchar(5));	// () <- 데이터 구성 및 타입
> insert into [test_tab : 테이블명] values(1, 'kim', 'seoul')	// 데이터 추가
```
![화면 캡처 2022-05-24 182850](https://user-images.githubusercontent.com/57117748/169999538-14439ca8-5860-491a-9dc6-5d721e43413a.png)
![화면 캡처 2022-05-24 182904](https://user-images.githubusercontent.com/57117748/169999552-53386c36-2382-409f-a234-23e395a292e8.png)
![화면 캡처 2022-05-24 182915](https://user-images.githubusercontent.com/57117748/169999556-9b721dc7-1a1a-447c-8d80-ab4677bde9a5.png)

1. 이중화구성 스크립트 설정
```
vi /etc/mu.cnf

-------------------------------------------------------------
[mysqld]
server_id=[100 : NewMasterId]	// master id 구성
log-bin=mariadb-bin	// log 는 mariadb-bin 에 저장
autocommit=0		// autocommit 비활성
-------------------------------------------------------------

systemctl restart mariadb.service
```
![화면 캡처 2022-05-24 185500](https://user-images.githubusercontent.com/57117748/170004675-f1b667a9-4f09-48c6-9133-59f917b6c83a.png)

3. 사용자 권한 설정
```
> GRANT replication slave on *.* TO [replica : 사용자 이름]@'%' IDENTIFIED BY '[1234 : PW]'; // 사용자 생성 + 권한 설정 
> FLUSH PRIVILEGES;	// 사용자 설정에 대한 변경 값 적용
```

4. Slave 설정 파일 수정
```
# vim /etc/my.cnf
--------------------------------------------------------
[mysqld]
server-id=[200 : NewSlave d] // Slave id 구성
log-bin=mariadb-bin // log 는 mariadb-bin 에 저장
read_only=1
report-host=[slave.goorm.com : slave host name]
--------------------------------------------------------

# systemctl restart mariadb
```
![화면 캡처 2022-05-24 232217](https://user-images.githubusercontent.com/57117748/170059715-da55b0ed-438d-40a7-b441-171bfe4684a4.png)

5. 데이터 백업 전 잠금
```
> flush tables with read lock;	// mysql tables lock
```

6. master status 확인
```
> show master status;
```
![화면 캡처 2022-05-24 233451](https://user-images.githubusercontent.com/57117748/170061682-40bbacf4-b51b-40bd-8fd2-848148503373.png)

7. 새로운 터미널에서의 백업파일 생성
```
# mysqldump -u root -p --all-databases --lock-all-tables --events > mariadb_backup.sql 	// mariadb_backup.sql에 데이터베이스, 테이블 등 백업
```
![화면 캡처 2022-05-24 234231](https://user-images.githubusercontent.com/57117748/170063473-41b3c6f6-6eb3-4903-bc86-e3f05d146f5a.png)

8. 기존의 터미널에서 잠금해제
```
> unlock tables;	// 잠금해제
```

9. 백업한 파일을 슬레이브로 전달
```
# scp mariadb_backup.sql root@10.0.2.15:/tmp	// slave ip 해당 경로로 파일 복사 생성
```
![화면 캡처 2022-05-24 235248](https://user-images.githubusercontent.com/57117748/170065978-0f2df1c3-b9ab-4794-b556-57ea8ce8e444.png)

10. 슬레이브에서 백업 데이터 복원
```
mysql -u root -p < /tmp/mariadb_backup.sql 	// 백업 파일 슬레이브에서 복원
```
![화면 캡처 2022-05-25 000054](https://user-images.githubusercontent.com/57117748/170067831-763d1fca-47cd-4f21-922f-ca986fbfe83c.png)

11. 슬레이브에서 마스터 지정 및 설정
```
> change master to master_host='10.0.2.10' ,
> master_user='replica', master_password='1234', 
> master_log_file='masterdb-bin.000001',
> master_log_pos=461;
-----------------------------------------------------------
> 마스터 IP 주소
> 사용자 이름 및 PW
> log_file
> log_pos
-----------------------------------------------------------
```
![화면 캡처 2022-05-25 001325](https://user-images.githubusercontent.com/57117748/170070656-50eefcbd-8da8-4065-8831-97d5c695e884.png)

12. 슬레이브에서 설정 시작 및 상태 확인
```
> start slave;
> show slave status \G;
```
![화면 캡처 2022-05-25 001844](https://user-images.githubusercontent.com/57117748/170071879-24b0f962-0be6-4be1-a42a-ec64fae22387.png)

13. 동기화 확인
- commit 전
```
> use [examdb : DB명]
> insert into test_tab values(2, 'choi', 'suwon');
```
![화면 캡처 2022-05-25 003608](https://user-images.githubusercontent.com/57117748/170075559-94f3fc7b-f1fd-4095-ac81-3a028c43a516.png)

- commit 후
```
> use [examdb : DB명]
> insert into test_tab values(2, 'choi', 'suwon');
> commit;
```
![화면 캡처 2022-05-25 003826](https://user-images.githubusercontent.com/57117748/170076010-34b31473-8023-43d7-aee0-612fd7826e8b.png)

# 웹서버와 DB서버 연동
1. 웹서버 / DB서버 구성
2. 추가적인 패키지 설치
3. SELinux 설정

## SELinux
- 리눅스 환경을 좀 더 안전하게 사용하는 방법 중 하나
- 파일에 대한 접근 등에 있어 사용자 + 프로세스 확인 절차 추가
- 규칙 설정시 파일 / 포트 / 프로세스 동작
	1. 파일 컨텍스트 
		- ls -Z 를 통해 파일에 대한 접근 제어 확인
		- 위에서 표시된 3번째 항목 값에 따라 접근할 수 있는 프로세스 지정
		![화면 캡처 2022-05-25 005140](https://user-images.githubusercontent.com/57117748/170078861-9e2af01b-0161-4879-ae26-1706ef1efed9.png)
		```
		# restorecon -Rv /var/www/html	// 지정한 디렉토리 아래 파일을 모두 재설정(기본값 적용)
		# chcon -t admin_home_t /var/www/html/fileA // 내가 파일에 대해 임의로 값을 지정 (권장하지 않음)
		# semanage fcontext -a -t httpd_sys_content_t '/root/dirA(/.*)?'
		```
		![화면 캡처 2022-05-24 162530](https://user-images.githubusercontent.com/57117748/170080514-8b80c991-abd2-42f0-936d-c8583ff0846c.png)
	2. 포트 레이블(컨텍스트)
		- 각 포트 번호마다 특정 서비스 용도로 지정, 설정이 다르면 해당 서비스로 접근 차단
		- 필요한 경우(별도의 포트를 사용)에는 따로 포트 레이블 값을 변경
		```
		# semanage port -l | grep 80	// 확인
		# semanage port -a -t nfs_port_t 8888 -p tcp	// 변경
		```
	3. 불리언
		- 각 서비스(프로세스) 들에 대한 기능 활성화 여부
		```
		# semanage boolean -l | grep db 	// 확인
		# semanage boolean httpd_can_network_connect_db -m --on // 설정
		```
		* httpd_can_network_connect_db 외부의 httpd 서비스를 통해 내부의(로컬) 데이터베이스로 접근을 허용.
		![화면 캡처 2022-05-24 164009](https://user-images.githubusercontent.com/57117748/170080522-39f83bf1-2a2d-4241-8325-572dec71ecaa.png)


- 명령어
```
# getenforce		// 확인
# setenforce		// 설정
# vi /etc/selinux/config	// 영구설정
```

* 여기에서 활성화(enforcing) 로 설정했을 경우에만 위에서 말한 규칙에 따라 동작을 제어

