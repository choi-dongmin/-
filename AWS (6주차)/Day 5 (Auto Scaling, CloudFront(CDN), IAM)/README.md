# Amazon Auto Scling 
- 클라우드의 장점을 극대화 시키는 기능
	* 서비스 구축 단계에서 예상 최대치의 인프라를 구축 할 필요가 없다
	* 성능 요구 증가시 신속한 대응이 가능하다

- 가용성
	* 서비스나 시스템이 가동 및 실행되는 시간의 비율을 뜻한다.
	* 측정 단위는 9로 측정한다.
	* 고가용성 시스템 : 가용성이 높다는 "절대 고장나지 않음"을 뜻한다. 서비스 다운 타임 시스템이 허용되지 않는 시스템.

- 확장성
	* 성능 요구 증가시 서비스나 응용 프로그램이 향상 될 수 있정도
	* 확장성이 높을 경우 유동적인 서비스 요구 증가에 유연하게 대응이 가능
	* 스케일 방식
		1. scale up/dwon : 단일 시스템의 하드웨어적 사양을 변경
		2. scale out/in  : 하드웨어 사양 변경 대신 시스템 수량 변경 

## Auto Scaling 의 특징
- 사용자가 정의한 조건에 따라 Scale out/down 이 용의하다
- 실행중인 EC2 인스턴스의 수량을 일정하게 유지가 가능하다
- Scale out : 수요가 급증하면 인스턴스의 수량이 증가한다 (단 최대수량 존재)
- Scale in : 수요가 감소하면 인스턴스의 수량이 그만큼 감소한다 (단 최소수량 존재) 
- Scale up/dwon 존재하지 않는다
- Auto Scale 사용은 무료이나 그로인해 발생한 scale out 은 비용이 발생한다.

## Auto Scling Group
- 인스턴스의 조정 및 관리 목적으로 구성된 논리적 그룹 
- Auto Scaling을 수행하는 인스턴스의 모음

## 시작 템플릿
- Auto Scaling Group 에서 인스턴스를 시작하기 위해 사용하는 템플릿
- 주요 구성요소
	1. AMI
	2. 인스턴스 유형
	3. 키페어
	4. 보안그룹
	5. EBS
- Auto Scaling Group 당 하나의 시작 구성을 지정하여 사용
- 시작 템플릿은 수정 불가
	* 변경이 필요할 경우 새로 시작해 Auto Scaling Group 업데이트

## Auto Scaling Gourp 조정
- Scale out/in의 규칙 모음으로 EC2 인스턴스의 시작 삭제까지 모든 동작에 대한 규칙을 정의

## 리소스 확장 전략
1. 가용성 기준 최적화
	* CPU/리소스 활성화 수준을 40% 설정

2. 가용성 비용 균형
	* CPU/리소스 활성화 수준을 50% 설정

3. 비용 기준 최적화
	* CPU/리소스 활성화 수준을 70% 설정

4. 커스텀 스케일링 전략
	* CPU/리소스 활성화 수준을 직접입력


