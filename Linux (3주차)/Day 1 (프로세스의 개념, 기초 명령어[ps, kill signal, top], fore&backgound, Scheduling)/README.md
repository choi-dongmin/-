# 프로세스의 개념 
- 하드 디스크에 있는 파일이 실행되어 메모리에 올라간것.

##프로세스 유형

1. 데몬 프로세스
	- 서비스를 제공하는 프로세스. Background 에서 계속 실행중인 프로세스이다. (주로 실행하면 운영체제에 의해 실행되는 프로세스들)

2. 고아 프로세스
	- 프로세스가 실행중에 부모가 종료되어 없어지게 되어 systemd(최상위 부모 프로세스 : 1번)의 자식 프로세스로 편입됨.

3. 좀비 프로세스
	- 자식 프로세스가 종료되었는데 계속 프로세스 테이블에 남아있는 경우
	- defunct 프로세스라고 읽컫기도 함
	- 좀비 프로세스가 많아지면 테이블의 용량이 부족하여 에러가 발생하기도 한다.
	- 서버의 경우 항상 가동중이기에 좀비 프로세스를 수시로 관리 하여야 한다.

# 프로세스 기초 명령어


## ps -ef (현재 계정의 실행중인 프로세스들을 확인하기)
```
$ ps -ef (Svr4) // -e 모든 프로세스의 정보 -f 자세히 출력
$ ps -aux (BSD)


ps -ef 의 주요 정보 요소 ---------------------------------------

stime : 프로세스의 시작시간
PID : 프로세스의 고유한 아이디
PPID : 부모의 PID

```

