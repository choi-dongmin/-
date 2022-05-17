# Network Management
- 네트워크 데몬이 항상 Background 에서 돌아가며 관리하기 위한 명령으로 nmcli, ip, ifconfig 등을 사용한다.
- 네트워크 관리의 핵심은 "연결파일"을 만들어 device 와 연결 시키는 것이다.

## ip Command
- 가장 초기의 기본적인 Linux 네트워크 관리 명령어이다.
- ip Command를 통해 네트워크를 세팅을 하는 경우, 만약 network manager 가 설지되어 있는경우 network manager 설정으로 부팅된다.

- ip 기본 명령어
```
$ ip addr	// networks의 ip 정보들 출력
$ ip route	// networks의 route, gateway 정보들 출력
```

- 인터페이스의 ip를 ip Command를 사용해 설정하기
```
1. ip addr를 통해 Network Interface 에 새로운 ip 부여
$ ip addr add [NewIP/SUB] dev [Interface]		// 해당 인터페이스의 새로운 ip 부여
$ ip addr del [NewIP/SUB] dev [Interface]		// 해당 인터페이스의 ip 제거

2. ip route 를 통해 network Interface 의 새로운 Gateway 부여 
$ ip route add default via [GatewayNumber] dev [Interface]	// 해당 인터페이스의 새로운 Gateway 부여
$ ip route del [GatewayNumber] dev [Interface]	// 해당 인터페이스의 Gateway 제거

3. ip route 를 통해 라우팅 경로 추가, 삭제
$ ip route add [Destination IP] via [Stopover IP] dev [Interface]	// 목적지의 라우팅 경로 추가
$ ip route del [Destination IP] dev [Interface]	// 목적지의 라우팅 경로 삭제
```

- ip를 통한 인터페이스 활성화/비활성화
```
ip link set enp0s8 up
ip link set enp0s8 down
```

## ifconfig 명령어
- 리눅스의 전통적인 명령어중 하나이다.

- ifconfig 기본 명령어
```
$ ifconfig // 전반적인 네트워크의 정보를 확인
```