## Auto Scaling 구성
실습 가이드
1. 로드밸런서 생성 
	- 인스턴스 생성 (httpd 서비스 구성 및 http/ssh 허용 보안그룹 생성)
	- 타겟그룹 생성
	- 로드밸런서 생성	(ALB)
	![화면 캡처 2022-06-03 194713](https://user-images.githubusercontent.com/57117748/171839826-708ea303-7668-407b-ae4c-dd08fe97d2cd.png)

2. 시작 템플릿 생성
	- 인스턴스 구성 (1번과 같은 웹 인스턴스 구성 / 서브넷 및 보안그룹 선택)
	- EC2의 시작 템플릿 탭에서(미리 만든 이미지/보안그룹/키페어 등으로 템플릿 구성 부하테스트를 위해 원격접속 가능하게 키페어)
	![화면 캡처 2022-06-03 195312](https://user-images.githubusercontent.com/57117748/171840645-f524b1dd-5fcf-4b67-95e8-0a8d8a94f2e8.png)
	![화면 캡처 2022-06-03 195049](https://user-images.githubusercontent.com/57117748/171840314-2c2a5bf5-815e-414a-9e2a-193737a0ec60.png)
	
3. 오토스케일링 구성
	- 이름 및 시작 템플릿 선택
	- VPC와 서브넷 선택(서브넷은 로드밸런서와 일치)
	![화면 캡처 2022-06-03 195456](https://user-images.githubusercontent.com/57117748/171840884-2b62331f-f368-4390-bb01-c6558a64f117.png)

	- 로드밸런서 연결 설정
	![화면 캡처 2022-06-03 195615](https://user-images.githubusercontent.com/57117748/171841062-89d91d86-93a2-4358-9d00-8b10540888de.png)

	- 인스턴스 및 로드밸런서에 대한 상태 모니터링 설정	
	![화면 캡처 2022-06-03 195646](https://user-images.githubusercontent.com/57117748/171841130-d368c298-07ee-48dc-b8fa-cd4f458e67f8.png)

	- 오토스케일링 정책 설정
	![화면 캡처 2022-06-03 195758](https://user-images.githubusercontent.com/57117748/171841308-ef84946e-557b-480e-9d9a-cab7e0cd4de3.png)

	- 알람/태크 설정(생략)

4. 확인 방법
	- 각 인스턴스에 webserver 접속해보기 (생성 직후)
	
	- 로드밸런서 주소로 접속해서 확인 

	- 인스턴스에 원격접속해서 부하 발생($ sha256sum /dev/zero) 

	- 일정 시간(상태확인유예시간) 이후 인스턴스 개수 확인
	![화면 캡처 2022-06-03 194605](https://user-images.githubusercontent.com/57117748/171841629-bc0674b7-c3bd-4dcc-9bb0-200a2430699f.png)

# CDN
- Contents Delivery Network
- 컨텐츠의 '빠른 전달'을 목적으로 데이터를 분산하여 제공
- 대용량 데이터를 전달하기 위해 필수적인 기능

## CDN 동작원리
- 사용자는 컨텐츠를 제공받기 위해 서버로 요청을 전달
- CDN 서비스는 컨텐츠를 제공하기 위해 사용자를 가까운 CDN 서버로 연결
- CDN 서버가 사전에 Caching 된 데이터를 사용자에게 전달
- CDN 서버가 데이터를 가지고 있지 않은경우 오리진(원본) 서버에게 파일을 요철
- 이후 요청한 데이터는 Caching 되어 이후 요청시 사용 

## CDN Caching 방식
- Static Caching 
	* 사용자의 요청없이도 오리진 서버의 컨텐츠를 운영자가 미리 복사
	* 동영상 서비스, 게임 다운로드 등 큰 파일에 적합한 방법

- Dynamic Caching 
	* 최초 Cache 서버는 컨텐츠가 존재하지 않음
	* 사용자가 컨텐츠 요청시 Caching 된 데이터가 있는지 확인
	* 데이터가 없으면 오리진 서버에 데이터를 요청해 Caching 저장
	* Caching 데이터는 수명에 따라 사용된 후 삭제

# Amazon CloudFront
- 짧은 지연시간 및 빠른 전송속도를 제공하는 CDN 서비스
- 데이터, 동영상, APP, API 등 다양한 컨텐츠를 지원
- AWS의 타서비스와 연계 가능


## CloudFromt의 특징
- 프로그래밍이 가능하고 안전한 콘텐츠 전송 네트워크 CDN
- 동/정적 컨텐츠 가속 서비스
- HTTP, HTTPS 서비스, Custom SSL 지원
- 커스텀 오류 응답
- 쿠키/헤더 오리진 서버 전달
- 다양한 통계 보고서 제공
- 업로드 가속
- 다양한 AWS 리소스를 오리진으로 지정 가능(S3 Bucket, EC2 Instance, ELB...)

## CloudFromt의 방식
- 사용자가 컨텐츠를 요청할 경우 DNS가 요청을 최적으로 서비스 할 수 있는 CloudFront Edge Location으로 요청을 라우팅
- CloudFront Edge Location에서 Cloudfront는 caching 된 데이터가 있는지 확인해 있는경우 사용자에게 제공
- 컨텐츠의 요청을 오리진 서버로 전달
- 오리진으로부터 데이터가 도착함과 동시에 사용자에게 전달

## 활용
- 다양한 보안 서비스
	* 웹 서비스 및 컨텐츠에 대한 보안 서비스 제공
		1. Amazon Shield를 통한 DDoS 공격을 차단
		2. Amazon Shield를 통한 일반적인 공격 방어 및 탐지/대응
		3. Amazon WAF를 통한 웹 트래픽 모니터링 및 차단
		4. Amazon Signed URL/Signed Cookie를 통한 컨텐츠 보호
		5. SSL 인증서 연동 환경 구성

- 비용 최적화
	* 전송 비용이 발생하는 AWS 내부 서비스에 적용 ((S3 Bucket, EC2 Instance, ELB...)
	* 오리진이 AWS내에 있는 전송비용이 발생하는 서비스에 CloudFront 사용시 기존 서비스와 네트워크비용 대신 CloudFront 비용만 발생

# CloudFromt 사용
1. S3 Bucket 만들기

2. bucket 폴더 생성 및 파일 업로드
	- 권한 설정 ACL 및 퍼블릭 설정
	![화면 캡처 2022-06-03 200944](https://user-images.githubusercontent.com/57117748/171842848-ed90780f-b9bc-441e-9f47-faf37a46f85f.png)

3. CloudFormat 생성

4. 배포 설정
	- 도메인 설정 및 옵션 설정
	![화면 캡처 2022-06-03 201157](https://user-images.githubusercontent.com/57117748/171843147-51bb23d7-0bcd-40b8-9b2a-5825bd9a111c.png)
	![화면 캡처 2022-06-03 201303](https://user-images.githubusercontent.com/57117748/171843305-e3109639-d470-45c9-b708-2ee971f0722c.png)

# IMA (Identity and Access Management)

- 유저를 생성, 관리하고 각 유저에 대한 접근권한을 설정
- 각 사용자가 어떤 서비스, 리소스를 사용하는지 정의]
- AWS내의 리소스에 대한 엑세스를 관리하는 서비스

1. SSO (Single-Sing-On)
	- 한 번의 인증으로 여러 시스템에 대한 인증이 가능한 솔루션
	- 하나의 계정이 다수의 서비스와 연결됨
	- 사용자 편의성이 증대되어 관리비용이 절감

2. EAM (Extranet Access Manangement)
- 미국의 정보기술 연구 및 자문회사인 가트너에 의해서 정의
- SSO를 기반한 접근관리 기능을 추가
- 보안 정책을 기반으로 권한을 부여

3. IAM (Identity and Access Management)
- EAM에 비해 확장된 개념
- 업무 흐름에 따른 보안 정책 수립 계정 관리등을 수행

## IAM을 통해 제어되는 권한
- 리소스를 관리하기 위한 관리 콘솔에 대한 접근
- AWS 내부 리소스에 대한 접근 권한
- 데이터에 대해 프로그래밍 방식(API)로 접근하는데 필요한 권한

## 역할을 사용한 임시 자격증명 관리
- EC2 인스턴스의 애플리케이션에 권한 부여
- 타 계정의 리소스에 엑세스
- AWS 서비스에 대한 엑세스

## IAM 구성요소
1. IAM 사용자
	- AWS에서 생성하는 개체
	- 서비스와 리소스와 상호 작용
	- 최초 IAM 사용자는 어떠한 권한도 있지 않음

2. IAM 그룹
	- IAM 사용자들의 집합
	- 그룹에 대한 권한을 지정함으로 더 쉽게 사용자 권한 지정 가능

3. IAM 역할
	- AWS 자격증명을 처리 하거나 권한 및 정책을 보유한다는 점에서 IAM 사용자와 비슷하다.
	- 하지만 IAM 역할은 한 사용자가 아니라 역할이 필요한 사용자, 그룹은 누구든지 연결 할 수있도록 설정

## IAM 서비스의 동작 방식
	![화면 캡처 2022-06-03 191606](https://user-images.githubusercontent.com/57117748/171835679-87052dc9-7ef5-489f-b84d-d46b13c471dc.png)
