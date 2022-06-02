# RDS 
- Amazon RDS Service는 사용자들에게 인스턴스를 DBMS와 연결된 상태로 서비스를 제공한다.
- DB의 구성, 운영, 관리의 편의성
- 고가용성의 구성 가능 / 확장기능을 제공

## DB
- 데이터를 체계적으로 저장/검색/사용 하기 위한 방식
- DB를 관리하기 위해 DBMS를 사용한다
- DBMS : SQL
- 관계형 데이터 베이스(RDBMS)는 2차원 테이블 형태로 구성된다.

## RDS-Web 구성
1. 새로운 VPC 생성

2. 보안 그룹 생성
	- 보안그룹 1 MySQL : 3306 포트 : 모두에게 허용
	- 보안그룹 2 SSh : 22 포트 : 모두에게 허용 / HTTP : 80번포트 : 모두에게 하용

3. RDS 메뉴 목록에서 서브넷 그룹 생성
	- 이름 및 VPC 선택
	![화면 캡처 2022-06-02 222432](https://user-images.githubusercontent.com/57117748/171639544-6f3c49f9-2c1f-46c9-ab6a-4fa299126b37.png)

	- 가용영역 및 서브넷 지정 (VPC 의 가용영역 전부 선택, private 서브넷 2개 지정)
	![화면 캡처 2022-06-02 222449](https://user-images.githubusercontent.com/57117748/171639643-3368bb42-2435-4044-9a29-996d095c04b0.png)

4. RDS 생성 
	```
	- 방식 : 표준
	- 엔진유형 : mariadb
	- 버전 : 기본값(최신버전)
	- 인스턴스 구성 / 스토리지 / 가용성 기본값
	- 연결 : 위에서 만든 VPC 및 서브넷그룹 지정
	- 퍼블릭 액세스 : 아니오
	- 보안그룹 : db-sg (2번에서 만든 그룹)
	- 추가구성 항목에서 초기 데이터베이스 이름 : sample (이 외 기본값)
	```

5. EC2 생성
	```	
	- 이미지 : 기본(amazon)
	- 키페어 : 기존 지정 혹은 새로 생성
	- 네트워크설정에서 편집버튼 클릭	
		*이전에 만든 VPC 및 public 서브넷 지정
		퍼블릭IP 자동할당 활성화
		web-sg 보안그룹 지정
	- 생성
	```

6. Web 구성(EC2)
	- 인스턴스 연결 버튼 혹은 ssh 로 원격접속 

	![화면 캡처 2022-06-02 224043](https://user-images.githubusercontent.com/57117748/171642599-89b0e656-896c-43c2-b9bf-594ca250b51b.png)


	- 패키지 업데이트 및 설치
	```
	$ sudo -i 		// 관리자 모드 진입
	# sudo yum -y update	// 패키지 최신화
	# sudo yum -y install httpd php php-mbstring php-pear php-fpm php-mysql
	# systemctl enable --now httpd
	# systemctl enable --now php-fpm
	```

7. 네트워크 확인 	
	```
	# echo '<?php phpinfo(); ?> > /var/www/html/index.php'
	# systemctl enable --now httpd
	```
	* 퍼블릭 IP으로 네트워크 확인

	- 서비스를 제공할 스크립트 작성 DB.php 파일 구성
	```
	<?php

	$conn = mysqli_connect('[RDS 엔드포인트]', '[RDS DB계정명]', '[RDS DB PW]', '[RDS DBNAME]');

	$result = mysqli_query($conn, "create table test_tab (id int(2) , name varchar(5))");
	$result = mysqli_query($conn, "insert into test_tab values(1,'choi')");
	$result = mysqli_query($conn, "select * from test_tab");

	while($row = mysqli_fetch_array($result)) {
	    echo $row['id'].'  '.$row['name'].'<br>';
	  }

	?>
	```

8. httpd 재실해
	```
	# systemctl enable --now httpd
	```

9. DB 연결 확인
	- 퍼블릭IP/db.php

# Route 53
- AWS에서 제공하는 DNS 서비스
- DNS : 장비들의 IP 주소와 이름주소를 매핑해주는 서비스
- 도메인 주소를 직접 구매하거나 혹은 기존의 도메인 주소를 따로 등록시킨다.
- 가용성 및 확장성이 우수한 DNS 웹 서비스
- 사용하는 만큼 지불
- 내/외부로 라우팅 시에도 사용한다.
- IPv6와도 완벽 호환

## 라우팅 방식
1. 지연 시간 기반 라우팅
	- 여러 데이터센터에 EC2 리소스가 있고 그 중 가장 지연시간이 적은 리소스로 Route 53에서 DNS 쿼리 응답처리

2. 가중치 기반 라우팅 
	- 여러 데이터센터의 EC2 연결 시스템에 대한 가중치 비율 조정 가능

3. 지역 기반 라우팅
	- 요청이 시작된 지리적 위치를 기반으로 특정 엔드포인트에 라우팅을 수행하는것


## Route 53 사용

1. 등록된 도메인 -> 도메인 등록
	![화면 캡처 2022-06-02 181037](https://user-images.githubusercontent.com/57117748/171596862-890099cd-1eb7-48a6-b619-21a2a7ec12ed.png)

2. 도메인 이름 입력 및 장바구니 추가 후 계속
	![화면 캡처 2022-06-02 181002](https://user-images.githubusercontent.com/57117748/171596995-56ed5bcf-cb1d-4817-a141-c55f9a954bfe.png)

3. 등록 연락처 작성
	- 기본 영문으로 작성한다.
	![화면 캡처 2022-06-02 181522](https://user-images.githubusercontent.com/57117748/171597748-8dc2d5d1-7f98-448b-82de-7e5b6558641c.png)

4. 구매 완료 후 확인
	- 구매완료 수분 후 등록 연락처상의 이메일로 등록 완료 메일 확인

5. EC2 서비스 메뉴에서 탄력적 IP 주소 메뉴
	- 탄력적 IP 주소 할당 
	![화면 캡처 2022-06-02 182023](https://user-images.githubusercontent.com/57117748/171598683-1d080f65-3609-4e1d-a830-e7ae0f455196.png)
	- EC2와 연결
	![화면 캡처 2022-06-02 183905](https://user-images.githubusercontent.com/57117748/171602254-8e11e72b-8d01-4df2-ad11-d9e4dbc83174.png)
	![화면 캡처 2022-06-02 183932](https://user-images.githubusercontent.com/57117748/171602287-ebdb87a7-89b0-480c-b4a4-153ffe81f724.png)

6. 호스팅 영역 
	- Route 53 호스팅 영역에서 도메인 선택

7. 레코드 선택
	- 레코드 생성을 클릭하고 '항목'에 www를 입력, '값'에 탄력적 IP 주소 입력

8. 확인 
	- 신듀 도메인 페이지에서 확인 

# 로드 밸런싱
- 네트워크의 트래픽을 고의로 분산 시켜 서버의 부하를 미연에 방지한다.
- 서버의 부하는 시스템 성능의 하락 또는 네트워크 트래픽의 증가로 나타난다.
	1. Scale Up : 성능 향상(컴퓨터 업그레이드)
	2. Scale Down : 시스템 개수 증가 (부하 분산 : 로드밸런싱)

## 로드밸런싱의 방식
1. Roundrobin : 다수의 시스템에 번갈아 가면서 트래픽을 전송
2. Hash : 한번 연결을 시키면 같은 서버로 계속 연결
3. Least Connection : 가장 트래픽이 적은 시스템부터 트래픽을 전송

## ELB 
- AWS 에서 제공하는 로드 벨런스 서비스
- 고가용성 / 확장성 / 성능 / 보안성 
- 자동 조정 (인스턴스 개수)
- 종류
	* Application - http / https 서비스에 대한 기능(L7)
	* Network - TCP 기반 로드밸런싱 기능 (L4)
	* Gateway - third-part 방식의 네트워크 구성에서 사용
	* Classic - 이전 방식
- 특징
	* 상태 확인(health-check) 기능 제공
	* sticky session 기능
	* 고가용성 구성
	* SSL Termination 및 보안 기능 (ELB 자체에 인증서 설정이 유리 / ACM 서비스 사용 시 추가 비용 및 관리 X)

# 로드밸런서 구성
1. VPC 준비

2. 인스턴스 준비 (http 구성)
	- Amazon Linux 이미지로 생성
	- 패키지 업데이트 (yum -y update // sudo -i 를 통해 root로 구성)
	- httpd 구성
	- /var/www/html/index.html 생성
	- restart, enable
	![화면 캡처 2022-06-02 193006](https://user-images.githubusercontent.com/57117748/171612000-55a04892-95db-4af8-9158-8853ed482b1a.png)


3. 구성한 인스턴스로 이미지 생성
	- AMI를 통한 이미지 생성
	- /var/www/html/index.html 파일 수
	- restart, enable
	![화면 캡처 2022-06-02 193802](https://user-images.githubusercontent.com/57117748/171612047-1ac2f218-d850-4e99-b8f6-b535db672b91.png)

4. EC2 대상그룹 항목에서의 설정
	-  create target group 버튼으로 생성
	![화면 캡처 2022-06-02 194239](https://user-images.githubusercontent.com/57117748/171612769-fd17767b-21d9-4649-99bd-4c9b4cfab1bb.png)

	- Target group name 에서 그룹이름 지정

	- VPC 선택

	- health-check 방식 지정 (http / https)
	![화면 캡처 2022-06-02 194344](https://user-images.githubusercontent.com/57117748/171612920-13f4c417-3ad9-4ab4-acf8-ea275c5090fe.png)

	- 인스턴스 선택 후 create 버튼 클릭
	![화면 캡처 2022-06-02 205825](https://user-images.githubusercontent.com/57117748/171624388-e92d797f-ba3d-4121-903b-657246eb3174.png)


5. 로드 밸런서 항목에서의 설정 
	- 생성 버튼 클릭

	- 종류 선택 (Application Load Balancer)

	- 로드밸런서 이름 입력

	- Scheme 및 IP 주소 방식 선택	(internet-facing : 외부 / internal : 내부)
	![화면 캡처 2022-06-02 194804](https://user-images.githubusercontent.com/57117748/171613574-ad5bbe26-d4c9-492b-a9c6-bdb4ccadc7d9.png)

	- 네트워크 설정 (해당 VPC / 가용영역 / 서브넷)
	![화면 캡처 2022-06-02 194928](https://user-images.githubusercontent.com/57117748/171613810-571766e2-890f-4c2b-af3b-0170381f99c1.png)

	- 보안그룹 설정
	![화면 캡처 2022-06-02 195615](https://user-images.githubusercontent.com/57117748/171614794-ea56b4d9-577b-4e19-b339-1499ea7f7b6a.png)


	- 리스너 항목에서 포트 번호와 타겟그룹 지정
	![화면 캡처 2022-06-02 195527](https://user-images.githubusercontent.com/57117748/171614702-0888702c-4825-4e0e-a430-bd9ba9e7ce88.png)

