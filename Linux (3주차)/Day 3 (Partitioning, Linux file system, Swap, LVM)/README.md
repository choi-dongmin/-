# 파티셔닝

```
1. 디스크 설치

2. 디스크 인식

3. 파티셔닝	v

4. 파일 시스템 생성 (파일을 만드는 규칙)

5. 마운트
```

- 파티셔닝 : 하나의 물리적 저장장치 공간을 논리적으로 공간을 나누어 쓰도록 하는것.

## 파티셔닝 관련 명령어

```
fdisk 장치경로 : 해당 장치 관리

fdisk -l : 디스크의 현재 목록 및 정보

lsblk -f : 파티션의 관계 정보 및 파일 시스템 정보

```


![fdisk -l](https://user-images.githubusercontent.com/57117748/167830932-76430e58-ffe3-41b6-9123-47d8c58db4ce.jpg)

## 파티셔닝의 과정


1. fdisk [장치경로] : 해당 디스크 파티션 관리로 진입
```
fdisk /dev/sdb
```
![파티셔닝](https://user-images.githubusercontent.com/57117748/167831398-bf32661f-8610-4f3f-9c22-25cc8fd880ff.png)
- fdisk 명령어 모음
```
p : 현재 디스크 파티션 테이블 출력
w : 파티션의 정보 저장 후 종료
q : 파티션 정보 저장하지 않고 종료
n : 새로운 파티션 추가
t : 파티션 시스템 id 값 변경 (파일의 시스템 종류 변경시)
d : 파티션 삭제
l : 사용 가능한 파티션 목록
m : 도움말 출력
```
![화면 캡처 2022-05-11 105145](https://user-images.githubusercontent.com/57117748/167854091-b4f48682-b0e5-4cc4-a03a-d7c3078e58ab.png)


2. n : 새로운 파티션 추가
- n 을 입력해 새로운 파티션을 만든다
- 최대 4개의 파티션을 만들수 있으며 마지막 파티션은 Extended 로 설정해 다시 분할 할 수 있다.
- 첫 섹터를 지정하고 원하는 만큼의 파티션 사이즈를 입력한다. (default 는 전체용량)
![화면 캡처 2022-05-11 111131](https://user-images.githubusercontent.com/57117748/167855422-e57d7551-6c40-4b90-93a4-fbb3fe324f25.png)
![화면 캡처 2022-05-11 111250](https://user-images.githubusercontent.com/57117748/167855431-093caecc-9b7b-46c8-8306-e82ca8ead555.png)

- p 를 통해 생성된 파티션 확인
![화면 캡처 2022-05-11 111323_LI](https://user-images.githubusercontent.com/57117748/167855940-c0dc3289-53bf-4ec3-bb20-53521d230b86.jpg)

- w 를 통해 변경사항 저장 후 종료

- lsblk 를 통한 확인

![화면 캡처 2022-05-11 112950_LI](https://user-images.githubusercontent.com/57117748/167857941-8b0f7a6e-aba2-427f-aebc-f758b88603cd.jpg)



3. mkfs : 파티션에 파일 시스템 설정
- 각 파티션에 파일 시스템 설정
- 파일 시스템 설정 명령
```
mkfs -t ext4(xfs) [파티션 경로]
					or
mkfs.ext4(xfs) [파일경로]
```

 파일 시스템 정보 확인 가능
![ext4](https://user-images.githubusercontent.com/57117748/167859825-08bd66e0-264f-4ac2-a680-bae32e916aa7.jpg)
![xfs](https://user-images.githubusercontent.com/57117748/167860048-b3a7e4fb-9ed5-490d-8497-178c6412a4e4.jpg)



4. mount : 파일 시스템을 지정 디렉토리에 연결하는것
- 파티션의 파일 시스템과 디렉토리를 마운트 해야만 저장 가능
- 마운트 포인트로 지정 할 디렉토리 생성
- 마운트 포인트 : 지정 할 디렉토리 경로

```
# mount 파티션경로 마운트포인트 : 마운트
# umount 마운트포인트 : 마운트 해제
----------------------------------------
# mkdir /test1
# mkdir /test2
# mount /dev/sdb1 /test1
# mount /dev/sdb2 /test2
```

* 마운트는 재부팅시 자동해제 됨으로 /etc/fstab 을 vi editor 를 이용해 설정값을 입력한다.
- /etc/fstab 파일 구조

![/etc/fstab](https://user-images.githubusercontent.com/57117748/167864746-4de459b8-c6b4-4c50-80ab-5c5abf6702af.png)
```
1. # vi /etc/fstab 	: vi 를 이용해 fstab 진입
2. /dev/sdb1	/test1	ext4	default	0	0 : 파일 구조 형식에 맞춰 설정
3. 저장 후 종료 
```
- lsblk -f 로 확인

![화면 캡처 2022-05-11 123307_LI](https://user-images.githubusercontent.com/57117748/167865470-4d129f81-c436-4ef3-836d-642c970cf2f6.jpg)

# LVM (Linux Volume Maneger)
 여러 물리적 디스크를 하나로 연결시켜 하나의 디스크 처럼 만들어 파티션을 분할
- LVM 연관 언어
```
V(Physical Volume) : 실제 하드디스크의 파티션을 의미한다.
G(Volume Group)  : 다수의 PV 를 그룹으로 묶은것.
V(Logical Volume) : VG 를 파티셔닝한 각각의 파티션
PE(Physical Extent) : PV의 블록 단위
LE(Logical Extent) : LV의 블록 단위
```

## LVM 생성과정
1. 기존 파티션들의 파일 시스템의 종류(ID) 변경
- fidisk [장치경로]에서 p 를 통해 system 이 어떤 형태인지 ID를 가지고 있는지 확인. (defaults : 83 Linux)
![화면 캡처 2022-05-11 140808_LI](https://user-images.githubusercontent.com/57117748/167871076-5d32cb13-3e90-4079-a308-652c1031fd49.jpg)

- l 을 통해 사용 가능한 파일 시스템 종류 확인  (82:Linux LVM) 
![화면 캡처 2022-05-11 140821_LI](https://user-images.githubusercontent.com/57117748/167871110-e1e7a363-3435-42fe-95aa-e697fa8928ce.jpg)

- t 를 통해 파티션의 파일 시스템 종류 변경
![화면 캡처 2022-05-11 140920_LI](https://user-images.githubusercontent.com/57117748/167871761-1db7d271-8894-4b14-aac4-48580bc17a25.jpg)

- w 를 통해 저장 종료 후 재부팅 혹은 partprobe 진행 후 재확인
![화면 캡처 2022-05-11 141152_LI](https://user-images.githubusercontent.com/57117748/167872465-a0f4bd32-d01a-45a2-a1cb-654ec4763221.jpg)
![화면 캡처 2022-05-11 141207](https://user-images.githubusercontent.com/57117748/167872591-82e1a13e-171c-49f4-b9c5-8265a6f28f2c.png)

2. PV 생성 
- pvcreate [파티션경로] 	를 통해 PV 를 생성
```
# pvcreate /dev/sdb1
# pvcreate /dev/sdb2
```
![화면 캡처 2022-05-11 141622_LI](https://user-images.githubusercontent.com/57117748/167873159-54b844c7-be66-4a08-8827-31c1a048a9fe.jpg)

- pvscan 을 통해 PV 생성 확인
![화면 캡처 2022-05-11 141725](https://user-images.githubusercontent.com/57117748/167873277-5c3bcd53-dbfb-4ebe-b3df-eb7d9e26543a.png)

3. VG 생성
- vgcreate vg이름 PV1경로 PV2경로	 를 통행 VG 생성
```
# vgcreate newvg /dev/sdb1 /dev/sdb2
```
-vgdisplay -v 를 통해 생성확인
![화면 캡처 2022-05-11 142017_LI](https://user-images.githubusercontent.com/57117748/167875003-96da3fd1-0c7d-4cb3-8c2a-f7cd32439da6.jpg)
4. LV 생성
- lvcreate VG이름 -L or l 파티션사이즈 or 블록수 -n LV이름
- lvscan 으로 확인
```
# lvcreate vgname -L 1G -n newlv
```
![화면 캡처 2022-05-11 142759_LI](https://user-images.githubusercontent.com/57117748/167876288-1193e49b-5285-4f1d-8b75-b02165355696.jpg)
5. LV에 파일 시스템 생성
- mkfs ext4 or xfs 를 통해 LV에 파일 시스템 설정
```
mkfs -t ext4(xfs) /dev/vgname/lvname
					or
mkfs.ext4(xfs) /dev/vgname/lvname
```
![화면 캡처 2022-05-11 143051_LI](https://user-images.githubusercontent.com/57117748/167877569-d8de3726-4b87-429f-8be3-3fff081139dc.jpg)


7. LV 마운트
- mount 를 통해 지정 디렉토리와 마운트
```  
# mkdir /mnt/lvdir
# mount /dev/vgname/lvname /mnt/lvdir
```
![화면 캡처 2022-05-11 143355](https://user-images.githubusercontent.com/57117748/167878471-a8509270-ab0c-4dec-aeae-d4c91bae9e0a.png)

- lsblk -f 를 통해 LV의 파일시스템, 마운트포인트를 확인한다.
![화면 캡처 2022-05-11 143745_LI](https://user-images.githubusercontent.com/57117748/167885587-3cea17ca-d99d-4c65-b7cb-24878c767b89.jpg)

* mount는 항상 재부팅 후 사라지기 때문에 vi 에디터를 사용해 /etc/fstab 수정한다.
# 디스크 관리
# df : 파일 시스템별 디스크 사용량 확인
```
df -Th

파일 시스템의 종류 정보 출력하기 : -T 옵션
파일 시스템 사용량을 이해하기 쉬운 단위로 표시하기 : -h 옵션
```
![화면 캡처 2022-05-11 144710](https://user-images.githubusercontent.com/57117748/167880096-3d289008-6914-4a68-89f9-ca2ab75241ed.png)

## du : 디렉토리 파일들의 디스크 사용량 확인 
```
du -h dirpath
du -sh dirpath : 해당 디엑토리 파일들의 총 사용량만 나온다
```

## fsck 
- 디스크 파일 시스템의 손상 확인
```
fsck path

-f : 강제 실행
- a : 문제 발견시 복구
```

* 만약 수퍼블록이 깨진경우
```
dumpe2fs 장치경로 | grep superblock // superblock 의 백업위치 확인

fsck -b superblock백업위치 -y 장치경로
```

# swap
- 물리 메모리 사용 중 부족하다면 디스크 공간을 할애해 사용하는것.
- 물리 메모리 + swap = 가상 메모리
- swap 메모리가 한계라면 swap 메모리를 추가해 줘야 한다(보통 물리 메모리의 2배)

## paging
* 실행 전 물리 디스크에서 저장하다 실행하면 메모리에 보내고 (pagein)
* 메모리에서 작업을 수행하다 쓰지 않게 된다면 다시 물리 디스크로 보낸다(pageout) 

## swap 절차

1. swap 공간의 확인
	- free 명령어를 통해 swap 공간을 확인한다.
```
# free -h 	// -h 를 통해 사이즈를 보기 편하도록 만든다.
```
![화면 캡처 2022-05-12 001136](https://user-images.githubusercontent.com/57117748/167884764-a77fb4ee-1859-407e-9b90-37ed41416ef9.png)


2. 파티션의 시스템 종류의 변경 (시스템 id 변경)
- p 를 통해 시스템 종류 확인 후 t 를 이용해 변경 (Swap : 82)
![화면 캡처 2022-05-11 154634_LI](https://user-images.githubusercontent.com/57117748/167886105-165e694f-271c-42a1-875f-5424c47606b1.jpg)

3. 각 파티션을 swap 으로 만들기
- # mkswap 파티션경로 를 통해 지정
```
# mkswap /dev/sdb{1...3}
```
![화면 캡처 2022-05-11 155221_LI](https://user-images.githubusercontent.com/57117748/167886635-874199c3-9c49-4ec6-8c01-cd5e974f7759.jpg)
4. 수동 swap 마운트
- 만약 물리 메모리가 한계라면 지정했던 물리 디스크를 swap 해준다.
```
# swapon swappath(파티션명)	// 수동 swap 마운트
# swapoff swappath(파티션명) // 수동 swap 마운트 해제
----------
-a // 전체 적용
```
![화면 캡처 2022-05-11 160324_LI](https://user-images.githubusercontent.com/57117748/167888366-66edef2e-8659-4448-96d0-d4000787138d.jpg)

* 마운트는 항상 vi editor 으로 /etc/fstab 에게 설정을 해줘야 재부팅 후에 초기설정으로 저장된다.
![화면 캡처 2022-05-11 162107_LI](https://user-images.githubusercontent.com/57117748/167888861-147ad478-7e3d-4f99-ba45-b04129c65b93.jpg)

## 키워드
