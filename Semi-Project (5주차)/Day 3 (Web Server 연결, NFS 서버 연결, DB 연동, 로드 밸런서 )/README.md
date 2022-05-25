# Web Server 연동
- http / https 프로토콜 이용
- 정적 페이지(html, img) + 동적 페이지(WAS, WebAPP) 제공 
- 페이지 : HTML, CSS, PHP, Python....
- 구성방식 
	1. APACHE
	2. Ngnix
	3. IIS

## Web Server와 서비스 연동시키기

1. Apache 구성하기
```
# yum install -y httpd      
# systemctl restart httpd
# systemctl enable httpd      
# firewall-cmd  --permanent  --add-service=http 
# firewall-cmd  --reload  
# echo "test web server 1" > /var/www/html/index.html
```

2. 추가적인 스크립트를 사용 (PHP)
- 추가 패키지 설치
```
# yum -y install php php-mbstring php-pear php-fpm php-mysql
```
	
- 언어를 위한 설정파일 수정
```
# vi /etc/httpd/conf.d/php.conf

-------------------------------------------------------------
<FilesMatch \.php$>
        SetHandler "proxy:fcgi://127.0.0.1:9000"	//추가
#    SetHandler application/x-httpd-php				// 주석처리
</FilesMatch>
-------------------------------------------------------------

# systemctl restart httpd
# systemctl enable httpd
# systemctl restart php-fpm
# systemctl enable php-fpm
```
![화면 캡처 2022-05-25 184014](https://user-images.githubusercontent.com/57117748/170232590-06f0afe1-d5d5-4800-bfc0-b88db72550ea.png)

- 웹 서비스를 제공할 스크립트 작성
```
vi /var/www/html/info.php

--------------------------------------------------------------

<?php phpinfo(); ?>
```
![화면 캡처 2022-05-25 123056_LI](https://user-images.githubusercontent.com/57117748/170237602-390af96d-851b-4fc1-a065-9ddb1dffa5f4.jpg)

- DB 서비스와 연동할 스크립트 작성
```
vi /var/www/html/db.php

-----------------------------------------------------------------
<?php
$conn = mysqli_connect('[10.0.2.10 : DB서버 IP]', '[remote : SQL 사용자 이름]', '[123 : SQL 사용자 비밀번호]', '[examdb : 데이터베이스명]');

$sql = "SELECT * FROM [test_tab : 테이블 명]";
$result = mysqli_query($conn, $sql);
  while($row = mysqli_fetch_array($result)) {
    echo $row['ID'].'  '.$row['NAME'].'  '.$row['LOCATE'].'<br>';
  }
?>
```
![화면 캡처 2022-05-25 194010](https://user-images.githubusercontent.com/57117748/170243980-7b36394b-1c31-4238-87f9-7a25ae88bcb7.png)




- 데이터베이스 사용자 생성 및 설정 (DB 서버)
```
> GRANT select ON *.* TO remote@'%' identified by '123';
> flush privileges;
```
![화면 캡처 2022-05-25 192028](https://user-images.githubusercontent.com/57117748/170240487-68f6f4e2-b752-408b-8475-8470ffff0aa6.png)
![화면 캡처 2022-05-25 192016](https://user-images.githubusercontent.com/57117748/170240451-4adf6368-e56c-45c2-9f43-da6541468b2c.png)


- 웹서버에서 SELinux 설정
```
# semanage boolean -m httpd_can_network_connect_db --on
# semanage boolean -l | grep httpd_can_network_connect_db
```
![화면 캡처 2022-05-25 192321](https://user-images.githubusercontent.com/57117748/170241013-bbb1fc93-6350-4ccf-b9ee-c47c6f45b7d4.png)

* httpd_can_network_connect_db : on일시 http와 DB을 연결가능 

# NFS
- Network File System
- 리눅스/유닉스에서 사용하는 표준 프로토콜
- NAS 방식
- 보안방식 구성을 위해 kerberos 서비스와 연동

## 설정방식
```
1. 패키지 설치
2. 디렉토리 준비
3. 서비스 활성화
5. 방화벽오픈
6. 클라이언트에서 패키지 설치
7. 마운트
```
## NFS와 Web 연동

0. 패키지 확인 및 설치
```
# yum list nfs-utils
```

1. 공유파일 생성
```
# mkdir /contents1
# mkdir /contents2
```

3. /etc/exports 설정 파일 설정
```
# vi /etc/exports

---------------------------------------------------------

> [/contents1 : 공유폴더 경로]	[IP,IP대역,*](rw,sync,sec=sys)
> [/contents2 : 공유폴더 경로]	[IP,IP대역,*](rw,sync,sec=sys)
---------------------------------------------------------
# exportfs -r 	// 설정 변경 사항 적용
```
![화면 캡처 2022-05-25 212302](https://user-images.githubusercontent.com/57117748/170261008-9bd8824e-9f2d-4b7f-b05c-7e54254d8cab.png)

4. nfs-server 시작 및 방화벽 해제
```
# systemctl restart nfs-server
# systemctl enable nfs-server
# firewall-cmd --permanent --add-service=nfs
# firewall-cmd --reload
```

5. 클라이언트에서 마운트 설정 (웹서버)
```
# vim /etc/fstab

-------------------------------------------------------------
> [10.0.2.20:/contents1 : nfs 서버 IP 및 공유폴더 경로]     [/var/www/html : 공유폴더 마운트 경로]   nfs     rw,sync,sec=sys   0    0

```
![화면 캡처 2022-05-25 213437](https://user-images.githubusercontent.com/57117748/170263119-2bc6f991-6300-48d7-8fd0-66fcb767f27b.png)


* 위와 같이하면 NFS와 연결은 되지만 http와 nfs가 연동되지 않음(SELinux의 영향)

6. SELinux 추가 버전 설정 (마운트 해제 상태에서 진행)
```
# vi /etc/sysconfig/nfs
----------------------------------------------------------

RPCNFSDARGS="-V4.2"		// "" 안에 버전을 명시
----------------------------------------------------------

# systemctl restart nfs-server
```
![화면 캡처 2022-05-25 214027](https://user-images.githubusercontent.com/57117748/170264108-64706aac-e6af-464e-9b3b-7498932a9622.png)
![화면 캡처 2022-05-25 214202](https://user-images.githubusercontent.com/57117748/170264404-29b68b41-218f-464e-b8f1-3918a448613d.png)

7. 클라이언트에서 설정 추가
```
# vi /etc/fstab

------------------------------------------------------------
> 10.0.2.20:/contents1     /var/www/html   nfs     rw,sync,sec=sys,v4.2,context="unconfined_u:object_r:httpd_sys_content_t:s0"   0    0

//context 라는 항목에 값을 입력 (기존의 html 파일의 컨텍스트)

------------------------------------------------------------

mount -a
```
![화면 캡처 2022-05-25 214815](https://user-images.githubusercontent.com/57117748/170265635-b9855e0b-ee6a-4460-b4d0-3ec457e0aa56.png)


# 로드 밸런서
- 작업 부하에 대한 부분을 분산해주는 역할
- 네트워크 트래픽에 대한 로드 밸런싱이 일반적
- L4 로드밸런서와  L7 로드밸런서로 구분이 가능
- 인증서 관리에 대한 편의 제공 (SSL 설정)
- proxy 서버를 통한 기능 구현 가능 (nginx , pen ,  squid , haproxy ...)

## 로드 밸런스 설정
1. 패키지 설치
```
# yum -y install haproxy
```

2. 서비스 설정 파일 백업 및 수정
```
# cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.backup
# vi /etc/haproxy/haproxy.cfg

--------------------------------------------------------------------
frontend lb
        bind *:80
        default_backend web
        option  forwardfor
backend web
        balance roundrobin
        server  web01 10.0.2.10:80 check		// 웹 서버 IP 1개
        server  web02 10.0.2.15:80 check		// 웹 서버 IP 1개 
--------------------------------------------------------------------
```
![화면 캡처 2022-05-25 222038](https://user-images.githubusercontent.com/57117748/170271860-5ee2f9e8-a905-4741-ab15-9448e7abb209.png)


3. haproxy 실행 및 http 방화벽 해제
```
# systemctl restart haproxy^C
# systemctl enable haproxy^C
# firewall-cmd --permanent --add-service=http^C
# firewall-cmd --reload ^C
```