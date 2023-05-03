# Internet Protocol 2
## CIDER 표기법 
- IP와 Subnetmask를 함께 표기하는법. Subnetmask bit의 수 A:8 B:16 C:24( xxxx.xxx.xxx.xxx/24)

## 특수목적 IP
1. 0.0.0.0/8 
	- 모든 네트워크
2. 127.0.0.0/8 
	- Loopback address 네트워크의 기능을 점검
3. 각 네트워크의 첫번째 주소(Host ID bit 가 0인 주소)
	- 해당 네트워크의 주소
4. 각 네트워크의 마지막 주소(Host ID bit 가 1인 주소)
	- 해당 네트워크의 브로드캐스트 주소
5. 169.254.0.0/16 
	- DHCP에서 정상적으로 IP 할당을 못 받은 경우 운영체제가 임의로 IP주소를 지정하는 네트워크
6. 255.255.255.255
	- 브로드캐스트 주소

## Subnetting 
- IP 사용자가 늘어남에 따라 네트워크의 수를 늘리기위해 네트워크를 여러개의 네트워크로 작게 분할하여 네트워크를 사용하는 방법
- Host bit을 Network bit으로 변경 하는 방법

### Subnetting 의 방법
- 네트워크 수에 따른 방식
- 호스트의 수에 따른 방식 (네트워크의 규모에 따른 방식)

1. 네트워크 수에 따른 방식

* Q1) 195.210.32.0/24 의 IP 주소와 Subnetmask 를 가진 네트워크를 2개의 네트워크로 분할 하시오.
```
1. IP 주소를 2진수로 바꾸어 준다.								
								195.210.32.0/24    --> 11000011.11010010.00100000.00000000

2. Subnetmask 주소를 2진수로 바꾸어 준다.						
								24 = 255.255.255.0 --> 11111111.11111111.11111111.00000000

3. Host bit의 첫 bit 을 Network bit(0,1) 으로 Subnetting		
													  11000011.11010010.00100000.0 0000000 
													  11000011.11010010.00100000.1 0000000

4.Subnetmask의 마찬가지로 진행, 해당 비트에 들어올수 있는 가장 큰 수로 지정
													  11111111.11111111.11111111.1 0000000

5. 10진수로 변환 
								11000011.11010010.00100000.00000000		195.210.32.0	// N1
								11000011.11010010.00100000.10000000		195.210.32.128 	// N2
								11111111.11111111.11111111.10000000		25				// Subnetmask
6. 결과
								195.210.32.0/25
								195.210.32.128/25
7. 정리
Network 1		Network IP : 195.210.32.0/25
				Subnetmask : 255.255.255.128
				IP address Range : 195.210.32.0 - 195.210.32.127
				Broadcast add : 195.210.32.127
				IP Address Available for Hosts Range : 195.210.32.1 - 195.210.32.126

Network 2		Network IP : 195.210.32.128/25
				Subnetmask : 255.255.255.128
				IP address Range : 195.210.32.128 - 195.210.32.255
				Broadcast add : 195.210.32.255
				IP Address Available for Hosts Range: 195.210.32.129 - 195.210.32.254

```		

* Q2) 211.61.62.0/24 네트워크를 4개로 서브네팅 하시오.
```
1.		11010011.00111101.00111111.00000000

2.		11111111.11111111.11111111.00000000

3.		11010011.00111101.00111111.00 000000
		11010011.00111101.00111111.01 000000
		11010011.00111101.00111111.10 000000
		11010011.00111101.00111111.11 000000

4.		11111111.11111111.11111111.11 000000

5.		11010011.00111101.00111111.00000000	= 211.61.62.0 		// N1
		11010011.00111101.00111111.01000000 = 211.61.62.64		// N2
		11010011.00111101.00111111.10000000 = 211.61.62.128		// N3
		11010011.00111101.00111111.11000000 = 211.61.62.192 	// N4
		11111111.11111111.11111111.11000000 = 26				// Subnetmask

7. 정리
Network 1		Network IP : 211.61.62.0/26
				Subnetmask : 255.255.255.192
				IP address Range : 211.61.62.0 - 211.61.62.63
				Broadcast add : 211.61.62.127
				IP Address Available for Hosts Range: 211.61.62.1 - 211.61.62.62

Network 2		Network IP : 211.61.62.64/26
				Subnetmask : 255.255.255.192
				IP address Range : 211.61.62.64 - 211.61.62.127
				Broadcast add : 211.61.62.255
				IP Address Available for Hosts Range: 211.61.62.65 - 211.61.62.126

Network 3		Network IP : 211.61.62.128/26
				Subnetmask : 255.255.255.192
				IP address Range : 211.61.62.128 - 211.61.62.191 
				Broadcast add : 211.61.62.255
				IP Address Available for Hosts Range: 211.61.62.127 - 211.61.62.190

Network 4		Network IP : 211.61.62.192 /26
				Subnetmask : 255.255.255.192
				IP address Range :211.61.62.192 - 211.61.62.255
				Broadcast add : 211.61.62.255
				IP Address Available for Hosts Range: 211.61.62.191 - 211.61.62.254
```