![ps -ef](https://user-images.githubusercontent.com/57117748/167371739-2a142d42-bb80-41f0-b218-de7979025cba.png)


- 프로세스 점유율 확인
```
ps -u 

ps -u 의 주요 정보 요소 ---------------------------------------

cpu% : 해당 프로세스가 cpu 를 얼마나 점유하는지 표시
memory% : 해당 프로세스가 메모리를 얼마나 점유하는지 표시
```

![화면 캡처 2022-05-09 173727](https://user-images.githubusercontent.com/57117748/167372368-3f2c3911-89c2-484e-9e9f-becb2b97b3d0.png)

## kill sginal
-시그널 : 메모리에 실행되고 있는 프로세스에 보내는 신호.


- kill Signal List 목록 확인
![kill signal list](https://user-images.githubusercontent.com/57117748/167374242-bcaa1f4b-095b-4f21-a533-28fc02eda12f.png)

```
$ kill -l 		// kill signal list 출력

kill 의 주요 요소 ---------------------------------------
2 : 작업 중단 시그널을 보낸다.
9 : 강제종료
15 : kill 의 defult 값으로 기본종료
```


- kill signal 보내기
```
$ kill 9 PID 	// 해당 PID 를 가지는 프로세스를 강제종료한다.
$ kill PID 		// 해당 PID 를 가지는 프로세스를 기본(15) 종료한다.

```
![kill signal 보내기](https://user-images.githubusercontent.com/57117748/167376469-4f60c2f5-ee33-4d4f-903b-1d342d8b2db8.png)

- pkill : 프로세스 이름을 이용해 종료 (권장되지 않음)

## top
- 윈도우의 작업 관리자와 엇비슷하다.
- 메모리와 cpu 점유율 등을 볼 수 있다.

![화면 캡처 2022-05-09 122014_LI](https://user-images.githubusercontent.com/57117748/167378823-6a074058-9397-4d47-99a2-5f6b11cb85c5.jpg)


```
$ top

top 의 주요 정보 요소 ---------------------------------------

tasks : task의 상태들을 확인 가능 running, sleeping, stopped, zombie 등
%cpu : cpu의 상태를 확인 가능
us : 가용중인 cpu의 용량
id : 가용 할 수 있는 cpu 용량
PID : 각각의 프로세스의 ID 값
PR(Priority) : 우선 순위
NI(Nice) : 친절도 (?)
```

* 프로세스들은 경쟁 , 경합 시스템이다 PR(priority), NI(NICE) 값이 낮을수록 우선순위가 높다.
* pstree -p 프로세스의 관계를 부모, 자식으로 출력한다.

# foreground , background

1. foreground 
	- 프로세스가 실행되면 진행, 결과값이 모니터로 출력되는 것.

2. background 
	- 프로세스가 실행되면 진행, 결과값이 화면이 아닌 뒤에서 실행되는것

## jobs
- 백그라운드에서 실행하는 프로세스를 확인 

![화면 캡처 2022-05-09 124303](https://user-images.githubusercontent.com/57117748/167379934-50e70904-f500-4736-a400-128706a152d0.png)

## 작업전환
- Fore to back or Back to Foregorund 로 변환하는 것을 작업전환이라고 한다.

![작업 전환](https://user-images.githubusercontent.com/57117748/167380851-72aa11a2-f5f6-4acc-84d6-3ea1300f31e6.jpg)

```
$ bg %RunnningID		// fg to bg
$ fg %RunnningID		// bg to fg

fg,bg 의 주요 정보 요소 ---------------------------------------

& : 백그라운드에서 실행
^z : 일시정지
^d : 정상적 종료
^c : 종료
```

## nohup
- 계정에서 로그아웃 한 후에도 백그라운드 작업을 계속해서 진행
- nohup 은 nohup.out 으로 저장된다.

![nohup](https://user-images.githubusercontent.com/57117748/167382193-0c92cfb1-d8a4-4cca-ace8-3cfaebaa4546.jpg)

```
$ nohup command &

ex) $ nohup ls -al &
```

# 작업예약 (Scheduling)
- 지정된 시간에 프로세스 작업을 명령 하는 것

1. at
	- 1회 단발적 프로세스 작업 예약.

```
$ at Time command	// 시간의 지정 형식은 hh:mm yyyy-mm-dd
ex ) $ at 17:00 2022-12-12

at -l 	// at 예약이 된 작업 리스트를 목록으로 출력 
at -d	// at 예약 지우기
```
- 직접 날자와 데이터 위치와 이름 지정
![스케쥴링 at](https://user-images.githubusercontent.com/57117748/167383680-27337636-b61c-4620-b370-27f4de808832.png)

- 지금으로 부터 5분후 실행
![스케쥴링 at](https://user-images.githubusercontent.com/57117748/167385365-7ad9da79-5845-4900-9ff4-d8294a010222.png)
![스케쥴링 at](https://user-images.githubusercontent.com/57117748/167385591-00789196-1264-4696-a259-193fd36db48d.jpg)


2. crontab
	- 정기적인 프로세스 작업 예약.
	- vim Editor 를 이용해 편집한다.

![crontab](https://user-images.githubusercontent.com/57117748/167385763-797cf44f-95fe-4f7d-8fd8-c0c86c75833b.png)

```
- 일(0~59) 시(0~23) 일(1~31) 월(1~12) 요일(0~6) 작업내용 의 순서대로 입력한다.

EDITOR=vi; export EDITOR 		// Linux 에 따라 기본 편집기가 vi 가 아닐수 있음으로 설정

$ crontab -e 		// -e 를 통해 vim 처럼 사용가능
$ crontab -l 		// crontab 을 이용해 예약된 작업을 리스트로 출력
$ crontab -r 		// crontab 을 이용해 예약된 작업을 모두 삭제 (권장되지 않고 -e 에서 dd 를 이용해 라인 지우기가 권장.)
$ crontab -u UserName	// 타계정에 crontab 예약

```

만약 개별 사용자가 아닌 시스템 차원적으로 정기적 예약하고 싶은 작업이 있다면 

-> /etc/crontab 에 작업을 등록한다.


## 연습

1. 문제1
![화면 캡처 2022-05-09 163120](https://user-images.githubusercontent.com/57117748/167386967-0ec1fed4-3adc-404b-be13-a96265053ce2.png)


2. 문제2
-예문

![화면 캡처 2022-05-09 163152](https://user-images.githubusercontent.com/57117748/167387043-2946ab0f-d610-4f78-ae08-fe12c6743b2d.png)

-정답

![화면 캡처 2022-05-09 163341](https://user-images.githubusercontent.com/57117748/167387779-da328741-d1e9-4fba-8062-7f44800d9aaf.png)


3. 문제3
- 예문해석

![화면 캡처 2022-05-09 163453](https://user-images.githubusercontent.com/57117748/167387835-cdd8d417-2a76-455d-9928-7212bb0cb776.png)

-정답
```
매년 1월 첫째주(1-7) 일요일(0) 저녁 12:00 에 시스템을 리부트 하여라

10분마다 1시에서 5시사이에 매일 date 를 datefile01 에 이어써라

3월, 6월, 9월(3,6,9) 두번째주(8-14) 화요일(2) 14:20 에 passwd 파일을 읽어드린 값을 usefile 에 덮어라
```

## at,corntab 엑세스 제어 설정 파일

- 보안을 위해 deny, allow 파일을 사용해 스케쥴링 엑세스를 제어 할 수 있다.
- allow 파일이 없는 경우가 있다 이 경우에는 직접 만들어줘 제한하고 싶은 계정 이름을 입력.

```
1.at
/etc/at.deny
/etc/at.allow


2.crontab
/etc/cron.deny
/etc/cron.allow


case 1 
*.allow 파일은 존재 하지 않고 *.dney 파일만 존재 할 경우 *.dney 파일에 계정이 입력된 사용자는 스케쥴링의 사용이 불가하다.

case 2 
*.allow / *.dney 둘 다 존재 할 경우 *.allow 파일에 계정이 입력된 사용만 가능하다.

case 3
둘 다 존재하지 않는다면 root 만 스케쥴링 명령이 가능하다.

```
## 키워드
프로세스 : 하드디스크의 파일이 실행되어 메모리로 올라간 상태

프로세스 유형 : 프로세스는 3가지 데몬, 고아, 좀비 로 이루어져 있다.

ps -ef : 현재 계정에 실행중인 프로세스를 모두 보여준다. -e : 모든파일, -f : 자세히, -u : 프로세스 점유율 확인

kill : 시스템을 종료시키는 신호를 보내는 명령 2, 9, 15 등의 신호 목록들이 있다.

top : 윈도우OS 의 작업관리자 같은 모드로 진입

pstree : 부모 자식 프로세스의 상관 관계를 보여준다.

foreground, background : 프로세스가 어디에서 실행되는지 나타냄 백그라운드에서 실행하면 화면에 출력되지 않는다.

jobs : 백그라운드에서 진행중인 작업 목록을 확인

작업전환 : 포그라운드에서 백그라운드로 또는 그 반대의 방향으로 진행하던 작업을 전환시키는 것을 뜻하면 fg %RunningID bg %RunningID로 바꾼다.

nohup : 해당 프로세스는 게정이 로그아웃 되더라도 프로세스를 백그라운드에서 끝까지 작업하도록 하는 명령이다.

Scheduling : 작업예약 at, crontab 으로 명령하며 각각 단발성, 지속성을 가진 명령이다.

at : 지정된 시간에 1회의 프로세스 작업을 명령 하는 것. ($ at Time command	// Time = hh:mm yyyy-mm-dd)

crontab : 정기적으로 실행할 프로세스를 예약하는 명령. contab은 vi Editor 를 사용한다 (vim : corntab mm hh dd mm date command)

at,corntab 엑세스 제어 : /etc/at.deny or allow /etc/cron.deny or allow 을 통해 접근을 제어하는 방법 
	1. deny 파일만 있다면 dney 에 등록된 계정은 스케쥴링 불가
	
	2. deny, allow 둘 다 있다면 allow 에 등록된 사용자만 스케쥴링 가능
	
	3. 둘 다 없다면 root 사용자만 스케쥴링 가능  
