# LVM 심화

- LVM의 가장 큰 특징은 디스크를 현재 가용중이더라도 용량의 확장이 가능하다.

## VG 확장 및 축소

- 아래와 같은 상태에서 확장 및 축소를 진행

![화면 캡처 2022-05-12 111105](https://user-images.githubusercontent.com/57117748/168086159-33646ee1-1066-4fe9-841b-7b3a32f61f2b.png)



* 확장 extend 
```
1. vgextend : 해당 볼륨그룹에 PV 파티션을 추가

2. lvextend : 논리적 저장공간을 확장한다. 

3. 파일 시스템 확장 : lv 에 새로 추가된 pv(파티션)에게 기존 lv 가 가진 파일 시스템과 같은것으로 확장한다.

```

1. 기존의 VG에 PV 파티션을 추가.
```
# vgextend vg이름 파티션경로
```
![화면 캡처 2022-05-12 111007_LI](https://user-images.githubusercontent.com/57117748/168086272-10d7ef4d-cb64-4d07-b036-9ee4bab89f5f.jpg)


2. LV의 사이즈 확장
```
# lvxtend vg이름 -L 사이즈크기 or -l 블록수 lv경로 -r(파일시스템도 같이 확장)
```
![화면 캡처 2022-05-12 111308_LI](https://user-images.githubusercontent.com/57117748/168086876-0eaed1d7-cbe6-45d1-b647-056ae7077d61.jpg)

3. 해당 lv 파일 시스템 확장
	- xfs_grow 마운트 경로	
	- resize2fs lv경로 
```
# xfs_grow 마운트 경로			// xfs 계열
						or
resize2fs lv경로 				// ext4 계열
```

![화면 캡처 2022-05-12 111324_LI](https://user-images.githubusercontent.com/57117748/168088649-cdafd7f2-0452-4740-905a-6dc475d9959f.jpg)

4. lsblk 확인
![화면 캡처 2022-05-12 224512](https://user-images.githubusercontent.com/57117748/168089622-b6126a16-3cbb-4922-9c6a-bfd2d9f5e739.png)



* 축소 reduce
	- 축소를 하는것은 테이터 파일이 손상이 있을 수 있어 권장되지 않음

```
1. 대체 할 수있는 vg 생성 후 축소하려는 파티션의 pvmove 를 통해 pv를 옮긴다.

2. vgreduce 실행
```

1. 대체 vg 생성 후 pvmove 를 이용해 파일 옮기기
	- 옮기는 파일을 운영체제가 임의로 VG 안의 디스크에 옮긴다.

![화면 캡처 2022-05-12 112449](https://user-images.githubusercontent.com/57117748/168090957-4c78df54-0943-40fc-9b9f-2443fdc1e025.png)

- pvmove 실행 확인

![화면 캡처 2022-05-12 225443](https://user-images.githubusercontent.com/57117748/168091661-9b5dfd94-c400-49e8-9fe7-ab76031fb29e.png)


2. vgreduce 실행

![화면 캡처 2022-05-12 112622_LI](https://user-images.githubusercontent.com/57117748/168092339-dc2ac49f-df0f-443a-882f-f1b6f7038816.jpg)


# 리눅스의 시스템 부팅

```
전원켜짐 -> BiOS 단계 -> 부트로더 단계 -> 커널 초기화 단계 -> systemd 서비스 단계 -> 로인인 프롬프트 출력
```

1. BIOS
	- 전원이 켜지면 기본적인 하드웨어 상태를 체크 후 부팅장치를 선택하여 부팅 디스크의 MBR 을 로딩한다.
	- 이때 MBR 은 부트로더의 위치를 저장하고 있다.

2. 부트로더 
	- BIOS 단계에 MBR은 부트로더를 찾아 메모리에 로딩
	- 운영체제 커널을 로딩하는 단계
	- 리눅스의 커널 로더는 /boot 에 grub 파일로 존재 

3. systemd 서비스 단계
	- 리눅스가 본격적으로 실행되며 systemd 가 부팅을 위한 다양한 서비스를 실행
	- systemd 는 기존의 init 을 대체
	- 최고 상위의 부모 프로세스  PID : 1
	- 어떠한 부팅 과정을 거쳤는지 확인을 위한 명령어 $ dmesg
	- 서비스 완료 후 로그인 프롬프트 출력

## systemd 서비스

- init
```
구식 버전의 부팅 로더
systemd 에 의해 대체됨
현재도 /etc/init 에 파일이 남아있고 systemd 에 의해 조금 사용된다.
```

- init 프로세스와 Run Level 
```
init 프로세스에서 사용하던 런레벨(Run Level)의 개념에 대한 이해 필요
------------------------------------------------------------------

런레벨		의미						관련 스크립트 위치 	런레벨 변경 명령어
0			poweroff				/etc/rc0.d 			init 0
1			단일 사용자 모드			/etc/rc1.d 			init 1
2			다중 사용자 모드 nfs X 	/etc/rc2.d 			init 2
3			다중 사용자 모드 nfs O 	/etc/rc3.d 			init 3
4			사용하지 않는 예비번호 	/etc/rc4.d 			init 4
5			그래픽 유저 모드 			/etc/rc5.d 			init 5
6			reboot				 	/etc/rc6.d 			init 6
```

- 현재의 셀에서 런레벨을 확인하는 법
```
$ runlevel or $ systemctl get-defualt

3 5

이 전의 런레벨과 지금의 런레벨이 나온다.
```
![화면 캡처 2022-05-12 124947](https://user-images.githubusercontent.com/57117748/168101663-328007f9-82f0-430a-ac25-7acaf25e29fa.png)


- 현재 런레벨 변경
```
# init 런레벨
```
![화면 캡처 2022-05-12 141004](https://user-images.githubusercontent.com/57117748/168113733-0f58869f-adef-4092-863d-0f51955b1c07.png)



- 기본 런레벨 설정
```
# systemctl set-default target명 
```

![화면 캡처 2022-05-12 140723](https://user-images.githubusercontent.com/57117748/168113561-2a37e68a-b02d-4108-97ea-35044d8580fe.png)


- systemd
	1. centOS 7 부터 init 을 대체
	2. 전체 시스템을 실행하고 괸리하는데 unit 이라는 구성 요소를 사용한다.
	* service 는 가장 명백한 유닛으로 데몬을 시작, 종료, 재시작 로드 등 을 한다.

## systemctl
- systemd 를 기반으로 서비스를 시작하거나 종료에 쓰는 명령
- 우분트에서는 service 와 같은 명령

```
systemclt -a 				// 모든 서비스
systemclt status 서비스명	// 해당 서비스의 상태확인
systemclt start 서비스명		// 해당 서비스 실행
systemclt restart 서비스명	// 해당 서비스 재실행
systemclt stop 서비스명		// 해당 서비스 정지
systemclt enable 서비스명	// 해당 서비스 부팅시 실행
systemclt disable 서비스명	// 해당 서비스 부팅시 무시
systemclt mask 서비스명		// 해당 서비스 사용 불가
systemclt unmask 서비스명	// 해당 서비스 사용 불가 해제

```

- systemclt status

![화면 캡처 2022-05-12 122713_LI](https://user-images.githubusercontent.com/57117748/168100397-7f6ee08b-9d1c-4601-b74f-f9a4cb0f4366.jpg)

- systemclt stop

![화면 캡처 2022-05-12 123057_LI](https://user-images.githubusercontent.com/57117748/168100400-7adac504-5024-461d-b0dd-6054a34ebd19.jpg)

- systemclt disable


![화면 캡처 2022-05-12 123442_LI](https://user-images.githubusercontent.com/57117748/168100669-fba16cdc-0710-4332-a326-a1b1a19ad327.jpg)

- systemclt mask, restart

![화면 캡처 2022-05-12 123654_LI](https://user-images.githubusercontent.com/57117748/168101064-5fdc99f6-1ef5-496e-8ff0-c8f37e556287.jpg)



## systemd target unit (런레벨)
```
------------------------------------------------------------------

런레벨		의미						심볼릭 링크 					target 원본
0			poweroff				runlevel0.target 			poweroff.target
1			단일 사용자 모드			runlevel1.target  			rescue.target
2			다중 사용자 모드 nfs X 	runlevel2.target  			multi-user.targer
3			다중 사용자 모드 nfs O 	runlevel3.target  			..
4			사용하지 않는 예비번호 	X  							X
5			그래픽 유저 모드 			runlevel5.target 			graphical.target
6			reboot				 	runlevel6.target  			reboot.targer
```

- 기본 타겟 변경
	* 부팅시 세팅 할 runlevel 
```
$ systemctl set-default target명
```

![화면 캡처 2022-05-12 140723](https://user-images.githubusercontent.com/57117748/168103864-69cbe80f-56a0-4f47-8fa7-0aadbe0a5df1.png)

- 현재 타겟 변경
	* 현재의 런레벨 변경


## systemctl isolate target명 or runlevelNmber


![화면 캡처 2022-05-12 141104](https://user-images.githubusercontent.com/57117748/168104552-32ac9841-0777-4e60-8487-dcaa7383f9bf.png)
![화면 캡처 2022-05-12 141656](https://user-images.githubusercontent.com/57117748/168104558-ac3c49e8-78a6-4d8c-bc52-49057bd2f23f.png)


## 리눅스 종료
- shutdown -h 명령을 통해 종료한다.

```
$ shutdown -h now
$ shutdown -h +3 "이제 곧 종료됩니다."	// 3분후에 종료시키며 메세지 출력

```

## 데몬 프로세스
- 백그라운드에서 작업하는 프로세스
- 시스템에서 동작하는 각종 서비스가 데몬 프로세스이다.
- 동작방식은 슈퍼데몬만 동작하다가 서비스의 요청이 오면 해당 데몬을 동작시키고 완료하면 다시 해당 데몬은 멈춘다.



# root 계정의 비밀번호 찾기
1. 부트로드 화면에서 e 를 눌러 GRUB Editor Mod 로 진입

2.Linuxx16 의 위치를 찾아 해당 행의 끝으로 가서 rd.break 명령어 입력 후 ctrl + x 로 이머전시 모드 진입

![화면 캡처 2022-05-13 004513](https://user-images.githubusercontent.com/57117748/168115675-ae7206d7-f0c3-4684-95fe-e8dfaf3161cd.png)


3. mount -o rw,remount /sysroot
4. chroot /sysroot

![화면 캡처 2022-05-13 004900](https://user-images.githubusercontent.com/57117748/168116350-7abc0363-d289-4e4d-9e73-809a086a08a3.png)

5. passwd 
6. 새로운 암호 입력 X 2
7. touch / .autorelabel
8. ctrl + d 를 이용해 종료

![화면 캡처 2022-05-13 005205](https://user-images.githubusercontent.com/57117748/168117025-2f49da87-0d83-459e-a4ee-dd6e2996af0a.png)


# 사용자 및 그룹관리 

- 사용자 관리 : 다중 사용자 시스템이기 때문에 적절한 자원 할당이 필요하다.

## /etc/passwd
- 사용자의 계정정보가 저장된 기본파일

- /etc/passwd 구조

```
로그인 ID : X : UID : GID : 설명 : 홈 디렉토리 : 로그인 셀

1. 로그인 ID : 계정이름으로 32자를 넘지 못하지만 주로 8자 이하로 권장된다.
2. X : 유닉스에서 사용하던 사용자 암호를 기입하던 공간 지금은 사용하지 않는다.
3. UID : User ID 시스템이 사용자를 구별하기 위한 ID
	- 0~999와 65534 는 시스템 사용자를 위한 ID 이다 (0:root, 1:bin)
	- 일반 사용자는 1000 부터 시작한다
	- 로그인 ID 가 달라도 UID 가 같다면 시스템은 동일 사용자로 판단
4. GID : Group ID 해당 계정이 속한 주 그룹 ID
5. 설명 : 해당 계정 설명 주석처리
6. 홈 디렉토리 : 사용자 계정에 등록된 절대 경로
7. 로그인 쉘 : 기본 쉘
```

![화면 캡처 2022-05-13 000309](https://user-images.githubusercontent.com/57117748/168106607-06c8a241-ffa3-4be9-9640-356489c77faa.png)


## /etc/shadow

- 사용자의 패스워드를 저장하는 파일

- /etc/shadow 구조

```
로그인 ID : 암호 : 마지막 수정일 : 최소 사용 기간 : 최대 사용 기간 : 경고 : inactive : expire : flag

1. 로그인 ID : 계정이름
2. 암호 : 해시 암호화를 거친 패스워드 
	- 일방향이 아닌 복호화가 가능한 양방향 암호 $passwdalgorithm$salt$code 로 구성 
3. 마지막 수정일 : 1970 =.01.01 기준으로 날수를 기록하는 마지막 변경 날자
4. 최소 사용 기간 : 암호를 변경 후 최소로 사용해야 하는 기간
5. 최대 사용 기간 : 암호를 변경 후 최대로 사용할 수 있는 기간
6  경고 : 암호가 만료되기전 경고를 시작하는 날 수
7. inactive : 암호가 만료되고 계정을 사용 할 수 있는 유예기간
8. expire : 사용자 계정의 만료 날자
9  flag : 향후를 위한 빈 공간
```

## /etc/login.defs
- 계정을 생성을 위한 기본 설정값 저장 파일

![화면 캡처 2022-05-13 001852](https://user-images.githubusercontent.com/57117748/168109847-1ba5f407-e88d-4c78-9142-692206a4ab82.png)

## /etc/group
- 그룹에 대한 정보를 저장
```
그룹이름 : x : GID : 보조그룹

1. 그룹이름 : 그룹이름
2. x : 그룹의 암호 저장
3. GID : group id
4. 그룹맴버 : 보조그룹 표시
```
![화면 캡처 2022-05-13 002219](https://user-images.githubusercontent.com/57117748/168110591-83820965-24bf-4221-aded-e73352c1ef40.png)


## 사용자 계정 생성
- useradd 명령을 통한 사용자 계정 생성

![화면 캡처 2022-05-13 002514](https://user-images.githubusercontent.com/57117748/168111216-6548ef7b-8bf4-4582-a111-86c6bdd922c7.png)

* 항상 계정 설정후 passwd 명령을 통해 암호 부여

## useradd -D
- 사용자 계정 생성시 기본값 설정
![화면 캡처 2022-05-13 002930](https://user-images.githubusercontent.com/57117748/168112208-145442bf-b140-4e49-9686-f17139e63874.png)
```
1. Group : 기본 등록 그룹의 GID로 100 은 users 그룹이다.
2. Home : 홈 디렉터리 생성 위치
3. Inactive : -1 이면 비활성 상태 0 이면 암호가 만료되면 계정이 잠긴다
4. Expire : 계정 종료일
5. Shell : 기본 쉘
6. Skel : 홈 디렉터리에 이 경로의 폴더의 파일을 기본으로 붙여 넣는다.
7. CREATE_MAIL_SPOOL : 메일 디렉터리의 생성 여부를 지정한다.
```

## 연습

- 문제

![화면 캡처 2022-05-12 173715](https://user-images.githubusercontent.com/57117748/168118121-3a316d09-4341-474e-ace3-21bfd59e0735.png)

- 풀이

![화면 캡처 2022-05-12 173728](https://user-images.githubusercontent.com/57117748/168118192-ba46af25-8947-44a6-8b90-b152dae60cb2.png)

## 키워드

vgextend : LVM 의 핵심 기술로 디스크가 가용중이더라도 확장을 할 수 있다.

vgreduce : vg를 축소시킨다.

BIOS : 메인보드가 처음 디스크를 확인하고 MBR 을 찾고 부트로더를 찾는 과정

부트로더 : 운영체제 커널을 로딩하는 과정

systemd : 리눅스가 시작되면서 시스템 실행에 필요한 서비스를 실행하는 과정으로 centos7 전 version은 init의 과정이다

runlevel : 총 7단계로 나누어 지며 init 시절에 사용하던 시스템 로딩 및 종료 명령이다.

systemd target unit : init 이 사라지면서 systemd 에서 사용하는 시스템 로딩 및 종료 명령이다.

systemctl : 시스템 서비스 명령 체계

데몬 프로세스 : 백그라운드에서 작업한는 프로세스. 시스템 실행에 필요한 프로세스들이 주로 있다. 

/etc/passwd : 사용자의 계정정보가 저장되어 있는 파일

/etc/shadow : 사용자의 암호가 저장되어 있는 파일

/etc/group : 그룹의 정보가 저장되어 있는 파일

useradd : 사용자를 추가 생성하는 명령어 -s -md -g -G -u

useradd -D : 사용자 계정을 생성하는데 기본값 확인 및 변경가능 

