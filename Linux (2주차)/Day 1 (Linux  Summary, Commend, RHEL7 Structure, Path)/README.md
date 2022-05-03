# Linux 개요

## Unix 
- 미국의 GE, MIT, AT&T 에서 공동으로 Multics 프로젝트(초기 시분할 운영체제 : 시분할 운영체제 전송매체를 시간 단위로 분할해 전송)를 진행했다.
- AT&T Ken thompson 연구원이  PDP-7에서 Space Travel을 구동하기 위한 운영체제 Unix를 개발한다 .
- Unix 로 인해 수많은 가지가 생기게 되는데 그중 Minix 라는 교육용 운영체제를 본따 Linux x86 환경에서 동작할 수 있는 Linux 개발
가 만들어 진다.
- 자유소프트웨어 재단(FSF)의 도움을 받아 오픈소스로 성장하게 됨.

## Linux Distributed (리눅스 배포판)
- Linux Kernel + 응용소프트웨어 결합하여 배포하는 단위.

1. Red Hat 계열
	- Red Hat
	- RHEL(Red Hat Enterprise Linux)
	- Fedora
	- CentOS
	- RockyLinux

2. Debian 계열
	- Debian
	- Ubuntu
	- Kali

## CLI / GUI
CLI(Command Line Interface)
- 가상 콘솔

GUI(Graphical User Interface)
-그래픽 콘솔

