# 스토리지 
- 데이터를 저장하는 저장공간 및 장치
- 저장 시 데이터 손상에 대비한 저장장치 백업 필수
- 스냅샷 기능 사용 가능
	* 기본은 데이터의 복제를 위한 용도 하지만 그것으로 백업/복원도 가능함

## 클라우드에서의 스토리지 종류
- 블록 스토리지 : EBS
- 오브젝트 스토리지 : S3
- 파일 시스템 기반 스토리지 : EFS

# S3 (Simple Storgae Service)
- 무한대로 저장 가능 
- 저장하는 최상위 레벨을 bucket 이라고 일컫음 
- bucket에 각각의 파일들을 Object라고 부름
- Public URL 로 접속가능 아니라면 / Private 직접 접근만 가능

## S3 써보기
1. 버킷 생성
	- Public 설정 (ACL 활성화 / 체크박스 해제)
	- Private 설정 (ACL 비활성화 / 체크박스 그대로)

	![화면 캡처 2022-05-31 210713_LI](https://user-images.githubusercontent.com/57117748/171172006-62c5058a-24bd-4252-abb3-b88f24c309eb.jpg)
	![화면 캡처 2022-05-31 210910_LI](https://user-images.githubusercontent.com/57117748/171172061-9a2177f2-7dd3-4c4d-9e51-60146d66b366.jpg)

2. 속성 확인 

3. 객체 업로드
	- 각 하나씩

4. 해당 파일(객체)의 속성에서 URL 주소 확인 후 접속
	- private : access denied
	- public with no ACL : access denied
	- public with ACL : access

	![화면 캡처 2022-05-31 211821](https://user-images.githubusercontent.com/57117748/171172478-d0703762-d077-413c-8d20-9787b1aaf68f.png)


5. 해당 파일(객체) 파일의 권한에서 ACL 퍼블릭 억세스 읽기 허용
	![화면 캡처 2022-05-31 211835](https://user-images.githubusercontent.com/57117748/171172406-ab5915b4-5a30-4b53-8ba9-3d72e04c9c70.png)

6. Public 버킷에서 ACL 수정 Private 설정
	
7. Private 버킷에서 퍼블릭으로 설정해보기. (안되는게 맞음)
	![화면 캡처 2022-05-31 212659](https://user-images.githubusercontent.com/57117748/171173074-8f527009-2907-413c-9921-4b781072b540.png)
	![화면 캡처 2022-05-31 212723](https://user-images.githubusercontent.com/57117748/171173107-2a6241f9-4a5f-4ffc-88a9-46b1da0d7cde.png)


## S3를 이용한 시스템 백업
1. S3 버킷 생성

2. IAM 사용자 생성 및 정책 설정 혹은 기존의 IAM 사용자 사용

3. .csv 파일 다운로드 & 등록
	- 보안 서비스 항목에서 IAM 선택
	- 사용자 탭에서 사용자 이름 선택
	- 하위 탭의 보안 자격 증명 탭 선택
	- 해당 항목에서 '액세스 키 만들기' 버튼 클릭
	- .svc 파일 다운로드 버튼으로 다운받고 닫기 (만약 다운로드 하지 않는다면 다시 확인 불가)

4. Linux에 CLI 설치	/ [awscli 가이드](https://docs.aws.amazon.com/ko_kr/cli/latest/userguide/getting-started-install.html)
	
	- 설치 파일을 다운로드
	```
	$ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
	```
	![화면 캡처 2022-05-31 221632](https://user-images.githubusercontent.com/57117748/171182651-c8ead031-ab72-46ac-ab06-e1cd2f1055e3.png)


	- 압축을 풀기
	```
	$ unzip awscliv2.zip
	```
	![화면 캡처 2022-05-31 221643](https://user-images.githubusercontent.com/57117748/171182723-a6b2fcdc-8d96-4e6e-9185-fe9afff66eb5.png)
	![화면 캡처 2022-05-31 221715](https://user-images.githubusercontent.com/57117748/171182818-53e0e01e-b382-463b-9a7f-85fd54ef9d4a.png)


	- 설치 프로그램을 실행
	```
	$ sudo ./aws/install
	```	
	![화면 캡처 2022-05-31 221731](https://user-images.githubusercontent.com/57117748/171182862-cbb087e4-a52a-42a4-93dd-ff7f51328189.png)


	-설치 확인
	```
	$ aws --version
	```
	![화면 캡처 2022-05-31 221743](https://user-images.githubusercontent.com/57117748/171182960-348d7d0c-7aba-4faa-8273-ea6ef82f3273.png)


	- 사용자 설정 등록
	```
	$ aws configure
	
	
	* 이때 IAM에서 다운받은 엑세스 키의 정보를 바탕으로 입력한다
	* 그 밖에 Region 과 Format (Ex ep-northeast-2 / json)
	```


	- 디렉토리에서 파일 내용으로 확인(~/.aws)
	```
	ls ~/.aws
	```
	![화면 캡처 2022-05-31 223314](https://user-images.githubusercontent.com/57117748/171185889-5695d4e3-0efd-4b30-8dee-86b5f061e6bf.png)


	- AWS 명령 수동 동기화
	```
	# rdate -s time.bora.net //  시간 동기화 때문에 어러가 날 수 있음으로 linux 시간 설정

	# aws s3 sync [디렉토리이름] s3://[버킷이름/폴더이름]

	# aws s3 cp s3://[버킷이름/폴더이름/파일이름] [디렉토리이름]
	```
	![화면 캡처 2022-05-31 225037](https://user-images.githubusercontent.com/57117748/171189645-01705eee-213f-4bd3-afa1-bf68465d2632.png)

6. bucket 에 가서 확인
	![화면 캡처 2022-05-31 225102](https://user-images.githubusercontent.com/57117748/171189755-bbab78e4-6ece-4bc3-9694-d03457077148.png)

7. 배치파일 Scheduling 자동 동기화 (crontab 설정 방식 사용)
	```
	# Every monday of week start backup to s3
	0       0       *       *       1       aws s3 sync ~/backup s3://test-backup-first-1/backup
	```
	![화면 캡처 2022-05-31 230341](https://user-images.githubusercontent.com/57117748/171192493-69d05c28-77e7-42dd-b6b4-22ff9167e4ba.png)


# Virtual Networks
- Network : 원격으로 데이터를 주고 받기 위한 인터넷 연결 망
- Protocal : 네트워크를 구성을 위한 규칙 

## VPN (Virtual Private Network)
- 보안성이 높은 사설 네트워크 구성 또는 네트워크 우회를 위한 가상 사설 네트워크

# VPC (Virtual Private Cloud)
- Amazon 에서 제공하는 가상 사설 클라우드로 쉽게 말하면 개인화된 가상 네트워크이다 즉, 사용자가 구성요소들을 이용해 원하는 형태로 네트워크 망을 구성한다.
- 논리적으로 격리된 네트워크

## Amazon VPC 구성요소
1. Private IP 주소
	- VPC 내부에서만 사용 할 수 있는 주소, 외부 인터넷은 이 IP로 접근 불가
	- VPC에서 시작된 인스턴스 서브넷 범위에서 자동으로 할당
	- 동일 네트워크의 인스턴스 간 통신이 가능
	- 보조 private IP 할당 가능

2. Public IP 주소
	- 인터넷을 통해 나아갈수있는 IP 주소
	- EC2 생성 시 옵션으로 Public IP 사용 여부 결정
	- 후에 인스턴스에서 수동으로 연결 및 해제 불가
	- 인스턴스 재부팅시 자동 할당 (고정 할 수 없음)

3. Elastic IP 주소
	- 동적 컴퓨팅을 위해 사용되는 고정 퍼블릭 IP 주소
	- VPC의 인스턴스 및 네트워크 인터페이스에 Elastic IP 할당 가능
	- 다른 인스턴스로 매칭 변경 가능
	- 효율적 활용을 위해 실행중인 인스턴스와 연결되어 있지 않거나, 중지된 인스턴스, 분리된 인터페이스에 연결시 요금이 부과되며 5개로 제한됨 

## VPC와 서브넷
- VPC는 개인화된 사설 가상 네트워크를 의미하며, 리전의 모든 가용영역에 적용됨
- 서브넷 : VPC를 목적에 따라 특정 범위로 나눈 범위들
- 가용영역에 하나 이상의 서브넷을 추가할 수 있으나 여러 가용영역으로 확장이 불가하다.
- VPC와 서브넷의 크기	: 일반적으로 CIDR 형태로 지정

1. Public Subnet
	- 네트워크 트래픽이 게이트웨이로 라우팅 되는 서브넷
	- 인터넷 망을 통해 서비스를 수행하는 인스터스는 퍼블릭 서브넷에 연결

2. Private Subnet
	- 네트워크 트래픽이 게이트웨아로 라우팅되지 않는 서브넷
	- 인터넷에 직접 연결할 필요가 없고, 높은 보안이 요구되는 DB 등을 연결

## 라우팅 테이블
- 아웃바운드 트래픽(네트워크 트래픽)을 전달할 위치가 명시된 규칙 집합 테이블. 

## NAT Gateway
- Network Address Translation Gateway
- Private Subnet에 있는 인스턴스들의 네트워크 주소 변환을 통해 인터넷 통신을 연결하는 게이트웨이.
- 기본적으로 인터넷 연결을 지양하는 프라이빗에 있더라도 업데이트, 패치 등을 위해 인터넷 게이트웨이에 연결해 인터넷을 할 수 있다.
- NAT 인스턴스로 대체가 가능하. : 퍼블릭 서브넷에서 직접 관리하는 인스턴스
* NAT Gateway 사용요건
	1. NAT Gateway를 생성하기 위한 퍼블릭 서브넷을 지정
	2. NAT Gateway와 연결할 Elasitc IP 주소
	3. NAT Gateway를 생성 후 인터넷 트래픽이 NAT Gateway로 통신이 가능하도록 프라이빗 서브넷과 연결된 라우팅 테이블 설정 수정

## VPC Endpoint 
- NAT, IGW 등을 거치지 않고 AWS의 서비스를 비공개로 연결해 가능하게 하는 서비스 
- 굳이 AWS 서비스를 이용할때에 직접 복잡한 라우팅을 거치는 부분 해소

## 보안그룹 / 네트워크 ACL
- IP와 포트 기준으로 통신을 허용 및 차단하기 위한 기능

1. 보안그룹
	- 인스턴스 레벨의 접근 규칙 설정 : 인스턴스에 보안그룹을 적용해 설정
	- Allow 만 설정 가능 (Deny 불가)
	- 반환 트래픽에 대한 허용

2. Network ACL
	- 서브넷 레벨의 접근 규칙 설정 : 연결된 서브넷의 모든 인스턴스에 자동 적용
	- Allow / Deny 규칙 설정 가능
	- 반환 트래픽에 대한 허용 설정 필요

## VPC 피어링
	- 서로 다른 VPC 간 트래픽 라우팅


## VPC 설정
1. 서울 리전에서 VPC 생성 (10.0.2.0/24)
 	- AZ = 2
 	- 퍼블릭 1 / 프라이빗 2 가용 영역 마다
 	- NAT GW 및 Endpoint 설정 하지 않는다.
 	![화면 캡처 2022-06-01 142715](https://user-images.githubusercontent.com/57117748/171334009-f00eb04c-6b96-4c41-86ed-57ff8441c65e.png)
 	![화면 캡처 2022-06-01 142750](https://user-images.githubusercontent.com/57117748/171334084-eef4a41c-7862-44a0-b595-175a30f8338c.png)

2. 확인
	![화면 캡처 2022-06-01 142144](https://user-images.githubusercontent.com/57117748/171334241-38e62490-8903-4609-a7e4-e24941b12f19.png)


3. 버지니아 북부 리전에서 VPC 생성 (10.0.10.0/24)
 	- AZ = 2
 	- 퍼블릭 1 / 프라이빗 2 가용 영역 마다
 	- NAT GW 및 Endpoint 설정 하지 않는다.

4. 확인 

5. 양쪽의 VPC 피어링
 	- 피어링 연결 생성
 	![화면 캡처 2022-06-01 143512](https://user-images.githubusercontent.com/57117748/171334873-afc13c1b-cb01-4528-881f-4f05b3835db0.png)


 	- 요청자 / 수락자 VCP ID 설정 (Ex 요청자 : 버지니아 북부 리전에서 VPC / 수락자 : 서울 리전에서 VPC)
 	![화면 캡처 2022-06-01 143303](https://user-images.githubusercontent.com/57117748/171334685-d093fada-c264-4118-928f-27065017acb4.png)


 	- 수락자 리전에서 피어링 연결 수락하기
 	![화면 캡처 2022-06-01 142258_LI](https://user-images.githubusercontent.com/57117748/171334809-61355c1c-5192-4902-b559-a5ccc0c318e3.jpg)
