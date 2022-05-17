# RPM / YUM
- 리눅스에서 소스코드를 binary 파일로 컴파일 하기 위한 패키지

## RPM (Redhat Package Menagement)
- 패키지 관리 도구로서 프로그램 설치를 가능하게 하는 패키지
- Ex windows exe 확장자 같은 역할
- 만약 패키지의 의존성이 있는 경우 선 패키지가 설치 되어 있지 않으면 설치 오류가 발생한다.

1. RPM 설치
- rpm 명령을 통해 RPM 패키지를 설치한다.

```
$ rpm -ivh [패키지이름]		// 해당 패키지 설치
$ rpm -Uvh [패키지이름]		// 해당 패키지 설치 및 업그레이드
```
![화면 캡처 2022-05-16 102344_LI](https://user-images.githubusercontent.com/57117748/168555038-3db68ff2-2ec2-42a7-81b0-3d7b909efb76.jpg)

- RPM 관련 명령어
1. -qa : 현재 설치된 패키지들의 목록 보기
2. -qa | grep [패키지명] : 확인하고자하는 패키지 확인
3. -q : 해당 패키지의 설치 확인
4. -qf [RPM 외 다른 파일] : 해당 파일이 속한 패키지 확인
5. -qi [패키지명]: 패키지의 상세 정보
6. -qip [rpm파일] : rpm 파일의 정보 확인
7. -qif [RPM 외 다른 파일] : 해당 파일 패키지의 상세 정보
8. -ql [패키지명] : 해당 패키지가 설치한 파일 목록들 확인
8. -qR [패키지명] : 해당 패키지의 의존성 확인 하기.

2. RPM 삭제
- rpm -e [패키지명] 을 통해서 삭제한다.

```
$ rpm -e [패키지명]
```

## YUM (Yellodog Update Modified)
- RPM의 단점(의존성 높음)을 보완하여 설치 할 수있는 컴파일 하기 위한 패키지.
- 주로 인터넷의 repository 를 이용하기 때문에 인터넷 연결만 된다면 모든 프로그램을 설치 가능
- 즉 yum 을 실행하면 repo에서 해당 패키지를 다운로드한다 이때 의존성이 있는 패키지라면 자동으로 필요한 패키지를 다운로드한다.
- /etc/yum.repo.d : yum repository 설정 디렉토리

```
$ yum install [패키지명] : 해당 패키지를 설치하는 명령이다
$ yum update [패키지명] : 해당 패키지를 업데이트하는 명령이다
$ yum remove [패키지명] : 해당 패키지를 삭제
```
![화면 캡처 2022-05-16 112613_LI](https://user-images.githubusercontent.com/57117748/168559719-f573ec30-2d8b-4c2e-92ef-35686f0aa9bc.jpg)
![화면 캡처 2022-05-16 112641_LI](https://user-images.githubusercontent.com/57117748/168559729-799064b5-98b9-493e-b543-c2a8e10909db.jpg)

- YUM의 명령어 

```
$ yum list				// repository에 있는 설치가 가능한 패키지 목록 확인
$ yum list installed	// 현재 설치된 패키지들 목록 확인
$ yum list update 		// 현재 설치된 패키지들 중 업데이트가 가능한 패키지 목록 확인
$ yum info [패키지명] 	// 해당 패키지의 정보확인
$ yum deposit [패키지명]	// 해당 패키지가 가진 의존성을 확인
$ yum groups list 		// 설치 할 수있는 그룹 패키지의 목록
$ yum groups info "그룹패키지" 		// 설치 할 수있는 그룹 패키지의 목록
$ yum groups install "그룹패키지"	// 설치 할 수있는 그룹 패키지의 목록
$ yum groups remove "그룹패키지" 	// 설치 할 수있는 그룹 패키지의 목록
```

- YUM 이벤트 확인

1. cat /var/log/yum.log

2. yum history
	- yum 에서 진행된 이벤트 확인
```
$ yum history		// yum 에서 진행된 이벤트 확인
$ yum history info [ID]	// 해당 ID 의 histiry info 확인
$ yum history rollback [ID]	// 해당 ID로 롤백
```

![화면 캡처 2022-05-16 183055](https://user-images.githubusercontent.com/57117748/168562881-089345bb-3227-4943-bc82-ccff16cbf1e3.png)


- repository 초기화 및 확인
```
$ yum repolist 						// 설정된 repository 출력
$ yum clean all 					// repositiry 초기화
```
![화면 캡처 2022-05-16 183922](https://user-images.githubusercontent.com/57117748/168564446-0afeda1c-4290-4833-bf58-9bd96f68ecbf.png)


* 참고 우분트의 apt 와 yum

1. apt apt-get에서 update는 저장소 정보 업데이트 / upgrade는 프로그램 업데이트

2. yum yum에서 update 는 프로그램 업데이트 / upgrade는 구식 패키지를 지우고 다시 설치

# SHH (Secure Shell) 
- 시스템과 독자적으로 구동되어 제공하는 프로세스 즉, 서비스이다.
- systemctl 서비스의 관리 시작 중지를 하는 명령
- /usr/lib/system.d/system 디렉토리에 서비스 유닛 파일들이 저장된다.
- ssh 같은경우 22번 port를 사용한다.

## ssh 서비스 구동 확인
1. systemctl 명령과 status 옵션 그리고 sshd 데몬 파일을 통해 서비스 구동 상태를 확인 할 수있다.
```
$ systemctl status sshd
```
![화면 캡처 2022-05-16 185152](https://user-images.githubusercontent.com/57117748/168566883-575dfad5-02dd-4ecc-88ba-3086724b757b.png)

2. netstat 을 통해 22번 포트가 열려 있는 상태를 확인 할 수있다.
```
$ netstat -an | grep 22
```
![화면 캡처 2022-05-16 185616](https://user-images.githubusercontent.com/57117748/168567751-745223a9-3ca1-4019-93ca-28975cfb6704.png)

## ssh 사용하기

- ssh 명령과 해당 계정의 이름 ip 주소를 통해 ssh 가 가능하다.

```
$ ssh root@10.0.2.15		// ip주소 10.0.2.15 의 관리자 모드로 진입

```

![화면 캡처 2022-05-16 122019_LI](https://user-images.githubusercontent.com/57117748/168568870-e70899b2-ca01-4f8a-9d53-79da1f539bb6.jpg)

* ssh root 접근제어
	- /etc/ssh/sshd_config 파일을 vi 편집기 수정을 통해 ssh 로 자신의 root 계정의 접근을 제어 할 수 있다.
	- /etc/ssh/sshd_config -> PermitRootLogin no 
	- 항상 vi 편집을 마친 후에는 서비스를 다시 실행한다.

```
vi /etc/ssh/sshd_config

systemctl restart sshd
``` 
![화면 캡처 2022-05-16 190557](https://user-images.githubusercontent.com/57117748/168569687-655124f8-f50e-4087-a39a-99854d8f6275.png)

## ssh 에서 비밀번호 없이 로그인하기 (key 기반 인증)
1. 클라이언트에서 ssh -keygen 을 통해 공개키와 개인키를 생성
```
# ssh-keygen	// 공개키와 개인키를 설정 /root/.ssh 디렉토리에 key 생성
```
![화면 캡처 2022-05-16 150949 (2)_LI](https://user-images.githubusercontent.com/57117748/168603426-621d707b-9310-4635-9906-ff88417227f2.jpg)
![화면 캡처 2022-05-16 151029_LI](https://user-images.githubusercontent.com/57117748/168603790-e485b5ef-7a10-4f39-9123-90a81f9bc31c.jpg)


2. 생성한 공개키를 서버에 등록하기
```
# ssh-copy-id UserName@ IP
```

3. 생성 완료 후 클라이언트의 /root/.ssh/id_rsa.pub 의 값이 서버의 /.ssh/authorized_keys 에 이어쓰기 된것 확인

![화면 캡처 2022-05-16 151749_LI](https://user-images.githubusercontent.com/57117748/168605310-0318ca28-dac3-4a9c-a871-b0037aed5d3b.jpg)

4. key 기반 인증 가능 

![화면 캡처 2022-05-16 152011](https://user-images.githubusercontent.com/57117748/168605522-28373d7b-166a-4d86-92b6-798a2a73a9b1.png)


## 퀴즈
1. 클라이언트 user1을 만들어 패스워드 없이 서버에 로그인 할 수 있도록 만들어라

![화면 캡처 2022-05-16 225732](https://user-images.githubusercontent.com/57117748/168609724-67d06c01-984c-43de-a726-79b1760277ab.png)



# 암호학
## 일방향 암호화
- 암호화를 진행한 암호문이 다시 복호화를 진행 할 수 없는 암호화
- Ex) md5, sha 암호화 논리

## 양방향 암호화 
- 암호화를 진행한 암호문이 다시 복호화를 진행 할 수 있는 암호화
- key : 암호문을 복호화 할 수있는 일정 길이의 문자열
1. 대칭키(비밀키) :암호문을 만들고 복호문을 만드는데 필요한 key가 동일한 key를 가진다. Ex des, aes
	* 단점 : 키 전달 문제 
	* 해결책 : 비대칭키 암호화
2. 비대칭키(공개키, 개인키) 
	* 공개키는 누구나 가질수 있는 전세계 공유가 가능한 키, 개인키는 누구에게도 주지 않는 키
	* 공개키로 암호화 한 것은 개인키로만 해독이 가능하다.
	* 개인키로 암호화 한 것은 공개키로만 해독이 가능하다.
	* 단점 : 공격자가 개입해 중간에서 키를 바꿔치기 할 수 있다.
		- 키의 신뢰정 문제의 해결책으로 인증기관(CA) 이 도입 되었다
		- CA 에서 인증한 공개키 = 인증서
		- 그러나 대칭키 방식보다 속도가 현저히 느리다, 해결책으로 대칭키만 암호화해 전달하는데 사용 (SSL)

* SSL
	- 평문 통신 프로토콜을 암호화 해주는 프로토콜
	- http + SSL = HTTPS
	- ftp + SSL = FTPS
```
1. 서버가 인증기관에 정보와 공개를 보내 인증요청 
2. 인증기관은 정보와 서버의 공개키를 인증서 개인키(인증서)로 만듬
3. 서버에 인증서를 발급하고 웹브라우저에 인증서 공개키를 제공
4. 클라이언트가 서버에 접속 요청
5. 서버가 SSL 로부터 인증받은 인증서 전달
6. SSL의 공개키로 인증서 검증해 서버의 정보와 공개키 획득
7. 획득한 서버의 공개키로 대칭키를 암호화해서 전송
8. 서버의 개인키로 해독해 대칭키를 획득
9. 안전하고 대칭키를 이용해 암호문을 주고 받음
```
[SSL 설명](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=ssl_study&logNo=30151160122&view=img_1)

* SCP, SFTP 은 SSH의 부록 모듈

# 네트워크 관리

- 네트워크 관리자 : 기본 네트워크 데몬

## 네트워크 설정 

1. 자동 -> DHCP가 자동으로 네트워크 세팅

2. 수동 -> IP, Subnetmask, DNS, Gateway 를 지정

## 시스템 관리자

- nmcli
- # systemctl status network 
- network 활성 / 비활성 nmcli on/off

![화면 캡처 2022-05-16 153557](https://user-images.githubusercontent.com/57117748/168611355-9f92a636-009f-4078-9a40-853991f11cbe.png)

1. connection
	- 네트워크 설정하기 : 네트워크 연결 설정
```
nmcli connection or con or c [옵션]


connection 옵션 ---------------------------------------------------------------

show : 연결된 네트워크를 목록을 보여줌
show [프로필]: 해당 네트워크 프로필의 자세한 정보 
up [프로필] : 해당 프로필의 네트워크 연결
down [프로필] : 해당 프로필의 네트워크 연결해체
reload : 다시 실행하기
```

- nmcli connection
![nmcli connection](https://user-images.githubusercontent.com/57117748/168614432-e79f1e98-f6e8-4f6b-b0b3-a9adc620ccf2.jpg)


- nmcli connection show
![화면 캡처 2022-05-16 154517_LI](https://user-images.githubusercontent.com/57117748/168614895-76df4549-7c38-45bd-9141-029d1edfe6c8.jpg)

- nmcli connection up / down
![화면 캡처 2022-05-16 154623_LI](https://user-images.githubusercontent.com/57117748/168615381-ed7948dc-c155-4f78-9d7d-a126d5593c07.jpg)
![화면 캡처 2022-05-16 155329_LI](https://user-images.githubusercontent.com/57117748/168615390-e7bf1b64-6e4e-43d4-919a-a71debca16f5.jpg)



- 네트워크 추가 및 제거
	* nmcli con add type ethernet con-name 프로필명 ifname 연결네트워크카드 ip4 IP주소 gw4 게이트번호	: 네트워크 추가

```
# nmcli con add type ethernet con-name [프로필명] ifname [연결 네트워크 카드] ip4 [IP주소/Sub] gw4 [게이트웨이 주소]	// 네트워크 추가

# nmcli con add type ethernet con-name test ifname enp0s8 ip4 192.168.56.10/24 gw4 192.168.56.1	// 네트워크 추가
```
- nmcli connection add

![nmcli connection add](https://user-images.githubusercontent.com/57117748/168619105-76b55955-a5e9-48e4-9bea-14f57631feae.jpg)

- 네트워크 수정
	* mod 를 이용해 네트워크의 속성을 수정한다.

```
modify [프로필] ipv4.addresses [IP/Sub]	// 해당 프로필 IP 주소 바꾸기
modify [프로필] ipv4.gateway [x.x.x.x]	// 해당 프로필 gateway 주소 바꾸기
modify [프로필] ipv4.dns [8.8.8.8]	// 해당 프로필 dns 주소 바꾸기
modify [프로필] ipv4.method [manual / auto]	// 해당 프로필 네트워크 설정 수동 / 자동 바꾸기
```

- nmcli connection mod 

![화면 캡처 2022-05-16 162645_LI](https://user-images.githubusercontent.com/57117748/168623233-97d41ffc-68c7-45f5-b5a4-78e14eb34edf.jpg)
![화면 캡처 2022-05-16 162830](https://user-images.githubusercontent.com/57117748/168623375-d2ff207f-fe11-4857-a346-a24a06e2d41b.png)


- 네트워크 삭제
	* 네트워크 삭제하기전 꼭 연결을 먼저 해제한다.

```
# nmcli con down [프로필명]	// 네트워크 연결해제

# nmcli con del [프로필명]	// 네트워크 삭제

```

![화면 캡처 2022-05-17 000801](https://user-images.githubusercontent.com/57117748/168624631-1d48b3c1-6a07-4350-b31c-4b9ce94f03ba.png)


2. device
- dev 를 통해 device 에 관련된 명령을 부여

- dev 옵션
```
nmcli dev status	// 네트워크 디바이스 목록 확인
nmcli dev show [디바이스명]	// 해당 디바이스 상세정보 확인
```

![화면 캡처 2022-05-17 001236](https://user-images.githubusercontent.com/57117748/168625530-03cfcec4-f89b-4b89-a891-65e6c26fb8b1.png)

