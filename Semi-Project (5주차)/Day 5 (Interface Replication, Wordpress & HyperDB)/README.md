# interface 이중화
- 기존에는 하나의 인터페이스에 하나의 IP 주소를 할당 및 사용
- 기존에는 인터페이스 문제 발생시 서비스 제공에 지장
- 기존에는 인터페이스 주소가 다르면 접근이 어려움
- 인터페이스 여러개를 묶어 사용하는 형태

## 티밍
- RHEL 7 부터 제공하기 시작함 
- 인터페이스의 대한 장애 대비
- 효율성을 높임
- IP 주소에 대한 부담감 감소
- 여러 인터페이스에 대해 IP 주소 하나만 설정
- 그러나 사용자에 따라서 인터페이스의 낭비로 볼 수 있음

## Runner 
- 티밍 구성시 동작 방식을 지정 5가지 종류
1. boradcast : 각각의 모든 인터페이스에게 패킷을 전달
2. roundrobin : 인터페이스마다 한번씩 번갈아 가면서 패킷을 전달	--- 부하분산
3. loadbalance : 처리량(상태)에 따라 부하 분산기능을 제공 		--- 부하분산
4. activebackup : 장애복구 방식의 백업 형식의 기능 제공 		--- 백업형식
5. LACP : 네트워크 장비중 switch 에서 설정한 방식으로 동작

## 티밍 구성방법 
- 구성 순서
```
정적 아이피 할당
1. 마스터 인터페이스(팀인터페이스) 를 생성 (가상인터페이스)
2. 포트 인터페이스를 연결	-> 실제 인터페이스
	=> DHCP 서버에서 IP 주소를 할당받음

동적 아이피 할당
1. 팀 인터페이스 생성
2. 팀 인터페이스에 IP 주소를 정적으로 입력
3. 포트 인터페이스 연결
```

0. 연결 할 인터페이스 연결 해제 
```
# nmcli connection down [인터페이스 명]
```

1. 마스터 인터페이스 생성
```
# nmcli connection add type team con-name [team01 : 마스터 인터페이스] ifname [team01 : 생성할 가상 디바이스 이름] 
> config '{"runner": {"name": "[activebackup : 러너 타입]"}}'
```

2. 가상 인터페이스 네트워크 수정
```
# nmcli con mod team01 ipv4.addresses 192.168.56.100/24	// 지정 할 IP 주소 
# nmcli con mod team01 ipv4.gateway 192.168.56.1	// Gateway 주소 
# nmcli con mod team01 ipv4.dns 8.8.8.8		// DNS 주소
# nmcli con mod team01 ipv4.method manual 	// 정적 할당으로 바꾼다.
# nmcli con up team01
```

3. 가상 인터페이스를 통한 인터페이스 포트 연결
```
# nmcli connection add type team-slave con-name [team01-port1 : 가상 포트 명] ifname [enp0s8 : 디바이스명] master [team01 : 가상 인터페이스 이름]
```

* 설정한 러너를 변경하고 싶을때
```
# nmcli con mod team01 team.config '{"runner": {"name": "roundrobin"}}'
```

# Wordpress & HyperDB
- wordpress 의 구성 방식은 패키지 형식 또는 직접 설치하는 방식 2가지가 있다.
* HyperDB : DB의 장애, 로드 밸런싱 및 파티셔닝을 지원한는 데이터 베이스 클래스

## 패키지 형식 순서
- Wordpress repository에 연결이 되어 있어야 가능하다. 
- 버전에 대한 고려 비중이 낮다.


1. 패키지 다운로드 및 실행
```
# yum install -y httpd php php-mbstring php-pear
# systemctl enable --now httpd
# firewall-cmd --add-service=http --permanent
# firewall-cmd --reload
```

2. 별도 시스템에 MariaDB 구성 및 사용자 설정
```
# yum install -y mariadb-server

# systemctl restart mariadb.service
# systemctl enable mariadb.service
# firewall-cmd --add-service=mysql --permanent
# firewall-cmd --reload

# mysql_secure_installation
```
```
# mysql -u root -p

> CREATE database wp_db;
> GRANT all privileges ON wp_db.* TO remote@'%' identified by '1234';
> FLUSH PRIVILEGES;
```

3. Wordpress 구성 
```
# yum install -y epel-release
# yum install -y wordpress
```

