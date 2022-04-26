

# 네트워크 모델과 프로토콜

## OSI Reference Model 네트워크의 흐름
- A 송신자가 B 수신자에게 데이터를 전달하려고 OSI 7 Layer 에서는 어떤일이 일어날까?
1. 송신자가 데이터를 전달하려 한다면 OSI 7 Layers 역순 절차대로 encapsulation을 진행한다.
2. 수신자가 데이터를 수신하려 한다면 OSI 7 Layers 절차대로 decapsulation을 진행한다.

```
송신자  											수신자
L7 (DATA)										L7 (DATA)						
				encapsulation 									decapsulation
L6	[L6(DATA)] 									L6	[L6(DATA)]
				encapsulation 									decapsulation
L5	[L5(DATA)] 									L5	[L5(DATA)]
				encapsulation 									decapsulation
L4	[L4(DATA)] 									L4	[L4(DATA)]
				encapsulation 									decapsulation
L3	[L3(DATA)] 									L3	[L3(DATA)]
				encapsulation 									decapsulation
L2	[L2(DATA)T]									L2	[L2(DATA)T]	
				encapsulation									decapsulation
L1	아날로그 데이터 					 ---------->		   	L1	 아날로그 데이터				
```

- Encapsulation (캡슐화)	: 해당 레이어에서 처리할 프로토콜을 헤더에 기록해 하위 레이어에게 전달하는것.
- Decapsulation (역캡슐화)	: 하위 레이어에서 캡슐화를 해제하고 상위레이어로 전달하는것.

## TCP/IP Suite
- TCP/IP 
	* 사실상 현재는 OSI Rference Model 보다는 TCP/IP Protocal suite Protocol 을 더 많이 사용한다.
	* 단순히 TCP/IP가 아닌 TCP/IP Suite 에 있는 모든 프로토콜을 지칭한다.

1. Network Interface Layer(=Network Access Layer)
	- OSI의 Physical Layer, Data-Link Layer와 유사한 기능을 수행하는 계층이다.
	- TCP/IP의 가장 최하위 계층.
	- OSI의 L1, L2와 같은 프로토콜.
	- 대표적인 프로토콜 : 이더넷, ARP
```

Ethernet : 특정 구역 내 정보통신망인 LAN에 사용되는 이더넷의 물리적인 주소 MAC Address를 위함.
ARP : 네트워크에 접속되어 있는 컴퓨터의 인터넷 주소(IP 주소)와 이더넷 주소를 대응시키는 프로토콜.
```
### 이더넷(EtherNet)
-  데이터 링크 계층에서 MAC(media access control) 패킷과 프로토콜의 형식을 정의한다.
- MAC Address = 물리주소
	* 물리주소
		- 네트워크 카드에 할당 된 네트워크 주소, 길이는 6Bytes 이고 16진수로 나타냄(1~9, a~f)

