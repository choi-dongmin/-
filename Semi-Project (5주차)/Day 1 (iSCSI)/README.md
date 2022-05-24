# iSCSI 개요
- 블록 스토리지 공유. 설정. 구성하는 패키지 툴  
- 스토리지 제공방식(DAS, NAS, SAN) 중 SAN의 대표적인 IP-SAN 방식.
	1. DAS (Direct Access Storage) 
		* 직접 케이블을 스토리지와 연결하는 방식
		* 장 : 편리하고 기본적인 연결 방식
		* 단 : 공간적, 거리적 제약이 있다
	2. NAS (Network Access Storage)
		* 네트워크로 시스템과 스토로지를 연결
		* 사용 가능한 구성(파일 시스템 설치) 후 네트워크 제공
		* 장 : 거리의 제약이 사라짐
		* 단 : 속도가 느리고 보안이 취약하다
	3. SAN (System Access Netwok)
		* 네트워크로 시스템과 스토리지를 연결
		* 스토리지 자체를 제공 -> 클라이언트가 직접 파일 시스템 포맷
		* FC-SAN : 광섬유 장비들을 이용한 네트워크 구성 : 속도향상, 비용증가
		* IP-SAN : TCP 프로토콜을 이용한 연결방식 : 속도향상, 비용유지

## ISCSI 용어정리
- 기본 용어
	1. 타겟 : target
		- 스토리지 장치를 제공하는 시스템으로 서버와 같은 역할
	2. 이니시에이터 : initiator 
		- 타겟에서 제공하는 스토리지 장치에 연결해 블록 스토리지를 제공받는 시스템으로 클라이언트와 같은 역할


- 타겟 용어
	1. TPG : Target Portal Group
		- 대상 포털 그룹으로서 ACL, LUN, Portal 세 항목을 하나의 그룹으로 만든 연결에 대한 설정 집
	2. ACL : Access Contol List 
		- 접근제어 리스트로서 타겟에 의해 제동되는 스토리지에 연결할 수 있는 초기자 즉, 이니시에이터를 지정하는 항목
	3. LUN : Logical Unit Number 
		- 초기자에게 제공할 스토리지 장치에게 부여된 논리 장치 번호
	4. Portal 
		- 초기자가 타겟에 연결할 떄 사용하는 IP 주소와 포트번호

- 초기자 용어
	1. Discovery 
		- 초기자에서 연ㄴ결하려는 대상을 검색하기 위한 단계
	2. Login
		- Discovery에서 발견한 대상으로 연결하는 단계

## IQN 주소 
- IQN 주소는 ISCSI Qualified Name의 약자로 특정 ISCSI Target과 Initiator 호스트르 구별하기 위한 주소이다.

```
iqn.YYYY-MM.[메인 도메인의 역순]:[서브도메인 혹은 설명]

iqn.2022-05.com.goorm:target
```

# iSCSI 구성 순서
- 기본 설정 with LVM