```
# vim /etc/wordpress/wp-config.php
```
```
define( 'DB_NAME', 'wp_db' );
define( 'DB_USER', 'remote' );
define( 'DB_PASSWORD', '1234' );
define( 'DB_HOST', '[DB 네트워크 IP]' );
```

4. SELinux 해제 및 Wordpress 동작 확인
```
# semanage boolean -m httpd_can_network_connect_db --on
# systemctl restart httpd

* 확인은 외부 네트워크에서 해당 IP/wordpress
```

4. HyperDB plugin 다운로드 
```
# wget https://downloads.wordpress.org/plugin/hyperdb.1.8.zip
# unzip hyperdb.1.8.zip
```

5. HyperDB 설정
```
# vim hyperdb/db-config.php
```

-hyperdb/db-config.php
```
$wpdb->add_database(array(
        'host'     => DB_HOST,		// 해당 부분은 wp-config.php의 설정을 불러옴
        'user'     => DB_USER,		// 해당 부분은 wp-config.php의 설정을 불러옴
        'password' => DB_PASSWORD,	// 해당 부분은 wp-config.php의 설정을 불러옴
        'name'     => DB_NAME,		
));

/**
 * This adds the same server again, only this time it is configured as a slave.
 * The last three parameters are set to the defaults but are shown for clarity.
 */
$wpdb->add_database(array(
        'host'     => '[DB2 IP]',		// 해당 부분은 2번째 DB IP
        'user'     => 'remote',			// 해당 부분은 2번째 DB USER
        'password' => '1234',			// 해당 부분은 2번째 DB PW
        'name'     => 'db_wordpress',	// 해당 부분은 2번째 DB NAME
        'write'    => 0,
        'read'     => 1,
        'dataset'  => 'global',
        'timeout'  => 0.2,
));

```

5. 설정 파일위치 설정
```
# cp hyperdb/db-config.php /etc/wordpress/
# cp hyperdb/db.php /usr/share/wordpress/wp-content/
```

6. 재실행
```
# systemctl restart httpd
```

## 직접 설치 순서
1. Wordpress 내려받기 및 압축해제
```
# wget https://ko.wordpress.org/latest-ko_KR.zip 
(버전은 문서 확인 후 선택)
# unzip latest-ko_KR.zip
```

2. # wp-config-sample.php 수정
```
vim wordpress/wp-config-sample.php
```

```
define( 'DB_NAME', 'wp_db' );
define( 'DB_USER', 'remote' );
define( 'DB_PASSWORD', '1234' );
define( 'DB_HOST', '[DB 네트워크 IP]' );

- 위 처럼 저장후 wp-config.php 파일 이름 변경
```

3.  wordpress 디렉토리 파일들 /var/www/html 로 이동
```
# cp -r wordpress /var/www/html
```

4. httpd 재실행
```
# systemctl restart httpd
```

5. HyperDB plugin 다운로드 
```
# wget https://downloads.wordpress.org/plugin/hyperdb.1.8.zip
# unzip hyperdb.1.8.zip
```

6. HyperDB 설정
```
# vim hyperdb/db-config.php
```

-hyperdb/db-config.php
```
$wpdb->add_database(array(
        'host'     => DB_HOST,		// 해당 부분은 wp-config.php의 설정을 불러옴
        'user'     => DB_USER,		// 해당 부분은 wp-config.php의 설정을 불러옴
        'password' => DB_PASSWORD,	// 해당 부분은 wp-config.php의 설정을 불러옴
        'name'     => DB_NAME,		
));

/**
 * This adds the same server again, only this time it is configured as a slave.
 * The last three parameters are set to the defaults but are shown for clarity.
 */
$wpdb->add_database(array(
        'host'     => '[DB2 IP]',		// 해당 부분은 2번째 DB IP
        'user'     => 'remote',			// 해당 부분은 2번째 DB USER
        'password' => '1234',			// 해당 부분은 2번째 DB PW
        'name'     => 'db_wordpress',	// 해당 부분은 2번째 DB NAME
        'write'    => 0,
        'read'     => 1,
        'dataset'  => 'global',
        'timeout'  => 0.2,
));
```

7. 설정 파일위치 설정
```
# cp hyperdb/db-config.php /var/www/html
# cp hyperdb/db.php /var/www/html/wp-content/
```

8. 재실행
```
# systemctl restart httpd
```