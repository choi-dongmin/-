# 로드밸런서 상태 체크
- 로드밸런서의 자재 상태 및 백엔드에 대한 정보를 체크를 하는 방법은 2가지가 있다.

## 기본적인 로드 밸런싱 설정
1. 패키지 설치
	```
	# yum -y install haproxy
	```

2. 서비스 설정 파일 백업 및 수정
	```
	# cp /etc/haproxy/haproxy.cfg /etc/haproxy/haproxy.cfg.backup	// 설정파일 백업
	# vi /etc/haproxy/haproxy.cfg	// 설정파일 백업
	```

	```
	frontend lb	
	        bind *:80		// 로드 밸런서 IP:Port
	        default_backend web
	        option  forwardfor
	backend web
	        balance roundrobin		// roundrobin 형식의 밸런싱
	        server  web01 10.0.2.10:80 check		// 웹 서버 IP 1
	        server  web01 10.0.2.15:80 check		// 웹 서버 IP 2
	```
	![화면 캡처 2022-05-25 222038](https://user-images.githubusercontent.com/57117748/170271860-5ee2f9e8-a905-4741-ab15-9448e7abb209.png)

3. haproxy 실행 및 http 방화벽 해제
	```
	# systemctl restart haproxy^C
	# systemctl enable haproxy^C
	# firewall-cmd --permanent --add-service=http^C
	# firewall-cmd --reload ^C
	```

- 브라우저를 통한 확인
	1. 기본적인 로드 밸런싱 설정 
		
	2. /etc/haproxy/haproxy 설정 수정
		```
		# vim /etc/haproxy/haproxy.
		```

		```
		frontend loadbalancer
		        bind *:80
		        stats enable
		        stats auth admin:123	// 해당 페이지 접속시 아이디 및 비밀번호
		        stats show-node	
		        stats refresh 60s
		        stats uri /haproxy?stats	// uri
		```

		```
		# systemctl restart haproxy
		```
		![화면 캡처 2022-05-26 123836_LI](https://user-images.githubusercontent.com/57117748/170448300-48e52338-afaf-4eb7-8456-497a24c4582c.jpg)

- 패키지를 통한 터미널 내에서 확인
	1. socat 패키지 설치
		```
		# yum install -y socat
		```

	2. /etc/haproxy/haproxy 설정 수정
		```
		# vim /etc/haproxy/haproxy.cfg
		```

		```
		global
			stats socket /var/lib/haproxy/stats
		```
		![화면 캡처 2022-05-26 173135](https://user-images.githubusercontent.com/57117748/170450494-0e092b52-228d-4316-a320-b95bd8c4dd08.png)

		```
		# systemctl restart haproxy
		```

	3. 정보 확인
		```
		# socat /var/lib/haproxy/stats stdio
		show stat
		show info
		.
		.
		.
		``` 
		![화면 캡처 2022-05-26 173825](https://user-images.githubusercontent.com/57117748/170451754-feed9862-7fc5-419c-9da5-a44f413bf6bb.png)


# 로드밸런서에 웹 인증서(SSL) 설정 및 연동
- 로드밸런서 자체에 인증서를 구성할 수 있다.

## 구성 순서
```
1. 기본적인 로드밸런서 구성
2. 인증서 준비
3. 설정에서 인증서 등록
```

1. 구성된 로드 밸런서에 인증서를 준비 및 해당 인증서 퍼미션 설정
	```
	# openssl req -x509 -nodes -newkey rsa:2048 -keyout /etc/pki/tls/certs/haproxy.pem -out /etc/pki/tls/certs/haproxy.pem 	// 개인키와 인증서를 합쳐서 생성
	```
	```
	# chmod 600 /etc/pki/tls/certs/haproxy.pem 	// key에게 퍼미션 설정
	```
	![화면 캡처 2022-05-26 175732](https://user-images.githubusercontent.com/57117748/170455039-40eeb95a-4ec8-4ec4-a435-7cdf59c88183.png)
	![화면 캡처 2022-05-26 175905](https://user-images.githubusercontent.com/57117748/170455305-eabe0d1a-9297-431b-9e97-e73e227907bd.png)

2. /etc/haproxy/haproxy.cfg 설정 
	```
	# vi /etc/haproxy/haproxy.cfg
	```

	```
	global
	    maxsslconn     40					// 추가
	    tune.ssl.default-dh-param 2048		// 추가
	.
	.
	.
	frontend lb
        bind *:443 ssl crt /etc/pki/tls/certs/haproxy.pem
	```
	![화면 캡처 2022-05-26 145601](https://user-images.githubusercontent.com/57117748/170457090-3e7edcf7-f4e6-4c9a-884c-1267d8a729ab.png)
	![화면 캡처 2022-05-26 145601](https://user-images.githubusercontent.com/57117748/170460268-fa699859-f63c-4408-ae59-3bdbe6fb034d.png)

3. 재실행 및 방화벽 해제
	```
	# systemctl restart haproxy
	# systemctl enable haproxy	
	# firewall-cmd --permenent --add-service https
	# firewall-cmd --reload
	```

4. 확인
	```
	curl -k https://[로드밸런서 IP]
	```
	![화면 캡처 2022-05-26 153401_LI](https://user-images.githubusercontent.com/57117748/170460490-d4656078-8322-4d38-ae6d-40f89dfd16ba.jpg)


# 로드 밸런스 로그 설정
- 로드 밸런스 발생하는 이벤트 로그를 확인하기 윈한 로그파일 설정

1. vi /etc/rsyslog.conf 설정
```
.
.
.
######MODULE#####
$ModLoad imudp			// 주석 해제
$UDPServerRun 514		// 주석 해제
$AllowedSender UDP, 127.0.0.1	// 모듈 영역에 추가
.
.
.
######RULE######
*.info;mail.none;authpriv.none;cron.none;local2.none    /var/log/messages	// local2.none 추가

# HAProxy log
local.*			/var/log/haproxy.log

```
![화면 캡처 2022-05-26 183753](https://user-images.githubusercontent.com/57117748/170461889-b9c23cb0-16dc-4252-9fd1-7f7376052005.png)
![화면 캡처 2022-05-26 184149](https://user-images.githubusercontent.com/57117748/170462794-7f09208f-b917-4b36-b66b-c043fdc5ce03.png)


2. 서비스 재시작
```
# systemctl restart rsyslog	
```

# L4 로드밸런서로 MariaDB 로드 밸런싱
- HAProxy 의 기본방식은 L7 http/https 로드 밸런서로서의 기능
- L4 밸런싱을 이용해 DB 또한 로드 밸런싱이 가능하다.
- TCP/UDP 각 포트 번호에 따라 부하분산 설정 가능.

## 로드 밸런서와 DB 연동

1. 데이터베이스 서버 2대 및 원격작업 가능한 사용자 및 테이블 준비
- 사용자 생성 및 테이블 권한 부여
```
> GRANT select,insert ON *.* TO remote@'%' identified by '1234';	// 어디서나 접근 가능한 remote 계정은 모든 DB와 모든 테이블에 select,insert 권한 부여 패스워드는 1234
> FLUSH PRIVLIGES;
```

2. vi /etc/rsyslog.conf 설정
```
global
        log             127.0.0.1 local2
        chroot          /var/lib/haproxy
        pidfile         /var/run/haproxy.pid
        maxconn         255
        user            haproxy
        group           haproxy
        daemon

defaults
        mode            tcp
        log             global
        timeout connect 30s
        timeout server  60s
        timeout client  60s

frontend loadbalancer_db
        bind *:3306
        default_backend mariadb

backend mariadb
        balance roundrobin
        server  db01    10.0.2.20:3306  check
        server  db02    10.0.2.30:3306  check

```

3. SELinux 정책 설정 및 재시작
```
semanage boolean -m haproxy_connect_any --on
systemctl restart haproxy
```

4. 확인
```
mysql -u remote -h 10.0.2.25[로드밸런서 IP] -p
```

## 키워드
- 로드 밸런스 : 트래픽 과부화를 막기위한 분산 서비스로 일반적으로 L7에서 그외에는 L4 에서 동작한다.

- SSL : 인증서를 통해 암호화를 구성하는 방법