0 .target 주소 및 호스트의 이름 설정
```
# nmcli connection add type ethernet con-name nat ifname enp0s3 ipv4.addresses 10.0.2.100/24 ipv4.gateway 10.0.2.1 ipv4.dns 8.8.8.8 ipv4.method manual
# nmcli c up nat

# hostnamectl set-hostname target.goorm.com
```
![화면 캡처 2022-05-23 173619](https://user-images.githubusercontent.com/57117748/169781205-fc2bbcf1-d0cf-4112-9e2f-d37337c85f16.png)

1. 패키지의 설치
```
# yum -y install targetcli
```

2.  스토리지를 준비 
```
# fdisk [/dev/sdb : 장치경로]	// 파티셔닝 진행
# partprobe /dev/sdb
# pvcreate [/dev/sdb{1..3} : 파티셔닝 경로]	// 물리 볼륨 만들기
# vgcreate [/dev/sdb1 /dev/sdb/2 : VG를 위한 PV 경로]	// 볼륨 그룹 만들기
# lvcreate -L [1G : 1G 크기의 논리볼륨] -n [LVname]	// 논리 볼륨 만들기
```
- 파티셔닝
![화면 캡처 2022-05-23 180322](https://user-images.githubusercontent.com/57117748/169784155-0486ceff-a1ef-49c2-821a-85b975643a11.png)

- pv
![pv](https://user-images.githubusercontent.com/57117748/169784225-6a714cec-377f-4f9f-a914-c3bfa3d10a1d.png)

- vg
![화면 캡처 2022-05-23 182717](https://user-images.githubusercontent.com/57117748/169788705-b28426cf-6da5-471e-a7eb-937511e0e3f8.png)

- lv
![화면 캡처 2022-05-23 183059](https://user-images.githubusercontent.com/57117748/169789426-bdee5fc6-f5f8-4da2-b9be-ef643d391f37.png)


3. targetcli 구성
```
# targetcli	// targetcli 진입
> backstores/block create dev=[/dev/newvg/newlv : LV 경로] name=[exam : blockName]	// Block 설정
> iscsi/ create [iqn.2022-05.com.goorm:target : NewIQN] 	// IQN 설정 (타겟이름)
> iscsi/[iqn.2022-05.com.goorm:target : IQN]/tpg1/acls create [iqn.2022-05.com.goorm:initiator : NewACL] 	// ACL 설정 (초기자 설정)
> iscsi/[iqn.2022-05.com.goorm:target : IQN]/tpg1/luns create /backstores/block/[exam : blockName]	// LUN설정
> iscsi/[iqn.2022-05.com.goorm:target : IQN]/tpg1/portals/ delete 0.0.0.0 3260 // 기본포탈 지우기
> iscsi/[iqn.2022-05.com.goorm:target : IQN]/tpg1/portals/ create [10.0.2.100 : Target 서버 IP] 3260 // 포탈 연결해주기
> exit
```

- Block 설정
![화면 캡처 2022-05-23 184141](https://user-images.githubusercontent.com/57117748/169791785-1ef5d432-acac-4e07-a7c9-f8a592c2a060.png)

- IQN
![화면 캡처 2022-05-23 184458](https://user-images.githubusercontent.com/57117748/169792216-fd8b125c-e204-4a5e-a9e3-ef05eb20dd2d.png)

- ACL
![화면 캡처 2022-05-23 184907](https://user-images.githubusercontent.com/57117748/169793038-612b1218-bd6f-4073-9515-01464a749e3b.png)

- LUN
![화면 캡처 2022-05-23 185648](https://user-images.githubusercontent.com/57117748/169794564-0adfa553-9752-4198-9430-0f3c4cf59d09.png)

- Portal Del / CRE
![화면 캡처 2022-05-23 190006](https://user-images.githubusercontent.com/57117748/169795143-5aa07a50-ee77-4598-83bd-7a7b88d361cf.png)


4. target 활성화 및 방화벽 해제
```
# systemctl restart target
# systemctl enable target
# firewall-cmd --permanent --add-port=3260/tcp
# firewall-cmd --reload
```

5. initiator 주소 및 호스트의 이름 설정
```
# nmcli connection add type ethernet con-name nat2 ifname enp0s3 ipv4.addresses 10.0.2.200/24 ipv4.gateway 10.0.2.1 ipv4.dns 8.8.8.8 ipv4.method manual
# nmcli c up nat2

# hostnamectl set-hostname initiator.goorm.com
```

6. iscsi-initiator-utils 확인
```
# yum list iscsi-initiator-utils
```

7. initiator 구성
- Initiator IQN 설정
```
#vi /etc/iscsi/initiatorname.iscsi
```
![화면 캡처 2022-05-23 192708](https://user-images.githubusercontent.com/57117748/169799988-73a64aff-fadd-4266-b7f2-ffee978c88c7.png)

8. iscsiadm 명령어를 통한 검색, 연결, 확인
```
# iscsiadm -m discovery -t st [10.0.2.100 : Target Server IP]	// 검색
# iscsiadm -m discovery -T node [iqn.2022-05.com.goorm:target : IQN] -l	// 로그인
# iscsiadm -m session -p 3
```
![화면 캡처 2022-05-23 192708](https://user-images.githubusercontent.com/57117748/169804322-66650d38-212f-4788-85e5-0dc38c8561c2.png)
![화면 캡처 2022-05-23 195716](https://user-images.githubusercontent.com/57117748/169804748-c2c46555-ac65-47f3-9280-bb25f0afa67d.png)

9. initiator 에서 파티셔닝 후 파일시스템 포맷 및 마운트
```
# fdisk [/dev/sdb : 장치 경로]
# mkfs.xfs [/dev/sdb1 : 파티셔닝 경로]
```

10. /etc/fstab
```
.
.
.
[/dev/sdb1 : 파티셔닝경로] [/mnt : 마운트 경로] xfs _netdev 0 0
```
![화면 캡처 2022-05-23 161513_LI](https://user-images.githubusercontent.com/57117748/169808226-6a87f75d-48d6-4eb4-a42a-aa72509c8612.jpg)

11. 확인
![화면 캡처 2022-05-23 202436](https://user-images.githubusercontent.com/57117748/169808954-cbf06974-1d67-4281-974d-3ac0f6e5ca32.png)

12. lvm scaling
```
# vgextend newvg /dev/sdb3
# lvextend newvg -L +1G /dev/newvg/newlv
# xfs_growfs [/mnt/ : 마운트 경로]
```

## 오류 해결책
1. 검색 중 오류 
	iscsiadm -m discovery -t st -p 10.0.2.100	-> -p 옵션에서는 타겟이 사용하는 IP주소로 지정
	=> 검색 시 오류 발생 : 네트워크 연결 혹은 방화벽 문제 로그인

2. 연결 중 오류 iscsiadm -m node -T iqn.2022-05.com.goorm:target -l
	=> 인증 오류 발생 : /etc/iscsi/initiatorname.iscsi 파일에 설정한 IQN 과 타겟에서 설정한 ACL 과 일치? 연결 확인

3. iscsiadm -m session -P 3
	=> 마지막 줄에 Attached scsi disk 에서 연결된 장치의 이름 확인

## iscsiadm options 
1. 연결 해제
```
# iscsiadm -m node -T iqn.2022-05.com.goorm:target -u
```

2. 검색 기록 제거
```
# iscsiadm -m node -T iqn.2022-05.com.goorm:target -o
```

3. 리로드(재검색)
```
# iscsiadm -m node -T iqn.2022-05.com.goorm:target -R
```

## 키워드 
- 스토리지 : 데이터를 영구적으로 저장하기 위한 공간(장치)
- 타겟 : 스토리지 장치를 제공하는 시스템으로 서버와 같은 역할
- 이니시에이터 : 초기자 타겟이 제공하는 저장공간을 사용하는 시스템으로 클라이언트와 같은 역할
- iSCSI : IP-SAN 형태의 블록 스토리지를 공유. 설정. 구성하는 패키지 툴
- iSCSI 구성 과정
	1. 타겟 구성
		1) 패키지 설치
		2) targetcli 준비
		3) 서비스 활성화(설정값 초기화 방지)
		4) 방화벽 설정
	2. 이니시에이터 구성
		1) IQN 설정
		2) iscsiadm 검색 및 로그인
		3) 장치 관리(파티셔닝/파일시스템)