2. 호스트의 수에 따른 방식
	- 호스트의 갯수에 따른 방식으로 진행할땐 '호스트 ID가 최소 몇개'가 되어야 하는지 중요하다
	- 2^H-2 가 적어도 최소 호스트 수를 포함해야한다.
	- 위에서 H 는 Host ID bit 의 자릿수이다.


* Q1 203.220.48.0/24 네트워크를 네트워크 당 100 Hosts가 수용 가능한 네트워크로 서브네팅 하시오.
```
1. Host ID bit 와 최소 호스트 값을 비교해 값을 찾는다.
										2^7(123bit)-2 > 100 

2. IP 주소를 2진수로 바꾸어 준다.
										11001011.11011100.00110000.00000000	

3. 2^7번째의 Host ID 부터 Subnetting.	
										11001011.11011100.00110000.0 0000000	
										11001011.11011100.00110000.1 0000000

4.Subnetmask의 마찬가지로 진행, 해당 비트에 들어올수 있는 가장 큰 수로 지정
										11111111.11111111.11111111.1 0000000

5. 10진수로 변환 
										11001011.11011100.00110000.0 0000000 = 203.220.48.0 	// N1
										11001011.11011100.00110000.1 0000000 = 203.220.48.128 	// N2
										11111111.11111111.11111111.1 0000000 = 25				// Subnetmask
6. 결과
								203.220.48.0/25
								203.220.48.128/25
7. 정리
Network 1		Network IP : 203.220.48.0/25
				Subnetmask : 255.255.255.128
				IP address Range : 203.220.48.0 - 203.220.48.128
				Broadcast add : 203.220.48.127
				IP Address Available for Hosts Range : 203.220.48.1 - 203.220.48.126

Network 1		Network IP : 203.220.48.128/25
				Subnetmask : 255.255.255.128
				IP address Range : 203.220.48.128 - 203.220.48.255
				Broadcast add : 203.220.48.255
				IP Address Available for Hosts Range : 203.220.48.129 - 203.220.48.254
```





* Q2) 160.250.128.0/24 네트워크를 네트워크 당 20 Hosts가 수용 가능한 네트워크로 서브네팅 하시오.
```
1.		2^5(32bit)-2 > 20

2.		10100000.11111010.10000000.00000000

3.		11010011.00111101.00111111.000 00000
		11010011.00111101.00111111.001 00000
		11010011.00111101.00111111.010 00000
		11010011.00111101.00111111.011 00000
		11010011.00111101.00111111.100 00000
		11010011.00111101.00111111.101 00000
		11010011.00111101.00111111.110 00000
		11010011.00111101.00111111.111 00000

4.		11111111.11111111.11111111.111 00000

5.		11010011.00111101.00111111.000 00000 = 160.250.128.0	// N1
		11010011.00111101.00111111.001 00000 = 160.250.128.32	// N2
		11010011.00111101.00111111.010 00000 = 160.250.128.64	// N3
		11010011.00111101.00111111.011 00000 = 160.250.128.96	// N4
		11010011.00111101.00111111.100 00000 = 160.250.128.128	// N5
		11010011.00111101.00111111.101 00000 = 160.250.128.160	// N6
		11010011.00111101.00111111.110 00000 = 160.250.128.192	// N7
		11010011.00111101.00111111.111 00000 = 160.250.128.224	// N8
		11111111.11111111.11111111.111 00000 = 27

7. 정리
Network 1		Network IP : 160.250.128.0/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.0 - 160.250.128.32
				Broadcast add : 160.250.128.32
				IP Address Available for Hosts Range:160.250.128.1 - 160.250.128.31

Network 2		Network IP : 160.250.128.33/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.33 - 160.250.128.63
				Broadcast add : 160.250.128.63
				IP Address Available for Hosts Range:160.250.128.34 - 160.250.128.62

Network 3		Network IP : 160.250.128.64/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.64 - 160.250.128.95
				Broadcast add : 160.250.128.95
				IP Address Available for Hosts Range:160.250.128.65 - 160.250.128.94

Network 4		Network IP : 160.250.128.96/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.97 - 160.250.128.127
				Broadcast add : 160.250.128.127
				IP Address Available for Hosts Range:160.250.128.97 - 160.250.128.126

Network 5		Network IP : 160.250.128.128/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.128 - 160.250.128.159
				Broadcast add : 160.250.128.159
				IP Address Available for Hosts Range:160.250.128.129 - 160.250.128.158

Network 6		Network IP : 160.250.128.160/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.160 - 160.250.128.191
				Broadcast add : 160.250.128.191
				IP Address Available for Hosts Range:160.250.128.161 - 160.250.128.190

Network 7		Network IP : 160.250.128.192/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.192 - 160.250.128.224
				Broadcast add : 160.250.128.224
				IP Address Available for Hosts Range:160.250.128.193 - 160.250.128.223

Network 8		Network IP : 160.250.128.224/27
				Subnetmask : 255.255.255.224
				IP address Range : 160.250.128.224 - 160.250.128.255
				Broadcast add : 160.250.128.255
				IP Address Available for Hosts Range :160.250.128.225 - 160.250.128.254
```