## Unix 운영체제의 핵심 구성요소
![Unix 구성요소](https://user-images.githubusercontent.com/57117748/166209130-f0c9a9dd-6423-467e-af28-3a717495b316.png)

1. H/W : 하드웨어

2. Kernel : 운영체제의 핵심 구성요소로 하드웨어를 직접제어 할 수있는 막강한 권한을 가짐

3. Shell : 사용자의 명령어를 커널이 이해할 수 있는 형태로 전달 즉, Unix/Linux System에서 사용자와 Kernel 사이에 인터페이스를 담당하는 프로그램  

4. Directory : 파일을 포함하는 특수한 파일

## 명령어의 기본 구조

![명령어의 기본구조](https://user-images.githubusercontent.com/57117748/166209137-88e3a2d9-013d-437b-b27c-2f8d72a89adb.png)

- commend [option]... [argument]... 구조로 이루어진다.
- Ex) commend : ls option : -a argument : / 

1. commend : 시스템이 설치되어 있는 프로그램 이름(지정된 위치)
2. option : commend가 실행한 출력값
3. argument : commend를 실행할 때 적용할 대상

## 명령어의 사용방법의 확인
1. --help : 사용 가능한 명령어 확인 
2. man commend : commend 의 정식 메뉴얼 확인

![명령어의 사용방법의 확인](https://user-images.githubusercontent.com/57117748/166209145-30012c52-7464-4e8f-9795-8bf30fe5de6f.png)

### Section 메뉴얼 페이지 
```
Section				설명

(1)					일반 명령어
(2)					System Call
(3)					C Library
(4)					Special File(Device File) 및 드라이버
(5)					File Format
(6)					Game/Screensaver 등
(7)					기타(Misc)
(8)					시스템 관리 명령어 및 데몬
```

- Section 을 지정해 메뉴얼 확인 구조 (몇몇의 페이지는 직접 섹션을 지정해야 가능)

```
# man -s section commend

ex ) # man -s 1 passwd : 섹션 1의 메뉴얼의 메뉴얼을 확인.
ex ) # man -s 5 passwd : 섹션 5의 메뉴얼의 메뉴얼을 확인.
```

- 원하는 메뉴얼 페이지 검색(제목, 설명)

```
# man -k KEYWORD

ex) man -k passwd : 제목과 내용의 중 passwd 를 모두 찾아 출력.  
```
- 원하는 메뉴얼 페이지 검색(제목)

```
# man -f KEYWORD

ex) man -f passwd : 제목 중 passwd 를 모두 찾아 출력. 
```

## 파일 시스템 계층 구조
- Unix / Linux 는 모든것을 파일 형태로 저장한다.

![RHEL7 계층구조](https://user-images.githubusercontent.com/57117748/166211850-68087769-0701-4237-9746-6bd2af34bfbc.png)

1. bin : binary의 약자로 밀반 명령의 디렉토리이다.
2. boot : 부팅에 관련된 디렉토리이다.
3. dev : 시스템의 장치 파일 디렉토리이다. Runtime 데이터 디렉토리이기 때문에 컴퓨터가 종료될때 데이터를 삭제하고 켜질때 데이터를 받는다.
4. etc : 시스템 설정 파일 디렉토리이다.
5. home : 일반사용자의 홈 디렉토리이다.
6. root : root 사용자의 홈 디렉토리이다.
7. tmp : 임시파일이 저장되는 디렉토리이다.
8. usr : 소프트웨어의 파일,패키지 디렉토리이다.
9. sbin : 시스템 관리 명령어 디렉토리이다.
10. var : 시스템의 가변 데이터가 저장되는 디렉토리이다. (web logs..)

계층적이라는 이유는 Windows OS 와는 다르게 항상 최상위 디렉토리는 / 이다. (root)

## 작업 디렉터리와 파일경로의 지정
1. 작업 디렉토리
	- 현재 작업중인 디렉토리를 뜻한다.
	- 명령어 및 스크립트를 사용
	- 로그인 후 기본작업 디렉토리는 사용자의 홈이다.
	- pwd 로 현재위치 확인 가능.

2. 홈 디렉토리
	- 시용자가 파일 생성/삭제 등의 작업을 할 수있는 경로. 

## 절대경로 / 상대경로
1. 절대경로 : root 디렉토리 기준으로 파일 경로를 지정하는 방법 즉, 최상위 디렉토리 부터 시작.
2. 상대경로 : 작업 디렉토리를 기준으로 파일 경로를 지정하는 방법, 즉 현재 위치한 디렉토리 부터 시작.

# Linx 명령어 (commend)
## ls
- 디렉토리내의 파일 확인

```
# ls

# ls -a (all) 숨겨진 파일이나 디렉토리 표시
# ls -l (long) 자세한 내용 출력 (퍼미션, 포함된 파일 수, 소유자, 그룹, 파일 크기, mtime, 파일 이름)
# ls -S (size) 파일 크기 순으로 정렬하여 출력
# ls -r (reverse) 역순(z-a)으로 출력
# ls -R (recursive) 하위 디렉토리까지 출력
# ls -h (human) 파일 크기는 K, M, G 단위를 사용하여 보기 좋게 표시
# ls -c (ctime) mtime 대신에 ctime 출력
# ls -u (atime) mtime 대신에 atime 출력
# ls -i link file 을 위한 i-node 값 출력
```

## pwd
- 현재 위치한 디렉토리 표시

```
# pwd
```

## cd
- Change Directory의 약자로 디렉토리간 이동 할때 쓰는 명령

```
#cd DirectoryName

#cd ../  : 상위폴더로 이동
#cd ../DirectoryName : 상위폴더의 해당 디렉토리로 이동

#cd ./ 현재 폴더
#cd ./DirectoryName : 해당 폴더 디렉토리로 이동

```

## uname 
- 현재에 운영체제 및 커널의 정보를 확인.

```
# uname

# uname -s : kernel 이름을 표시
# uname -r : 현재의 운영체제의 버전 표시
```

## cal
- 달력을 표시 

```
# cal

# cal -1 : 현재의 월 표시
# cal -3 : 현재의 월 앞,뒤 월까지 3개 표시
# cal 23 3 2022 : 지정된 년 월 일 표시
# cal 3 2022 : 지정된 년 월 표시
# cal 2022 : 지정된 년 표시
```

## 한줄에 여러 명령어 입력하기
- 한줄에 복수의 명령을 입력하고 싶다면 세미콜론(;)을 사용해 구분자로 이용해주면 된다.
- COMMAND1 실행 -> COMMAND2 실행...

```
#commend; commend; commend;

# hostname; uname; ls -l; 
```

## 키워드 
