# Docker life cycle

![Docker life cycle](https://miro.medium.com/max/1400/1*djgbdaqV3oR40wbGTQYk3w.png)

# 도커의 리소스 사용량 제한
- 기본적으로 container들은 같은 호스트의 H/W 리소스를 공유하기 때문에 App마다 할당량 조정 가능.
- container의 리소스 독점을 제한함으로 안정성을 유지시킨다.
- --cpu-share / --memory
```
$ docker container run --cpu-share=512	// cpu 리소스 할당 

$ docker container run --memory=1g	// cpu 리소스 할당 
```

## Docker container update
- 리소스 제한을 한 리소스의 할당량을 조절(할당량을 조절 하지 않았다면 컨테이너 중지 후 작업할 필요 있음) 
```
$ docker container update --cpu-share=512	// cpu 리소스 수정

$ docker container run --memory=1g	// cpu 리소스 할당 

$ docker container update --memory-swap	// 할당 하지 않은 컨테이너 리소스 할당 그러나 먼저 컨테이너를 멈춰야 한다
```
![화면 캡처 2022-06-14 123314_LI](https://user-images.githubusercontent.com/57117748/173533109-37589e3e-961c-4636-b650-d60dbfca7203.jpg)

## Docker container stats
- 리소스의 사용량 확인 
```
$ docker container stats --no-stream	// --no-stream 실시간 확인 하지 않음
```
![화면 캡처 2022-06-14 121941](https://user-images.githubusercontent.com/57117748/173533881-a8708569-2485-46be-9783-af475488edf0.png)

# 환경변수
- run 동작시 -e 를 통해 환경변수 지정 가능
- '이름=값'
```
$ docker container run -it -e TEST_VAR=1234 centos:7
```
![화면 캡처 2022-06-14 124008](https://user-images.githubusercontent.com/57117748/173536274-b6d4b688-41e4-473d-9a3e-2403b7a7a389.png)


- 이미지의 따라서는 필수적인 경우가 존재 Ex) MySQL

```
$ docker container run -d -e MYSQL_ROOT_PASSWORD=1234 mysql:5.6	//MySQL은 필수
```
![화면 캡처 2022-06-14 124301_LI](https://user-images.githubusercontent.com/57117748/173535938-f438af9e-0e3f-4159-b5ef-c46e73e1e6a6.jpg)

# Docker Network
- Docker의 Container들의 네트워크 지정 및 수정을 하기위한 명령어

## Docker Network의 기본 네트워크
1. Network Bridge : :컨테이너들의 네트워크를 설정한다 NIC와 컨테이너 사이에서 연결하는 역할을 하며 각 컨테이너를 하나의 대역으로 묶어준다
2. Network Host : 사용자가 사용하는 localhost와 같은 네트워크를 공유하는 네트워크 
3. Network none : 네트워크 연결을 하지않는 컨테이너에게 주어지는 네트워크


## Network Bridge / 네트워크 브릿지
- 네트워크 브릿지

![화면 캡처 2022-06-14 151649_LI](https://user-images.githubusercontent.com/57117748/173544551-ee45b245-9b55-48bb-b1f9-2b9bb2758f91.jpg)

- 네트워크의 기본 브릿지 3개중 하나로 도커 네트워크를 만들때 defualt 값이 되는 네트워크이다
- 네트워크 생성 시 대역, 게이트웨이 주소를 자동으로 부여한다

## Network Host	/ 네트워크 호스트
- 현재 사용자가 사용하는 인터페이싀 주소를 컨테이너가 함께 공유하는 방식
- 포트 표시되지 않음(하나의 IP가 동일하기에 같은 포트를 사용한다면 복수의 같은 이미지를 쓸 수 없다 : 포트포워딩시 가능 docker container run -p 옵션)
```
# docker container run -d --name host_web --network host httpd:latest
de50e2fb4c07c3dd1967c74348c62639ba43c884e447be5954fdd2bf498d49b2


// host_web의 host natwork 확인
[root@localhost ~]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS             PORTS      NAMES
de50e2fb4c07   httpd:latest   "httpd-foreground"       10 minutes ago      Up 10 minutes                 host_web
1060d9e26e84   httpd          "httpd-foreground"       59 minutes ago      Up 59 minutes      80/tcp     web
f79183b06dc4   centos:7       "/bin/bash"              About an hour ago   Up About an hour              centos2
200a2f30f3db   centos:7       "/bin/bash"              About an hour ago   Up 54 minutes                 centos1
ff08cc6da53c   mysql:5.6      "docker-entrypoint.s…"   About an hour ago   Up About an hour   3306/tcp   DB


# docker network inspect host 		// network 섹션에 아무것도 없다
[
    {
        "Name": "host",
        "Id": "ce63d7d36e98a2921cc63a33310517ffcae30086db13e4eb16e89e3c48ad597d",
        "Created": "2022-06-13T15:18:08.117168923+09:00",
        "Scope": "local",
        "Driver": "host",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": []
.
.
.
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "d9459e4087add15ebe1e4d0fad843119c7d1ba70e861fdfaa397bb4ed84457c2",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/default",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "",
            "Gateway": "",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "",
            "IPPrefixLen": 0,
            "IPv6Gateway": "",
            "MacAddress": "",
            "Networks": {
                "host": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "ce63d7d36e98a2921cc63a33310517ffcae30086db13e4eb16e89e3c48ad597d",
                    "EndpointID": "93ceac67b3acf02ed9f6b3babddc33efa2ac3e48987fa6d15553e1d55179e895",
                    "Gateway": "",
                    "IPAddress": "",
                    "IPPrefixLen": 0,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "",
                    "DriverOpts": null
                }
            }
        }
    }
]


// 로컬 호스트의 IP
enp0s8: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.56.109  netmask 255.255.255.0  broadcast 192.168.56.255
        inet6 fe80::5c73:1495:db80:9339  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:6a:fb:f6  txqueuelen 1000  (Ethernet)
        RX packets 7441  bytes 633420 (618.5 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4903  bytes 640663 (625.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

enp0s17: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.2.11  netmask 255.255.255.0  broadcast 10.0.2.255
        inet6 fe80::6b7e:bdae:df70:d1b1  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:3a:40:54  txqueuelen 1000  (Ethernet)
        RX packets 343  bytes 150529 (147.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 371  bytes 68320 (66.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

// 호스트에서 curl 을 통한 확인 (호스트에 httpd 없음)
[root@localhost ~]# curl 192.168.56.109
<html><body><h1>It works!</h1></body></html>
[
root@localhost ~]# curl 10.0.2.11
<html><body><h1>It works!</h1></body></html>
```

## Netowkr None / 네트워크 None
- 네트워크를 연결하지 않을 컨테이너에게 지정하는 네트워크 
- 네트워크에 연결 되지 않은 상태


## 네트워크 생성
```
$ docker network create [name]	// bridge 기본
```

- 옵션
```
--subnet : 네트워크를 만들때 해당 네트워크의 대역폭을 직접 지정한다.
--gateway : 네트워크를 만들때 해당 네트워크 게이트웨이 주소 설정 가능
--dns : 네트워크를 만들때 해당 네트워크 DNS 주소 설정 가능
--link : 브릿지타입의 경우 같은 브릿지에 연결한 컨테이너에 대해 이름으로 통신 가능하게 설정(이후 해당 컨테이너이 주소를 변경하거나 삭제후 다시 만들어도 이름을 똑같이 지정한 경우 연결)
--expose : 실시간 포트 오픈
-p : 브릿지타입의 경우 포트매핑 설정
```

## 다른 네트워크끼리 연결시키기
```
$ docker network connect [networkname][container]	// 네트워크와 container 연결

$ docker network disconnect [networkname][container]	// 네트워크와 container 연결해제
```

## 네트워크 삭제
- 기본 3개의 네트워크는 삭제 불가하다
- 사용중인 네트워크는 삭제가 불가하다
```
$ docker network rm
$ docker network prune	// 사용하지 않는 네트워크 삭제
```

# 연습1
1.네트워크 만들기
-
```
# docker network create --subnet 172.100.0.0/16 external_net
b402b61220b67256513764bf281c7c47bd5f7844f7ed96a036db456ec0eba63c


# docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
d35b9ad7c12c   bridge         bridge    local
b402b61220b6   external_net   bridge    local
ce63d7d36e98   host           host      local
6f1e23171045   none           null      local


[root@localhost ~]# docker network inspect external_net
.
.
.
[
    {
        "Name": "external_net",
        "Id": "b402b61220b67256513764bf281c7c47bd5f7844f7ed96a036db456ec0eba63c",
        "Created": "2022-06-14T14:45:53.912551137+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.100.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

- internal_net이름의 네트워크 만들기 이떄 IP 대역폭은 192.168.10.0/24  이다. 
```
# docker network create --subnet 192.168.10.0/24 internal_net
2e8157659966bb71671e868a0705e1865259ee5acdff301e0ca1e845b993da29

# docker network ls
NETWORK ID     NAME           DRIVER    SCOPE
d35b9ad7c12c   bridge         bridge    local
b402b61220b6   external_net   bridge    local
ce63d7d36e98   host           host      local
2e8157659966   internal_net   bridge    local
6f1e23171045   none           null      local

# docker network inspect internal_net
[
    {
        "Name": "internal_net",
        "Id": "2e8157659966bb71671e868a0705e1865259ee5acdff301e0ca1e845b993da29",
        "Created": "2022-06-14T14:46:48.2962918+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.10.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

2. 컨테이너 생성
- httpd 이미지 사용해 컨테이너 생성하기 이때 네트워크는 external_net
```
# docker container run -d  --name web --network external_net httpd
d08731fd65c55d8183651e107a6dd08056aabf9588a54435e2c3de9c3de38621
```

- mysql 이미지를 사용해서 컨테이너 생성하기 이때 네트워크는 internal_net
```
# docker container run -e MYSQL_ROOT_PASSWORD=1234 -it -d --name DB --network  internal_net mysql:5.6
ff08cc6da53cb03044d971a365b58f006927a4c3f17fa53090d5e0b06961779b
```

- centos 이미지를 사용해서 컨테이너 만들기 이때 네트워크는 external_net
```
# docker container run -it -d --name centos1 --network external_net centos:7
200a2f30f3dbd303c94cf8b076995077648dc2ae8f9d7f7c977fdcf449dc0a7c
```

- centos 이미지를 사용해서 컨테이너 만들기 이때 네트워크는 internal_net
```
# docker container run -it -d --name centos2 --network internal_net centos:7
f79183b06dc4758e11013f258339617b91ce83f884720b5564cf0db6fd02b536
```

3. 네트워크 확인
- external_net 네트워크 확인 
```
# docker network inspect external_net
[
    {
        "Name": "external_net",
        "Id": "b402b61220b67256513764bf281c7c47bd5f7844f7ed96a036db456ec0eba63c",
        "Created": "2022-06-14T14:45:53.912551137+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.100.0.0/16"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "200a2f30f3dbd303c94cf8b076995077648dc2ae8f9d7f7c977fdcf449dc0a7c": {
                "Name": "centos1",
                "EndpointID": "448934fd2a6d08d4548e5efba18c3821dbb95f8e45659274ad2703d9e783b705",
                "MacAddress": "02:42:ac:64:00:03",
                "IPv4Address": "172.100.0.3/16",
                "IPv6Address": ""
            },
            "d08731fd65c55d8183651e107a6dd08056aabf9588a54435e2c3de9c3de38621": {
                "Name": "web",
                "EndpointID": "7f68d8f68052268eab75b85834c8e5cb2da42a4de7929cbc816d32457c158f91",
                "MacAddress": "02:42:ac:64:00:02",
                "IPv4Address": "172.100.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

- internal_net 네트워크 확인
```


# docker network inspect internal_net
[
    {
        "Name": "internal_net",
        "Id": "2e8157659966bb71671e868a0705e1865259ee5acdff301e0ca1e845b993da29",
        "Created": "2022-06-14T14:46:48.2962918+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.10.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "f79183b06dc4758e11013f258339617b91ce83f884720b5564cf0db6fd02b536": {
                "Name": "centos2",
                "EndpointID": "30d0030c79077a90bb7f0bcab48627964c872690957d3d036b5f4c1450fc112f",
                "MacAddress": "02:42:c0:a8:0a:03",
                "IPv4Address": "192.168.10.3/24",
                "IPv6Address": ""
            },
            "ff08cc6da53cb03044d971a365b58f006927a4c3f17fa53090d5e0b06961779b": {
                "Name": "DB",
                "EndpointID": "67e3a36f8af2117b6c96b181fb587a5148897e1ca579ccde3eed27236384dad8",
                "MacAddress": "02:42:c0:a8:0a:02",
                "IPv4Address": "192.168.10.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

4. 다른 네트워크를 연결
- 연결 구조
![화면 캡처 2022-06-14 152443](https://user-images.githubusercontent.com/57117748/173551822-732850cb-9d78-4f10-a09d-5a829a27d035.png)


- centos1을 internal_net에 연결
```
# docker network connect internal_net centos1	
```

- 연결 확인
```
# docker network inspect internal_net
[
    {
        "Name": "internal_net",
        "Id": "2e8157659966bb71671e868a0705e1865259ee5acdff301e0ca1e845b993da29",
        "Created": "2022-06-14T14:46:48.2962918+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.10.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "200a2f30f3dbd303c94cf8b076995077648dc2ae8f9d7f7c977fdcf449dc0a7c": {
                "Name": "centos1",
                "EndpointID": "435e96830e91c3b317576a09fb1e76370020c7b472df821a9c3828d000bf0a2e",
                "MacAddress": "02:42:c0:a8:0a:04",
                "IPv4Address": "192.168.10.4/24",
                "IPv6Address": ""
            },
            "f79183b06dc4758e11013f258339617b91ce83f884720b5564cf0db6fd02b536": {
                "Name": "centos2",
                "EndpointID": "30d0030c79077a90bb7f0bcab48627964c872690957d3d036b5f4c1450fc112f",
                "MacAddress": "02:42:c0:a8:0a:03",
                "IPv4Address": "192.168.10.3/24",
                "IPv6Address": ""
            },
            "ff08cc6da53cb03044d971a365b58f006927a4c3f17fa53090d5e0b06961779b": {
                "Name": "DB",
                "EndpointID": "67e3a36f8af2117b6c96b181fb587a5148897e1ca579ccde3eed27236384dad8",
                "MacAddress": "02:42:c0:a8:0a:02",
                "IPv4Address": "192.168.10.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```

- centos1 에서 internal_net의 컨테이너로 ping 보내기
``` 
[root@200a2f30f3db /]# ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
64 bytes from 192.168.10.2: icmp_seq=1 ttl=64 time=0.103 ms
64 bytes from 192.168.10.2: icmp_seq=2 ttl=64 time=0.162 ms
64 bytes from 192.168.10.2: icmp_seq=3 ttl=64 time=0.161 ms
64 bytes from 192.168.10.2: icmp_seq=4 ttl=64 time=0.134 ms
64 bytes from 192.168.10.2: icmp_seq=5 ttl=64 time=0.101 ms
64 bytes from 192.168.10.2: icmp_seq=6 ttl=64 time=0.164 ms

```

5. 네트워크 해제
```
# docker network disconnect internal_net centos1
```

6. 네트워크 해제 확인
```
# docker network inspect internal_net
\[
    {
        "Name": "internal_net",
        "Id": "2e8157659966bb71671e868a0705e1865259ee5acdff301e0ca1e845b993da29",
        "Created": "2022-06-14T14:46:48.2962918+09:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.10.0/24"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "f79183b06dc4758e11013f258339617b91ce83f884720b5564cf0db6fd02b536": {
                "Name": "centos2",
                "EndpointID": "30d0030c79077a90bb7f0bcab48627964c872690957d3d036b5f4c1450fc112f",
                "MacAddress": "02:42:c0:a8:0a:03",
                "IPv4Address": "192.168.10.3/24",
                "IPv6Address": ""
            },
            "ff08cc6da53cb03044d971a365b58f006927a4c3f17fa53090d5e0b06961779b": {
                "Name": "DB",
                "EndpointID": "67e3a36f8af2117b6c96b181fb587a5148897e1ca579ccde3eed27236384dad8",
                "MacAddress": "02:42:c0:a8:0a:02",
                "IPv4Address": "192.168.10.2/24",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

```

6. 네트워크 해제 후 centos1 에서 internal_net의 컨테이너로 ping 보내기
- centos1 에서 internal_net의 컨테이너로 ping 보내기
#다른 네트워크를 해제 후 핑 보내보기
[root@200a2f30f3db /]# ping 192.168.10.2
PING 192.168.10.2 (192.168.10.2) 56(84) bytes of data.
^C
--- 192.168.10.2 ping statistics ---
12 packets transmitted, 0 received, 100% packet loss, time 10999ms

# docker network create host
docker container run -d --name host_web --network host httpd:latest

# docker network inspect host