# ICMP(Internet Contorol Message Protocol)
- IP 프로토콜을 보조하는 프로토콜. 에러보고 및 정보제공을 수행한다.

![ICMP Header](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRKYficrAH_pIoP5ORJA35I9eYyA72SBRA1Dw&usqp=CAU)

- 필수 Header
```
- Type : ICMP의 메세지 종류를 나타내는 필드 - 8bit
- Code : ICMP의 메세지 종류의 세부 분류를 나타내는 필드 - 8bit
- Checksum : ICMP 메세지의 에러 검증을 위한 필드 - 16bit
```

- 추가 header
```
- Other message specific information : ICMP 메시지의 종류에 따라 데이터를 포함하는 경우 데이터가 위치하는 필드
```

# Transport Layer
- 데이터 단위가 에러 없이 전송되도록 관리하고 메시지 분할과 재조립, 프로세스 간 정보 흐름 조정 등을 수행

- Port : 네트워크와 통신하는 프로세스가 사용하는 호스트 내에서의 논리주소 (포트의 범위 : 0 ~ 65535)
	1. Well-known Port(0 ~ 1023) : 잘 알려진 포트로 주요 프로토콜이 사용하는 포트 범위
  	2. Registered Port(1024 ~ 49151) : 애플리케이션이 사용하는 포트 범위
  	3. Dynamic Port(49152 ~ 65535) : 운영체제가 클라이언트에게 할당하는 포트 범위

## TCP

- 전송데이터 중 하나로 신뢰적인 프로토콜이며 네트워크로 데이터를 전달하기 전 연결 역할을 수행하는 프로토콜.
- 흐름제어(Flow Contorol), 혼잡제어(Congestion Control) 등의 기능을 제공함.
- 연결 지향형(Connection Oriented) 프로토콜이다.