- 인터페이스의 ip를 ifconfig 를 사용해 설정하기
```
$ ifconfig [Interface] [NewIP] netmask [Subnetmask] broadcast [broadcast]
```
![화면 캡처 2022-05-17 112618_LI](https://user-images.githubusercontent.com/57117748/168781337-1de4d211-0b7f-40e3-ba59-de15ffa6e7df.jpg)
![화면 캡처 2022-05-17 112635_LI](https://user-images.githubusercontent.com/57117748/168781451-43c14447-36e5-4589-82cd-942a7f12e22f.jpg)

- ifconfig 를 통한 인터페이스 활성화/비활성화
```
$ ifconfig [Interface] up // 해당 인터페이스 활성화
$ ifconfig [Interface] down // 해당 인터페이스 비활성화
```
![화면 캡처 2022-05-17 112119_LI](https://user-images.githubusercontent.com/57117748/168781147-eb8663d6-a91d-4d04-bdbc-6c1a77d91b19.jpg)
![화면 캡처 2022-05-17 112151_LI](https://user-images.githubusercontent.com/57117748/168780935-b8051cc7-6bfe-4175-9e55-d5322a65072f.jpg)

## vi Editor 를 통한 profile 생성 및 네트워크 설정
1. /etc/sysconfig/network-scripts 폴더의 ifcfg-[DeviceName] 파일을 확인
![화면 캡처 2022-05-17 184415](https://user-images.githubusercontent.com/57117748/168782880-20210cbc-0b23-47db-8091-903c9cc24ffe.png)

2. vi editor 를 통해 편집모드로 진입
![화면 캡처 2022-05-17 113834](https://user-images.githubusercontent.com/57117748/168782917-0ce2214d-2940-4e32-b2d8-72fee510f4c5.png)

3. vi editor 를 통해 편집
- BOOTPROTO=none : 수동 방식으로 설정 = DHCP 사용하지 않기에 ip,subnet,gateway,dns 부여 
![화면 캡처 2022-05-17 114542_LI](https://user-images.githubusercontent.com/57117748/168783241-97a297de-3426-435e-a68c-be1dd6df083d.jpg)

4. 재부팅 및 네트워크 다시 연결
![화면 캡처 2022-05-17 114751](https://user-images.githubusercontent.com/57117748/168784494-e244e4c3-e1f5-4363-82c6-343a92a8e9ff.png)

## Interface DNS 설정
- /etc/resolv.conf = dns server 설정파일
- resolv.conf에서 변경시 재부팅하면 초기화됨 그래서 nmcli 명령어를 사용


-nmcli 명령어를 통해 DNS 설정
```
$ nmcli con mod [Interface] ipv4.dns [DNSServer]	// umcli 로 dns 설정 
$ nmcli con reload 
$ nmcli con up [Interface]

```
![화면 캡처 2022-05-17 164126](https://user-images.githubusercontent.com/57117748/168787509-01dc6b58-51b1-4042-a1b5-546a45fa501c.png)
![화면 캡처 2022-05-17 191258](https://user-images.githubusercontent.com/57117748/168788147-a5916a1a-d377-4bac-be5c-4e3242a976b4.png)

## 호스트의 이름 설정

- 호스트 이름 확인
```
hostname
```

- hostnamectl 명령을 통한 호스트 이름 바꾸기
```
hostnamectl set-hostname [NewName]
```

- vi 에디터를 통한 호스트 이름 바꾸기(재부팅시 적용)
```
vi /etc/hostname
```
![화면 캡처 2022-05-17 124836](https://user-images.githubusercontent.com/57117748/168792539-2ca843f9-ce23-4cbc-990d-c879c9a037c3.png)

## 네트워크 연결 정보 확인 명령어 복습
```
nmcli con show		// 네트워크 정보 출력
nmcli con show [프로파일이름]		// 해당 프로파일 네트워크 정보 출력

nmcli device status	// devices 출력
nmcli device show 	// devices 네트워크 정보 출력
nmcli device show [DeviceName]	// 해당 device 정보 출력 
```

## 네트워크 세팅 실습
1. 동적 연결 (DHCP) / 정적 연결 (Manul IP)
	- $ nmcli con add 를 이용해 새로운 네트워크 생성 
```
nmcli con add type ethernet con-name [NEW프로필명] ifname [인터페이스명]	// 동적연결 

nmcli con add type ethernet con-name [NEW프로필명] ifname [인터페이스명] ip4 [IP/Sub] gw4 [GatewayNum]
```
![화면 캡처 2022-05-17 212937](https://user-images.githubusercontent.com/57117748/168811370-f3deef7d-80c4-4ba0-8ddd-6fb62b2def1c.png)


2. 네트워크 연결
	- $ nmcli con up & down 을 이용한 네트워크 연결 및 해제
```
$ nmcli con up [프로필명]		// 해당 프로필의 네트워크 연결
$ nmcli con down [프로필명]		// 해당 프로필의 네트워크 연결 해제
```
![화면 캡처 2022-05-17 213552](https://user-images.githubusercontent.com/57117748/168812569-537d803d-e62e-4466-82d0-6488a67cbf5e.png)

3. network 삭제
	- nmcli con del 를 통해 network 를 삭제한다
	- 삭제하기전 항상 해당 네트워크를 연결해제 시켜야한다
```
$ nmcli con down [프로필명]		// 해당 프로필의 네트워크 연결 해제
$ nmcli con del [프로필명]	// 해당 프로필의 삭제
```
![화면 캡처 2022-05-17 215009](https://user-images.githubusercontent.com/57117748/168814716-02eab4ee-6cbb-4e71-9257-ce0fff84d8aa.png)

4. nnetwork 형식 바꾸기 (동적 -> 정적)
	- nmcli con mod 를 활용해 네트워크 프로필 수정하기
```
nmcli con mod [프로필명] connection.id [새프로필명] ipv4.method manual ipv4.addresses [IP/Sub] ipv4.gateway [Gateway]
```
![화면 캡처 2022-05-17 215803](https://user-images.githubusercontent.com/57117748/168816197-f8121776-bef0-4cd0-a0a3-0afaaf0ff20c.png)
![화면 캡처 2022-05-17 215843](https://user-images.githubusercontent.com/57117748/168816375-a25716b4-4c12-4dab-b672-1bcf49894e5c.png)

# Firewall
- 외부에서의 불법적인 트래픽 유입을 막고, 허가되고 인증된 트래픽만을 허용하려는 적극적인 방어 대책의 일종이다
- 호스트 방화벽 , 네트워크 방화벽 등이 있다.

## 운영체제에 기본적으로 설치된 방화벽
- Linux에 기본적으로 설치된 방화벽 (OSI 7 Layer 중 3-4계층)
- 기존의 iptables의 데몬에서 CentOS7 로 넘어오면서 firewalld 라는 데몬 프로세스 변경
- firewall 을 해제했다면 꼭 relaod 와 systemctl restart firewalld 를 해줘야 한다.
```
# firewall-config		// GUI 환경
# firewall-cmd		// CLI 환경
# firewall-cmd --reload		// 다시 불러오기

# firewall-cmd --add-service=[프로토콜]		/# / 해당 프로토콜 방화벽 해제 
# firewall-cmd --remove-service=[프로토콜]		// 해당 프로토콜 방화벽 설정 
# firewall-cmd --list-services		// 현재 방화벽 해제 중인 프로토콜 확인

# firewall-cmd --add-port=[포트번호/해당포트 프로토콜]		// 해당 포트 방화벽 해제
# firewall-cmd --remove-port=[포트번호/해당포트 프로토콜]	// 해당 포트 방화벽 설정
# firewall-cmd --list-ports[포트번호/해당포트 프로토콜]	// 현재 방화벽 해제 중인 포트번호 확인

* 부팅하고나서도 방화벽 유지하려면 --permanent
```
![화면 캡처 2022-05-17 222123](https://user-images.githubusercontent.com/57117748/168820796-20a4ec58-f5e9-473c-94d4-76d188908891.png)

# DNS Server
- DNS 는 Domain Name Service 의 약자로 사용자가 도메인 이름으로 접속을 요청할때 도메인 실제 주소인 IP로 바꾸어 전달
- IP -> Domain / Domain -> IP
- 호스트 이름 : 네트워크 상의 컴퓨터 각각의 이름
- 도메인 이름 : 네트워크의 범위를 지정하는 이름
- FQDN(Fully qualifend domain name) :호스트이름과 도메인 이름을 모두 표기한 전체 주소 이름 Ex) www.naver.com

## DNS 구조
- DNS 는 root 로부터 내려오는 Tree 구조를 가지고 있다.
![DNS 구조](https://www.researchgate.net/profile/Xiaoliang-Zhao-6/publication/3300774/figure/fig1/AS:394714885967874@1471118770433/The-DNS-name-space-tree-structure.png)

- DNS 흐름도
```
1. /etc/hosts 확인	// 가장 처음 DNS를 확인

2. DNS 캐쉬(웹브라우저, 시스템) 확인	// 비메모리성 캐쉬 확인

3. dns 서버 		// 서버에서 요청 및 확인 		

4. root dns 서버 요청 확인	// 분류에 따른 1차 dns 서버의 주소를 3번으로 알려줌

5. dns 서버가 1차 dns 서버에게 확인 요청 

6. 1차 dns가 확인후 하위 dns 주소를 다시 알려줌

7. 위 프로세스의 진행과정 중 도메인 발견
```

## DNS Records
- DNS 설정을 위한 설정값 

1. NS(Name Server) : 도메인의 Name 서버의 정보, DNS 서버 자신에서 domain name에 대한 주소를 알아내지 못할 때, 이 NS 레코드에 정의된 서버로 가서 주소를 알아온다
2. MX(Mail Exchange) : 도메인의 Mail Exchange 서버의 정보
3. A : 호스트의 IP
4. CName : 별칭
5. SOA(Start of Authority) : 해당 도메인에 대한 가장 기본적인 인증정보를 담고 있는 레코드
6. PTR : IP주소에 대한 호스트명
7. ANY : 호스트에 관련된 모든 레코드들의 정보


## nslookup
- DNS Tool 로서 windows 와 Linux 모두 호환
```
# nslookup 
> server 	// 현재 나의 호스트의 서버정보
> www.naver.com[FQDN] 	// 해당 FQDN 서버 정보, 도메인 정보, IP 등등


# nslookup 
> set type=any 		// 호스트에 관련된 모든 레코드들의 정보를 출력 받는다
> www.naver.com
> set type=a 		// 호스트에 관련된 호스트의 IP를 출력 받는다
> www.naver.com
```
![화면 캡처 2022-05-17 224030](https://user-images.githubusercontent.com/57117748/168824616-71826dd5-79a9-4714-936d-d2ee0232ab2b.png)
![화면 캡처 2022-05-17 231120](https://user-images.githubusercontent.com/57117748/168831573-9cd66080-4d80-43a5-86df-7bb190d8980b.png)

## dig
- Linux 에서만 호환되는 DNS Tool

```
# dig @[DNS서버] [도메인 및 FQDN]

*8.8.8.8 은 구글 DNS 서버
```
![화면 캡처 2022-05-17 151327_LI](https://user-images.githubusercontent.com/57117748/168832220-a731d54c-8315-40de-bad6-cbce2ac4daf6.jpg)

## Host
- 가장 최근에 나온 DNS Tool
```
host -a [도메인 및 FQDN]
```

## /etc/hosts
- 예전에는 위 파일에 직접 dns 를 설정했었고 지금은 이 파일을 관리 하는것이 DNS Server
```
/etc/hosts	// linux
C:\Windows\System32\drivers\etc\hosts	//windows
```

![화면 캡처 2022-05-17 144441](https://user-images.githubusercontent.com/57117748/168828607-994cf459-62c9-4037-8312-cb720f965d2d.png)

# DNS 서비스 구성
- 서버의 구축 순서는 다음과 같다
```
1. 설치 
2. 설정,구축 
3. 서비스시작 
4. 방화벽 오픈
5. 이용
```
1. cashing 전용 DNS 서버 구축 (로컬전용)

1-1 설치 
```
# yum -y install bind bind-chroot		//DNS 서비스 설치
```
![화면 캡처 2022-05-17 232412](https://user-images.githubusercontent.com/57117748/168834673-696f0c5f-614d-4eef-80e5-79d2dec9effc.png)

1-2 설정,구축
	- /etc/named.conf 를 에디터를 통해 options를 바꿔준다.
```
# vi /etc/named.conf
```
![화면 캡처 2022-05-17 152452](https://user-images.githubusercontent.com/57117748/168836492-c0208d98-a7d9-43ff-b02a-c4ede52f2bd0.png)

1-3 서비스 시작
	- systemctl restart / enable named 를 통해 서비스 시작
```
# systemctl restart named
# systemctl enable named
```
![화면 캡처 2022-05-17 233322](https://user-images.githubusercontent.com/57117748/168836900-c58a1dd0-a147-4962-82ad-d999c667fed9.png)

1-4 방화벽 제거
	- firewall-cmd 를 통해 DNS 방화벽 제거
```
# firewall-cmd --permanent --add-service=dns
# firewall-cmd --reload
```
![화면 캡처 2022-05-17 233551](https://user-images.githubusercontent.com/57117748/168837510-e2150a9a-a206-440f-85c3-d8a64947e5c3.png)


2. 본격적인 네임서버 구축
	- 도메인에 속한 시스템들의 이름을 관리하고 '외부에' 해당 시스템의 ip를 전송함
	- 실제 DNS 서버의 경우 2개의 서버(Master, Slave) 구축하여 안전하게 운영한다.
	- 여기서는 1의 서버만 구축

2-1 설치
	- yum install 를 이용해 httpd 설치
```
# yum -y install httpd
```


2-2 설정,구축 1
	- html 파일을 /var/www/html 디렉토리에 넣는다 (이때 index.html 을 넣으면 해당 도메인 첫화면)
	- /etc/named.conf 를 에디터를 통해 최하단에 아래와같은 문법을 넣어준다.
	- 저장 후 checkconf 를 통해 문법검사를 거친다.
```
# vi /etc/named.conf

------------------------------------------------
zone "[새 도메인명]" IN{
	type master;
	file "[포워드존파일]";	// /var/named/ 주석으로 경로표시
	allow-update{ none; };
};
------------------------------------------------

# named-checkconf

```
![화면 캡처 2022-05-17 235517](https://user-images.githubusercontent.com/57117748/168842102-d8c4c9ac-7c1b-4a0f-ad03-d0d2592ede47.png)
![화면 캡처 2022-05-17 235829](https://user-images.githubusercontent.com/57117748/168843259-56754db5-6150-4413-af63-eb30f590e661.png)


2-2 설정,구축 2
	- /var/named 디렉토리안에 named.conf 에서 작성했던 포워드존파일을 여기에 생성한다.
	- 이때 cp 명령어를 통해 기존의 named 디렉토리의 named.localhost 파일을 내용 복사하여 만든다

```
# cd /var/named
```
![화면 캡처 2022-05-18 000423](https://user-images.githubusercontent.com/57117748/168844194-f3552e0e-e10b-4815-9603-8fb0b991b3f4.png)

```
cp /var/named.localhost ./[DNS 속성설정파일명]
```
![화면 캡처 2022-05-18 000643](https://user-images.githubusercontent.com/57117748/168844691-1b1e8122-c433-4593-a953-6a749b3fd332.png)


2-2 설정,구축 3
	- named.localhost를 복사한 DNS 속성설정파일을 수정
	- named-checkzone 도메인명 포워드존파일 로 문법 체크
![화면 캡처 2022-05-18 001749_LI](https://user-images.githubusercontent.com/57117748/168847147-e86e710f-573c-4a35-a096-99c520dd96ab.jpg)

![화면 캡처 2022-05-18 001625](https://user-images.githubusercontent.com/57117748/168846759-44eb1d1e-3a5e-478e-b85e-a4bc296ea95f.png)

![화면 캡처 2022-05-18 003124](https://user-images.githubusercontent.com/57117748/168850083-a1993420-7018-4fe6-9202-f9355e7685a4.png)

```
1. Serial : SOA 레코드에 대한 버전 정보
2. refresh : Slave Server가 Master Server에게 DNS 정보가 갱신되었는지 물어보기 위해 대기하는 시간/주기
3. Retry : Master Server가 응답하지 않을 경우 재요청 할 때 까지 기다리는 시간
4. Expire : Slave Server가 이 시간동안 응답을 받지 못하면 더이상 물어보지 않음
```


2-2 설정,구축 4 
	- /var/named 디렉토리 퍼미션 설정
```
# chmod -R 754 /var/named

-----------------------

-R : 해당 디렉토리 파일 모두 다
```
![화면 캡처 2022-05-17 163306](https://user-images.githubusercontent.com/57117748/168850998-f64f9b19-e2e7-457d-b5c6-08082ee598b9.png)


2-3 서비스 시작 1
	- systemctl restart / enable named 를 통해 서비스 시작
```
# systemctl restart named
# systemctl enable named
```
![화면 캡처 2022-05-17 233322](https://user-images.githubusercontent.com/57117748/168836900-c58a1dd0-a147-4962-82ad-d999c667fed9.png)

2-3 서비스 시작 2
	- vi 에디터를 이용해서 /etc/resolv.conf 파일을 편집
```
vi /etc/resolv.conf
```
![화면 캡처 2022-05-18 003948](https://user-images.githubusercontent.com/57117748/168852256-6046318b-1215-4127-ae6d-40fff1d97957.png)

2-3 서비스 시작 3
	- dig 을 통해 확인
```
dig @[DNS서버] [도메인주소]
```