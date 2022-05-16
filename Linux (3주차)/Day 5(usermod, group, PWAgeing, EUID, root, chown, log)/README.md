
# usermod 
- 사용자의 계정 정보 변경 
- useradd 와 비슷한 옵션 대다수 분포

```
usermod 명령어 

-l : 계정의 이름을 변경
-u : UID 를 변경
-g : GID 변경(1차그룹)
-G : 보조그룹 변경 
-aG : 보조그룹 추가
-c : 설명추가
-md : 홈 디렉토리 변경
-s : 기본 shell 변경
```

## usermod 연습

- 문제
![화면 캡처 2022-05-13 103728](https://user-images.githubusercontent.com/57117748/168248587-8c68a9b1-3c61-4b37-b40d-cb237985cfc3.png)

- 풀이
![화면 캡처 2022-05-13 104501](https://user-images.githubusercontent.com/57117748/168248668-e142c6ff-375a-4468-89b6-9094575fc649.png)

- 확인
![화면 캡처 2022-05-13 104717](https://user-images.githubusercontent.com/57117748/168249000-283cb311-50f4-41dd-a59f-4eedf35454c1.png)

## userdel UserName : 유저 삭제 명령어
```
userdel UserName // 유저 삭제 명령어
find / -user [UID] -exec rm -r {} \; // 삭제된 유저의 잔존 파일들을 제거
```
# group

## groupadd
	- 그룹을 생성하는 명령어

```
# groupadd NewName 

# groupadd -g NewGID NewName
```

## groupmod
	- 그룹의 정보를 바꾸는 명령어

```
# groupmod -g GID GroupName 		// 그룹의 GID 바꾸기
# groupmod -n NewName GroupName 	// 그룹의 이름 바꾸기

```

![화면 캡처 2022-05-13 111000](https://user-images.githubusercontent.com/57117748/168251143-a37789e6-587a-45b4-862f-8f96c642e5b0.png)
![화면 캡처 2022-05-13 111312](https://user-images.githubusercontent.com/57117748/168251151-ca32e1b4-3e58-49a0-a3c3-80d38a8f4241.png)

![화면 캡처 2022-05-13 111000](https://user-images.githubusercontent.com/57117748/168250749-7d861b60-5bb5-4856-b522-7a9d22cfa28c.png)

- 심화


![화면 캡처 2022-05-13 112526_LI](https://user-images.githubusercontent.com/57117748/168251571-d05e2e94-9c0f-4393-8b17-4de96f5bed41.jpg)

## groupdel
-	그룹을 지우는 명령어

```
groupdel GroupName
```

# 암호 속성 (패스워드 에이징) 변경
- 계정의 암호 속성으로 /etc/shadow 에 저장된다

## chage

- 정의 암호 속성을 바꾸는 명령어

```
# chage -l UserName		// 현재 계정의 암호 속성을 확인
# chage -m UserName		// 최소 사용 기간 설정을 변경
# chage -M UserName		// 최대 사용 기간 설정을 변경
# chage -W UserName		// 최대 사용기간의 몇일 전부터 경고를 나타낼 건지 변경
# chage -I UserName		// 만료된 계정 암호의 유예 사용기간을 변경
# chage -E UserName		// 계정 암호 만료시키기
```

![화면 캡처 2022-05-13 113822_LI](https://user-images.githubusercontent.com/57117748/168252824-8cbeb371-900d-445b-8a4c-b5a73b22b63a.jpg)

## EUID / RUID

- RUID(UID) : 사용자가 로그인을 진행 한 계정의 UID값
- EUID : 현재의 명령을 실행하고 있는 계정의 UID 값
- Ex 대표적으로 setid 가 설정되어 있는경우 파일을 실행하면 그 파일의 소유자로 작업 실행, 이때 실행 파일 소유자의 UID 가 EUID
- Ex su 명령을 통해 사용자를 전환 한 경우 그 전환된 계정의 UID 는 EUID 이다

EUID / RUID 명령어
```
$ who : 현재 사용중인 계정의 이름
$ w : 현재 사용중인 계정과 그 작업정보를 출력한다
$ last : 사용자 이름과 로그인/아웃 시간, 터미널 번호 및 ip 주소

$ whoami 	//	EUID 출력
$ who am ip	//	RUID 출력
```

![화면 캡처 2022-05-13 115101](https://user-images.githubusercontent.com/57117748/168254961-7365c041-777a-435a-8d9c-166d649f18f2.png)

![화면 캡처 2022-05-13 115135](https://user-images.githubusercontent.com/57117748/168255657-aa08d099-0e6f-405e-9544-ffc29ca35968.png)

![화면 캡처 2022-05-13 115213](https://user-images.githubusercontent.com/57117748/168255679-ab816f49-ea55-47d4-aa49-d86ab8174c64.png)

# root 관리자 권한 사용하기

- su - root : 관리자 계정으로 전환 즉, 모든 권한 부여
- sudo : 관리자의 권한으로 실행 즉, 해당 작업에 대해서만 권한 취득

## sudo 
- sudo 를 사용하려면 두가지의 조건중 최소 하나의 조건을 포함해야 한다
	1. wheel 의 그룹에 포함되어 있어야 한다.
	2. sudoers 라는 파일에 계정이 등록되어 있어야 한다.

![화면 캡처 2022-05-13 121652_LI](https://user-images.githubusercontent.com/57117748/168257636-90b30e69-f0ad-4c06-a908-f6db562446ab.jpg)

* /etc/sudoers
	- 해당 경로의 sudoers 파일을 vi 에디터로 편집해 sudo 를 사용 할 수 있게 만들 수 있다.

- sudoers 추가 순서
```
1. vi 편집기를 이용해 /etc/sudoers 시작
2. 내용 입력 : UserName ALL=(ALL) NOPASSWD:/bin/cat  
	// ALL=(ALL) 어디서나 명령을 실행 할 수 있다.
	// NOPASSWD:/bin/cat : bin/cat 을 실행할 때 NOPASSWD 가 필요없다. 만약 이 부분이 ALL 이라면 패스워드를 묻고 모든기능 사용 
```

![화면 캡처 2022-05-13 123522_LI](https://user-images.githubusercontent.com/57117748/168261308-8b3c4d07-7a42-4dad-b78e-c0adf7649ca8.jpg)

![화면 캡처 2022-05-13 123305_LI](https://user-images.githubusercontent.com/57117748/168261312-232a09b9-21d7-410b-8a4a-390d9fab5a52.jpg)


# passwd 
- 사용자의 계정의 암호를 수정한다

```
# passwd [옵션] [계정이름]
```

- passwd 명령어
```
-l : 지정한 계정의 암호 잠금
-u : 지정한 계정의 암호 잠금해제
-d : 지정한 계정의 암호를 삭제
```

# chown
- 파일 및 디렉토리의 소유자 변경
- $ chown [수정 사용자계정]:[수정 그룹명] [파일명]
```
$ chown root 1234	// 1234 파일 소유자를 root 로 변경 
$ chown :root 1234	// 1234 파일 그룹을 root 로 변경 
$ chown root:root 1234	// 1234 파일 소유자와 그룹을 root 로 변경 
```

![화면 캡처 2022-05-13 124350_LI](https://user-images.githubusercontent.com/57117748/168263293-f8eabf3a-8ecb-4c6a-bb68-2ad11a883865.jpg)

* 그러나 해당 디렉토리의 소유자를 바꾼다고 해도 그 디렉토리 내의 파일 및 폴더는 변하지 않는다 이것을 해결을 하고 싶다면 "$ chwon -R [디렉토리이름]"  을 쓴다.


# log 관리
- 로그는 시스템에서 일어난 이벤트들을 기록한 것이다. 그 로그를 기록하는 행위를 logging 이라고 일컫는다.

- 로그의 역할
	1. 문제의 원인을 판단 할 수 있는 정보가 된다.
	2. 인가되지 않은 접근을 조회 할 수있다. (systemd-journal 을 통해 모든 로그를 수집)

![화면 캡처 2022-05-13 192653](https://user-images.githubusercontent.com/57117748/168265207-4ed62745-2a6b-47a0-812c-096537ce3945.png)


## 로그 관리 데몬
- systemd 시스템에서 로그는 "rsyslogd" 와 "system-journald" 두 데몬에 의해서 관리된다.

- 로그 관리 데몬흐름도 

![화면 캡처 2022-05-13 202001](https://user-images.githubusercontent.com/57117748/168272969-8e23a3ad-b71e-4ffc-86c4-df384a4a7510.png)


- 로그의 흐름도
```
syslog message -> systemd-journald -> /run/log/journal
				   -> rsyslogd -> /var/log									 
```
![화면 캡처 2022-05-13 202207](https://user-images.githubusercontent.com/57117748/168273264-f944d6a1-bf83-472a-a0d3-a6eecceb0342.png)


1. rsyslogd : 로그를 기록하기 위한 표준 프로토콜 syslog 를 저장하는 프로세스 
	- syslog 를 전달해서 각 파일 별로 로그를 저장한다.
	- 설정 파일 : /etc/rsyslog.conf
	- 데이터 확보를 위해 로그파일을 4주간 저장후 삭제
	- 규칙
```
Facility.Priority		Action 				
--------------------------------------------------------------
Facility : 리눅스에서 이벤트 발생시 분류기준
1. auth : 인증관련 서비스
2. cron : 예약작업의 서비스 (cron, at)
3. kern : 커널이 발생한 서비스
4. deamon : 데몬프로세스 관련 서비스
5. syslog : syslog 관련 서비스
6. user : 사용자 프로세스
7. * : 모든 서비스


Priority Level : 이벤트의 중요성					오름차순
1. none : 메세지 기록하지 않음						 
2. debug : 디버깅 관련 메세지
3. info : 단순한 프로그램 정보 및 통계관련 메세지
4. notice : 에러가 아닌 알람에 관한 메세지
5. warning : 주의 메세지
6. err : 에러 발생 메세지
7. crit : 급한 상황은 아니나 치명적 문제 발생의 메세지
8. alert : 즉각적 조치 필요 상황
9. emerg : 전체 공지가 요구되는 최상위 위험상황
10 * : 전체 메세지 저장 
```
![화면 캡처 2022-05-13 214425_LI](https://user-images.githubusercontent.com/57117748/168286146-435e5ec7-b009-4196-a6dc-1c276fc3303c.jpg)


	- 로그파일을 종류
	1. /var/log/message	: 대부분의 이벤트를 기록
	2. /var/log/secure : 인증에 관련된 로그 기록
	3. /var/log/cron : 주기적인 작업과 관련된 로그 기록 
	4. /var/log/boot : 부팅 과장에서 발생된 로그 기록
	
![화면 캡처 2022-05-13 142334](https://user-images.githubusercontent.com/57117748/168274733-49152678-23ca-465c-8d3f-37ee2ab8a88d.png)


2. system-journald : 부팅을 시작하면서부터 모든 로그를 수집한다.
	- system-journald에 의해서 처리되는 로그 기록으로 바이너리 형태의 파일로 저장
	- /run/log/journal
	- /run 디렉토리는 메모리기반 파일 스스템 tmpfs 로 마운트 되어 재부팅되면 데이터파일 삭제 (휘발성 데이터)
```
# journalctl

journalctl 명령어 -----------------------------------------------------
-r : 역순 조회 (최근부터)
-p Priority : Priority 조회
```

* system-journald 의 영구저장
```
1. NewDirctory	// 저장할 디렉토리 생성
2. chown root:systemd-journal NewDirctory // 디렉토리 소유자와 그룹 설정
3. chmod g+s NewDirctory // 디렉토리의 권한 설정
4. systemctl restartd-journald // restartd-journald 재시작
```

- 영구저장 조건 2가지중 하나라도 포함하면 저장하지 않음
	1. 현재 파일 시스템 전페 사이즈의 10% 를 초과하면 안한다.
	2. 현재 파일의 여유공간중 15% 를 초과하면 안한다.


## 키워드
- usermod : 사용자의 계정 정보를 수정하는 명령어
- userdel : 사용자의 계정을 지우는 명령어
- groupadd : 새로운 그룹을 만드는 명령어이다.
- groupmod : 그룹의 정보를 변경하는 명령어
- groupdel : 그룹을 지우는 명령어
- password ageing : 암호의 속성, 설정(min,max,warning,inactive,expire)을 바꾸는 명령어 
- EUID / RUID : EUID 는 현재 작업하는 계정의 UID / RUID 부팅 후 로그인한 계정의 UID
- su : 계정 전환 및 관리자 계정으로 전환
- sudo : 관리자의 권한으로 실행하는 명령이다. sudo 를 사용하기 위해선 wheel group에 속하여 있거나 혹은 etc/sudoers 파일에 형식에 맞게 저장되어 있어야 한다.
- passwd : 사용자 암호를 지정하는 명령어
- chown : 파일 및 디렉토리의 소유자와 그룹을 바꾸는 명령어
- log : 시스템에서 일어난 이벤트를 기록하는 것이고 이를 logging 이라고 일컫는다. 로그를 통해 문제의 원인을 판단 혹은 인가되지 않은 접근을 조회 할 수 있다.
- journal : 시스탬에서 발생한 모든 이벤트를 기록한다 그러나 휘발성 데이터 임으로 부팅을 하면 사라지고 이를 위해 디렉토리와 연결 시켜 저장하기도 한다.
- syslog : rsyslog.conf 에서 주어진 속성에 맞는 이벤트 들을 저장한다.
- Facility : 이벤트 발생 시 분류가 되는 기준으로 secure,message,kern,cron,deamon,syslog,user,* 이 있다.
- Priority : 이벤트의 중요성 정도로 debug,info,notice,warning,err,crit,alert,emerg 순서로 중요성을 나타낸다.
- system-journald : 부팅이 시작되면서 시작되는 모든 이벤트를 기록하며 그 기록은 bin 형태의 휘발성 데이터로 저장된다

