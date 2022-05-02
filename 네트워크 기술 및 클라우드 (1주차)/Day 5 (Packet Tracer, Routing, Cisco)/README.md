# Packet Tracer
- Cisco 에서 개발한 네트워크 시뮬레이션 프로그램.

## Packet Tracer Cable
* TP(Twisted Pair)
	- 주로 LAN 구간에서 사용한다.
	- RJ-45 플러그와 연결해 사용한다.

	![UTP and STP](https://miro.medium.com/max/400/0*8DTQKGWA8GrFxa9C.jpg)
	![UTP and STP](https://www.cables-solutions.com/wp-content/uploads/2016/12/T568A-T568B-Wiring-Standard.png)
	- TP Cable 종류
		1. UTP(Unshielded Twisted Pair) : 차폐가 되지 않은 꼬임선으로 주로 실내에서 사용됨.
			* Stright Through : 호스트와 스위치, 허브와 라우트 등 다른 네트워크 장비간 연결에 사용되는 케이블 방식.(케이블 양 끝쪽이 전부 T568B 의 형태)
			* Crossover Through : 라우트와 라우트, 스위치와 스위치 등 서로 같은 네트워크 장비 연결에 사용되는 케이블 방식. (케이블 한쪽이 T568A 의 형태)
		2. STP(Shielded Twisted Pair) : 차폐 기능을 포함한 꼬임선으로 옥외에서 사용됨.



	

* Serial : WAN 구간에서 Route를 서로 연결하는 케이블
* Rollover Cable : 네트워크 장비에 직접 접근하여 설정할 때 사용하는 케이블
![Rollover Cable](https://www.bffnas.com/wp-content/uploads/2015/01/Rollover-Cables.jpeg)

# Routing 
- 데이터를 목적지까지 전달 시키기 위한 최적의 경로(Best Route)를 결정하는 과정.

## Routing Protocol
- Routing을 수행하는 경로를 동적으로 구성해 "라우팅을 수행"하는 프로토콜
- Ex RIP, IGRP, EIGRP, OSPF, IS-IS, BGP 등

## Routed Protocol
 - Routing Protocol에 의해 Routing이 되는 네트워크 프로토콜
 - IP, Novell IPX, Appletalk 등

## Router 
- 네트워크와 다른 네트워크로 연결하는 장비
- 목적지까지 데이터를 전달하기 위한 최적의 경로를 계산 및 지정한다.

* Routing을 결정하는 요소
	- 홉의수 Hop count 
	- 대역폭 Bandwidth
	- 부하 Load
	- 지연 Delay
	- 신뢰도 Reliability

* Routhing의 분류 
	- Static Routing
	- Dynamic Routing

# Cisco IOS

## Cisco 의 구조 
![Cisco 의 구조 ](https://slidesplayer.org/slide/14059096/86/images/3/1+-1+%EB%9D%BC%EC%9A%B0%ED%84%B0%EC%9D%98+%EA%B5%AC%EC%A1%B0%EC%99%80+%EA%B8%B0%EB%8A%A5+1%29+%EB%9D%BC%EC%9A%B0%ED%84%B0%EC%9D%98+%EB%82%B4%EB%B6%80+%EA%B5%AC%EC%84%B1%EC%9A%94%EC%86%8C.jpg) 

## CIsco IOS Mod
1.  User Exec Mode : 최소한의 기능을 가진 모드로 Router, Switch의 초기 모드
2.  Privileged Mode(관리자 모드) : 네트워크 장비의 상태를 확인하거나 설정 내용을 확인할 수 있는 모드
3.  Global Configuration Mode :  네트워크 장비의 전역적인 설정을 하는 모드
	3-1 Interface Mode : 네트워크 장비의 인터페이스를 설정하는 모드
	3-2 Line Interface Mode : Console, VTY  등을 설정할 수 있는 모드
	3-3 Router Mode : Routing Protocol을 설정하는 모드
	3-4 VLAN : VLAN 설정 모드



## 진입 방법 및 명령어

- MOD 진입 방법
```
1. [Privileged Mode로 진입]
		Router> enable
		Router# 

2. [Global Configuration Mode로 진입]
	Router# configure terminal
		Router(config)# 
```

- 기본 명령어
```
현재 동작중인 설정 확인
Router# show running-config

영구 설정 확인
Router# show startup-config

영구 설정 삭제(장비 초기화)
Router# erase startup-config

```

## Routing 의 설정 단계
- Static Routing 을 위한 단계

```
1. 인터페이스 설정
Router> enable
Router# configure terminal
Router(config)# interface Fa or Gi or S 0/0
Router(config-if)# ip address IP_주소 서브넷마스크
Router(config-if)# no shutdown
Router(config-if)# exit

2. Static Routing 설정
Router# show ip route
Router# configure terminal
Router(config)# ip route IP_주소 서브넷마스크 NEXT_HOP_IP주소
Router(config)# exit
Router# show ip route

3. 설정 영구 저장
Router# copy running-config startup-config


```

## 키워드
- Packet Tracer : Cisco 에서 만든 가상 네트워크 시뮬레이션 프로그램으로. Cisco IOS를 사용한다.
- TP(Twisted Pair) : 또는 꼬임선이라고 하며 주로 LAN 구간에서 사용한다. 차쳬의 유무로 UTP. STP로 구분한다.
- tright Through : 상호 간 다른 네트워크 장비끼리 연결에 사용되는 케이블 방식.(케이블 양 끝쪽이 전부 T568B 의 형태)
- Crossover Through : 상호 간 같은 네트워크 장비끼리 연결에 사용되는 케이블 방식.(케이블 양끝이 T568B, T668A의 형태)
- Serial : WAN 구간에서 Route를 서로 연결하는 케이블.
- Routing : 데이터를 목적지까지 전달 시키기 위한 최적의 경로(Best Route)를 결정하는 과정으로 Static Routing, Dynamic Routing이 있다.
- Routing Protocol : 경로를 동적으로 구성해 "라우팅을 수행"하는 프로토콜
- Routed Protocol : Routing Protocol에 의해 Routing이 되는 네트워크 프로토콜