![이더넷 프레임](https://mblogthumb-phinf.pstatic.net/20101203_78/demonicws_1291365693500VFmnO_JPEG/ethernetframe_01.jpg?type=w2)

```
Invoke Capture

1. Preamble : Ethernet 의 프레임 데이터 전송을 알리는 필드 7Bytes(10101010 x 7)
2. SFD(SOF) : Ethernet 의 프레임(데이터의) 시작을 알리는 필드 1byte(10101011)
```
```
Ether Header Capture (to where, from where)

1. DST : 데스티네이션 어드레스 (목적지 물리주소) 6bytes
2. SRC : 소스 어드레스 (출발지 물리주소) 6byte
3. Type : 상위 프로토콜의 종류를 나타냄 2Byte 
```
```
Data

1. Data : 상위계층에서 캡슐화된 데이터
```
```
Tail

1. FCS : Ethernet 프레임 전송 오류를 검증하기 위한 필드 4Bytes
```

### ARP (Address Resolution Protocol)
- IP Add를 MAC Add로 변환하는 프로토콜
- ARP Protocol 동작 형태 (Mac Add를 알아내는 형태)
 	1. ARP Request
     	* Broadcast로 해당 MAC address 질의
	 2. ARP Reply
      	* Unicast로 해당 MAC address 응답

		 

![ARP 프레임](https://reaper81.files.wordpress.com/2010/07/arp-header3.png)
```
1. Hardware Type : 물리적 네트워크의 종류(Ethernet : 0x1)
2. Protocol Type : 논리주소를 제공하는 프로토콜 종류 (IPv4 : 0x 0800)
3. hardware Add Length : 물리주소의 길이 6byte
4. Protocol Add Length : 논리주소의 길이 4byte
5. OPCode : ARP Protocol 메세지 종류 (ARP Request : 1,  ARP Reply : 2)
6. Sender Hardware Add : 송신자의 물리주소
7. Sender Protocol Add : 송신자의 논리주소
8. Target Hardware Add : 수신자의 물리주소
9. Target Protocol Add : 수신자의 논리주소
```


2. Internet Layer 
	- OSI의 Network Layer와 유사한 기능을 수행하는 계층이다. 
	- IP Protocol
	- 대표적인 프로토콜 : IP
```
IP : OSI 네트워크 계층에서 호스트의 주소지정과 패킷 분할 및 조립 기능을 담당한다.
```


3. Transport Layer
	- OSI의 Transport Layer와 유사한 기능을 수행하는 계층이다.
	- 대표적인 프로토콜 : TCP, UDP
```
TCP : TCP는 근거리 통신망이나 인트라넷, 인터넷에 연결된 컴퓨터에서 실행되는 프로그램 간에 일련의 옥텟을 안정적으로, 순서대로, 에러없이 교환할 수 있게 한다
UDP : TCP와 함께 패킷으로 알려진 단문 메시지를 교환하기 위해서 사용된다.
```

4. Application Layer
	- OSI 의 Ssasion, Presentation, Application Layer와 유사한 기능을 하는 계층이다.
	- 대표적인 프로토콜 : HTTP, HTTPS, SSH, POP3, SMTP, FTP, Telent, SMB, DHCP, DNS, SSL
```
HTTP   : W3 상에서 정보를 주고받을 수 있는 프로토콜이다. 주로 HTML 문서를 주고받는 데에 쓰인다.
HTTPS  : W3 프로토콜인 HTTP의 보안이 강화된 버전이다.
SSH	   : 네트워크 상의 다른 컴퓨터에 로그인하거나 원격 시스템에서 명령을 실행하도록 해주는 프로그램 및 프로토콜.
POP3   : TCP/IP 연결을 통해 이메일을 가져오는데 사용된다. 
SMTP   : 인터넷에서 이메일을 보내기 위해 이용되는 프로토콜이다.
FTP    : 서버와 클라이언트 사이의 파일 전송을 하기 위한 프로토콜이다.
Telnet : 텔넷(TELNET)은 인터넷이나 로컬 영역 네트워크 연결에 쓰이는 네트워크 프로토콜이다. 
DHCP   : 호스트 IP 구성 관리를 단순화하는 IP 표준이다. 
DNS    : 호스트의 도메인 이름을 호스트의 네트워크 주소로 바꾸거나 그 반대의 변환을 수행할 수 있도록 하기 위해 개발되었다.
SSL    : 컴퓨터 네트워크에 통신 보안을 제공하기 위해 설계된 암호 규약이다.
```

## OSI 7 Layer vs TCP/IP Suite

```
TCP/IP Suite 				OSI 7 Layer

L4(Application Later)		L7(Appliaction Layer)
							L6(Presentaion Layer)
							L5(Sassion Layer)

L3(Transport Layer)			L4(Transport Layer)

L2(Internet Layer)			L3(Network Layer)

L1(Network Layer)			L2(Data-Link Layer)
							L1(Physical Layer)

```

* OSI Reference Model 은 7계층, TCP/IP Suite 는 4계층으로 후자가 더 가볍게 구성되어 있다.

# 네트워크 장비
## L1
- Hub (Dummy Hub) 
	* 여러 호스트를 연결시켜주는 네트워크 장비
	* 입력 신호를 받아 다른포트에 복제해 출력하는 장비
	* 연결된 호스트구분 불가

- Repeater
	* 접속 시스템 수를 증가시키거나 혹은 전송 거리를 연장하고자 할 때에 사용하는 네트워크 장비
	* 증폭장비

## L2
- Switch (Switch Hub)
	* 여러 호스트를 연결키고 프레임을 다룰 수 있는 네트워크 장비
	* 여러 물리 주소를 구별 할 수있다.(각 포트가 Collision Domain을 분별한다.)
	* 여러 포트가 연결되어도 속도저하는 없다.

- Bridge
	* 네트워크간 서로 연결을 위한 장비
	* 현재는 스위치가 그 역할을 대신한다.

## L3
- Router
	* 네트워크간 서로 연결을 위한 장비 
	* LAN to LAN / LAN to WAN / WAN to WAN
	* 목적지 네트워크 까지의 경로 계산 및 경로지정.
	* Ex 공유기

```
Router / Bridge 차이점

1.브리지는 데이터 링크 계층에서 기능하고 라우터는 네트워크 계층에서 작동합니다.

2.브리지에서 프레임은 프레임의 MAC 주소를 기반으로 전달됩니다. 반대로 라우터는 패킷의 논리적 주소(즉, IP 주소)를 확인합니다.
```

## L4
- Firewall
	* 비인가 외부의 접근을 차단하는 네트워크 보안 장비
	* IP + Port를 사용해 허용된 서비스 및 네트워크만 허용한다.
 
# 네트워크의 구조
## 네트워크 토폴로지 (Network Topology)
- 네트워크의 구성요소(링크, 노드 등)를 물리적으로 연결한 상태, 방식

## 네트워크 토폴로지 종류
- 버스형,링형,성형,트리형,그물형
	1. 버스형 토폴로지
		- 모든 네트워크 노드와 주변장치가 파이프 등의 일자형 케이플(Bus)에 연결된 상태이다.
		- 양 끝단에 정확한 신호 전달을 위한 Termination이 있다.
		- T탭을 이용해 연결한다.

	2. 링형 토폴로지
		- 노드가 순차적으로 링에 연결된 형태로 모든 노드가 하나의 링 형태로 연결.
		- 각 노드는 인정한 2개의 노드하고만 연결된다.

	3. 성형(Star) 토폴로지
		- 중앙에 제어노드가 있고 나머지 모든 노드를 제어노드와 연결해 놓은 형태의 네트워크 토폴로지.
		- 모든 노드가 '중앙제어노드'에 연결되어 통신함으로 중앙노드의 안정성과 제어가 중요하다.

	4. 트리형 토폴로지
		- 서형 토폴로지의 변형의 형태로 성형의 단점을 일부분 해소한다.
		- 중간에 제어노드를 두어 하위 노드들을 제어한다.

	5. 그물형 토폴로지(Mesh Topology) 
		- 전체의 노드가 1:1 로 연결되어 있는 형태.
		- 네트워크 구성 방식중에 가장 복잡하고 고비용이지만 가장 안정적인 토폴로지 형태.
		- 보통은 코어 네트워크에서 안전성을 위해 구축을 한다.

![Network Topologies](https://www.mbaknol.com/wp-content/uploads/2016/02/Network-Topology-Types-Mbaknol.jpg.webp)

- 환경에 따라 여러가지 토폴로지를 혼합해 사용한다.
## 키워드