![다운로드](https://user-images.githubusercontent.com/57117748/235821159-33febe8c-9609-4ebc-862f-8808017617af.png)


- 필수 Header
```
- Source Port : 출발지의 포트 넘버 - 16bit
- Destination Port : 수신 프로세스의 포트 넘버 - 16bit
- Sequence Num : TCP 세션을 동기화 하기 위한 순서번호. - 32bit
- Acknowledgement Number : 요청을 잘 받았다는 것을 나타내는 확인 번호 (수신한 데이터의 길이 + 1 로 표현) - 32bit 
- Offset(Header Length) : TCP 헤더의 길이 - 4bit 
- Reserved : 예약된 필드(사용되지 않음) -  4bit
- TCP Control Flags : TCP 의 상태를 관리한다.
	1. SYN : TCP 연결을 수립할때 동기화를 위한 Flag
	2. ACK : 요청 데이터의 대한 응답의 Flag
	3. PSH : 전송할 데이터가 있는 경우 사용
	4. URG : 긴급데이터의 유무를 나타내는 Flag 
	5. RST : TCP 연결을 강제로 초기화 시킬때 사용하는 Flag
	6. FIN : TCP 연결 정상 종료과정에서 나타내는 Flag
- Window : 수신 버퍼의 크기를 나타내는 필드 - 16bit
- Checksum : TCP Header + Data에 대한 오류 검증을 위한 필드 - 16bit
- Urgent Pointer : URG Flag가 SET 되었을 때 긴급 데이터의 마지막 위치를 가리키는 필드 - 16bit
```

- 추가 Header
```
- Options : TCP에서 추가적으로 정의할 필드가 있는 경우 사용 - 가변 길이
- Padding : Options 부분이 4 Bytes로 끊어지지 않는 경우 나머지 부분을 채우며  - 가변 길이
```

1. TCP 3 Way-Handshake
- 연결하고자 하는 두 장치 간의 논리적 접속을 성립하기 위해 사용하는 연결 확인 방식으로, 3번의 확인 과정을 거친다고 해서 3 way handshake
- TCP 3 Way Handshake는 TCP/IP프로토콜을 이용해서 통신을 하는 응용프로그램이 데이터를 전송하기 전에 먼저 정확한 전송을 보장하기 위해 상대방 컴퓨터와 사전에 세션을 수립하는 과정을 의미한다.

![3 Way Handshake](https://camo.githubusercontent.com/7b4127e982da5bb10ab26f66eff79791df0f46f97f0f2e6f993b25200e9e0681/687474703a2f2f7777772e746370697067756964652e636f6d2f667265652f6469616772616d732f7463706f70656e337761792e706e67)

(1) Client  -> Server : TCP SYN

(2) Client  <- Server : TCP SYN ACK

(3) Client  -> Server : TCP ACK


2. TCP 4 Way-HandShake
- 3 way handshake와 반대로 가상 회선 연결을 해제할 때 주고 받는 확인작업이다. 이 역시 4번의 확인과정을 거친다고 하여 4 way handshake 를 거친다.
- TCP는 신뢰성이 있는 프로토콜이기 때문에 종료를 거친다.

![4 Way Handshake](https://postfiles.pstatic.net/MjAyMjAzMjRfMTg1/MDAxNjQ4MDk2NDg1MzE3.oR0X9KaTKqXR2WBhEIZgKWO7MsOTMh2Zpp4H6lkkPaUg.Y8ODemJBrNf4DvGJqxAJ8ntNVVUOhUgaeSWc_YbPUjUg.PNG.kykyu/image.png?type=w773)

(1) Client  -> Server  : 연결 종료를 위한 FIN플래그 전송

(2) Client  <- Server : 서버 메시지 확인후 자신의 통신이 끝날때 까지 기다림 (Time_Wait)

(3) Client  <- Server : 서버가 통신이 끝나면 연결이 종료를 클라이언트에게 FIN 플래그 전송

(4) Client  -> 클라이언트는 확인했다고 메시지 전송


## UDP (User Datagram Protocol)
- 전송 프로토콜 중 하나로 비 신뢰적, 비 연결지향적 프로토콜
- TCP에 비해 가벼운 프토로콜로 '빠른 데이터 전송'을 요구할 때 사용하는 전송 프로토콜

![UDP Header](https://upload.wikimedia.org/wikipedia/commons/0/0c/UDP_header.png)

- Source Port : 출발지 포트 번호 - 16bit의 필드 길이
- Destination Port : 목적지 포트 번호 - 16bit의 필드 길이
- Length : UDP 패킷의 길이(UDP Header + UDP Data) - 16bit의 필드 길이
- Checksum : UDP 헤더와 데이터의 오류를 검증하기 위한 필드 - 16bit의 필드 길이

## 키워드
- CIDER 표기법 : IP와 Subnetmask를 함께 표기하는법.
- Subnetting :  네트워크를 여러개의 네트워크로 작게 분할하여 네트워크를 사용하는 방법으로 호스트/네트워크 수의 따라 하는 방식이 나눠진다.
- ICMP(Internet Contorol Message Protocol) : IP 프로토콜을 보조하는 프로토콜. 에러보고 및 정보제공을 수행한다. 
- TCP(Transmission Control Protocol) : 네트워크로 데이터를 전달하기 전 연결 역할을 수행하는 프로토콜. 
- 3 Way-Handshake : 데이터 전송을 위한 연결 전 세션을 수립과정으로 호스트간 SYN,ACK Flag를 사용한다..
- 4 Way-Handshake : 데이터 전송 종료를 위한 세션 수립과정으로 호스트간 FIN, SYN, ACK Flag 를 사용한다.
- UDP(User Datagram Protocol) : 전송 프로토콜 중 하나로 비 신뢰적, 비 연결지향적 프로토콜로 '빠른 데이터 전송'을 요구할 때 사용한다.
