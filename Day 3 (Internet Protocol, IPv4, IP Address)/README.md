# 네트워크의 통신 방식
1. Unicast : 통신하고자 하는 대상과 직접 메세지를 주고받는 방식 통신방식 (1 : 1)
2. Broadcast : 모든 네트워크의 모든 호스트에게 메세지를 보내는 통신방식 (1 : All)
3. Multicast : 네트워크 내의 특정 그룹 호스트들에게 메세지를 보내는 방식 (1 : Group) 

- 보통 2,3 번은 특정하게 원하는때에 사용한다.

# TCP/IP Suite

## Network-Interface Layer 
- OSI Reference Model의 Physical Layer, Data-Link Layer와 유사한 기능을 수행하는 계층이다.
- TCP/IP의 가장 최하위 계층. 물리적 주소, 논리 주소를 및 프로토콜을 헤더에 프레임으로 저장하기 위한 최하위 레이어. 
- OSI Reference Model의 L1, L2와 같은 프로토콜.
- 대표적인 프로토콜 : 이더넷, ARP

## Internet Layer 
- IP(Internet Protocol)은 논리주소(IP address)를 제공하는 프로토콜이다.
- Ex IP, IPX

### IP(Internet Protocol)
- Internet Protocol는 IP add를 기반으로 Host 와 Network를 인식해 목적지까지 패킷을 전달하기 위한 프로토콜.
- IP가 IP add 를 기반으로 경로를 찾아가는것을 Routing 이라고 일컫는다.
- 비 연결형 프로토콜 / 비 신뢰적 프로토콜 (그저 데이터를 전달하고 끝인 프로토콜)
- IPv4(Ip 주소의 길이 32bit) 와 IPv6(IP 주소의 길이 64bit) 로 구성된다. (IPv4의 IP add의 수가 한계에 다달아 IPv6를 개발 그러나 아직 IPv4 대세이다) 

* IPv4 Header

![IPv4 Header](https://upload.wikimedia.org/wikipedia/commons/thumb/5/54/Ipv4_header.svg/1280px-Ipv4_header.svg.png)

1. 필수 IPv4 Header
```
- Version : IP의 버전(IPv4 : 4, IPv6 : 6) - 4bit

- IHL(IP Header Length) : IP Header Length

- TOS : Type Of Service : QoS(Quality of Service)를 위한 필드 - 8bit
	*Qos (Quality of Servic : 어떠한 서비스가 과다한 트래픽을 사용할 경우를 대비해 각각 최소의 트래픽을 보장하는 방법 

- Total Length : IP의 패킷의 총 길이. - 16bit

- Identification : IP 패킷의 식별자 - 16bit

- IPFlags(x D M)
	1. x   - 1bit
		- Unused 
	2. DF(Don't Fragmentation) : IP 패킷의 단편화 여부를 결정하는 플래그 (0 : 단편화 가능 / 1 : 단편화 불가)
	3. MF(More Fregmentation) : IP 패킷의 단편 조각이 더 존재하는지 확인하는 필드 (0 : 더이상 단편 조각이 없음 / 1 : 단편조각이 더 있음)
	* Fregmentation : IP 패킷을 잘게 쪼개는 단편화 과정.

- Fregment Offset : IP 패킷을 단편화 시키고 원본 데이터로 재조립을 용이하게 하기위해 그 위치(Offset)가 어디에 해당하는가를 나타내는 필드   -13bit

- TTL(Time To Live) : 쓰레기 IP 패킷으로 인해 네트워크 속도 저하를 방지하기 위해 IP 패킷의 수명을 나타내는 필드 (0 ~ 255 : 네트워크를 거칠때매다 1씩 감소, 0이되면 소멸) - 8bit

- Protocol : 상위 프로토콜을 명시하는 필드  - 8bit

- Hedaer Checksum : Header의 데이터가 임의로 변경 혹은 오류가 있는지 검토하는 필드

- Source Address : 출발지 IP 주소

- Destination Address : 목적지 IP 주소
```

2. 추가 IPv4 Header
```
- IP Option : 부가적으로 지정해야할 옵션헤더 (가변길이)
- Padding : 민약 Option 에 남는 공간이 있다면 채우는 필드 (가변길이)

```

3. IP주소의 관리방식(IPv4)
- Classful : IP의 Class 단위로 IP Nerwork를 관라하는 방식.
	1. A Class (이하 Class 생략)
		* Network IP add 는 1 ~ 127 (127은 특수한 네트워크 Lookback add에 할당.)
		* 1.0.0.0 ~ 126.255.255.255 에 할당
		* 126개의 네트워크
		* 서브넷 마스크는 255.0.0.0
		* 하나의 네트워크 당 2^24-2 개의 호스트를 포함한다.

	2. B
		* Network IP add 는 128 ~ 191 
		* 128.0.0.0 ~ 191.255.255.255 에 할당
		* 2^14개의 네트워크
		* 서브넷 마스크는 255.255.0.0
		* 하나의 네트워크 당 2^16-2 개의 호스트를 포함한다.
	
	3. C
		* Network IP add 는 192 ~ 223 
		* 192.0.0.0 ~ 223.255.255.255 에 할당
		* 2^21개의 네트워크
		* 서브넷 마스크는 255.255.255.0
		* 하나의 네트워크 당 2^8-2 개의 호스트를 포함한다.

	4. D
		* Multicast 를 위한 용도로 사용되며 일반호스트가 아니다.
		* 224.0.0.0 ~ 239.255.255.255

	5. E
		* 연구용이며 일반호스트가 아니다.

- Classless : IP의 Class와 관계없이 IP 네트워크를 관리하는 방식.


* IP address
- IP adress 용어정리
	1. IP address : Host 에 할당된 고유한 논리주소이며 'Network'와 'Host' 부분으로 구분된다.
	2. Subnetmask : IP 주소에서 Network ID와 Host ID를 구별하기 위한 마스크 값.
	3. Gateway address : 같은 네트워크는 MAC add 를 이용하지만 다른 네트워크는 IP add를 사용한다. Gateway add는 다른 네트워크로 가기위한 관문 (Router IP 주소)
	4. Network address : IP 네트워크의 주소로 Host ID bit가 모두 0인 주소.
	5. Broadcast address : 네트워크의 모든 호스트에 메시지를 보낼 때 사용하는 IP 주소로 Host ID bit가 모두 1인 주소

* 공인 IP 주소 / 사설 IP 주소
1. 공인 IP 주소
	- Internet(공인 네트워크)에서 사용이 가능한 IP 주소.
2. 사설 IP 주소
	- 사설 네트워크에서 사용이 가능한 IP 주소.
```
할당 될 수 있는 사설 네트워크 주소  (LAN 또한 사설이 될 수있다.)

10.0.0.0 (Subnetmask : 255.0.0.0) - A 클래스
172.16.0.0 ~ 172.31.0.0 ( Subnetmask : 255.255.0.0) - B 클래스
192.168.0.0 ~ 192.168.255.0 (Subnetmask : 255.255.255.0) - C 
```

- NAT
	* 사설 IP 주소를 공인 IP주소로 변환 하거나 또는 그 반대의 절차를 진행한다.
	* 사설 IP가 공인 IP와 함께 공인 네트워크로 나아갈떄 NAT Table에 사설 IP를 기록하여 데이터를 다시 수신 받을때에 공인 IP까지 도달 후 사설 네트워크로 돌아올때 NAT Table 확인 절차를 거친후 진행한다.   

## 키워드