# 정규표현식
- 특정한 규칙을 가지는 문자열을 정의하기 위한 표현방식.

## 정규표현식의 메타문자
1. ^ (Carrot) : 행의 시작지점
2.$ (Dollar): 행의 마지막 지점
3.. (Dot): 임의 문자 1개 매칭 
4.* (Asterisk): 0 ~ 무한대의 갯수 임의 숫자 매칭
5.[] (Bracket) : 패턴 내에서 1개의 문자 매칭 
6.[^] : 패턴을 제어한 1개의 문자를 매칭

```
정규 표현식을 이용한 파일 내용 검색
 패턴검색
  $ grep 'root' /etc/passwd

 root를 포함하는 행 검색 후 행번호 출력
   $ grep -n 'root' /etc/passwd

 root를 포함하지 않은 행을 검색
   $ grep -v 'root' /etc/passwd

 root가 매치되는 패턴 개수 구하기
   $ grep -c 'root' /etc/group

 행의 처음에서 검색
  $ grep '^root' /etc/passwd

 nologin으로 끝나는 행 검색
  $ grep 'nologin$' /etc/passwd

 p로 시작하고 x로 끝나는 7자리 단어가 포함된 행 검색
   $ grep 'p.....x' /etc/passwd

 bash 패턴을 단어 단위로 검색
   $ grep -w 'bash' /etc/passwd
```

-정규표현식 사용 예제

```
  $ grep 'root' /etc/passwd

  $ grep '^root' /etc/passwd

  $ grep 'nologin$' /etc/passwd

  $ grep -w 'a..' /etc/passwd

  $ grep -w 'g..' /etc/passwd

  $ grep 'o*t' /etc/passwd

  $ grep 'a[abcde][a-z]' /etc/passwd

  $ grep 'a[^a-d][a-z]' /etc/passwd

  $ grep 'a[^a-l][a-z]' /etc/passwd

```
![화면 캡처 2022-05-04 115747](https://user-images.githubusercontent.com/57117748/166700973-031f6d85-e830-4372-bc51-5efc89a32de1.png)

# vi Editor

## vim edior 
- 리눅스의 기본 편집기
- vi 의 상향버전
- 텍스트 기반의 동작 방식
- 4가지의 동작 모드를 지원, vi는 3개 (Edit, Command, Extended, Visual : 사실상 코어는 E,C,E)

1. Edit
	-텍스트를 편집을 하는 모드로 [a],[i],[o] 를 통해 Command 에서 진입 가능하다.
	- 다시 Command로의 진입은 [ESC]

2. Command
	- Defualt 모드로 처음 실행시 기본 모드이다.
	- 명령을 실행 하는데에 사용(복사, 삭제, 이동)

3. Extended
	- 파일의 저장, 종료를 지원하고 다른쪽으로는 검색, 치환 등을 지원한다.
	- Extended로의 진입은 [:]
	- 다시 Command로의 진입은 [ESC]

## vim 단축키 정리

1.편집 기본 및 수정 
```
i : 현재 커서가 위치한 곳에서 부터 에디팅 시작.
R : REPLACE 모드 (키보드의 Insert 와 같은)


r : 문자 하나만 수정
S : 라인 자체를 삭제하면서 edit mode 돌입
s : 한 단어 삭제 후 edit mode 돌입
C : 커서부터 라인 끝까지 삭제 후 edit mode 돌입
cw : 한 단어 삭제하고 edit mode 돌입
```

2. 커서 이동 (command)
```
h : 왼쪽 이동
j : 아래 이동
k : 위 이동
l : 오른쪽 이동 라인 뒷방향으로 이동
gg 문서의 처음

a : 커서 뒤
o : 커서 아래 라인에
I : 라인에 앞
A : 라인에 뒤
O : 커서 위에
```

3. 삭제 (command)
```
dd : 라인 삭제 (잘라내기)
dw : 단어 삭제 (잘라내기)
```

4. 재실행 (command)
```
실행취소/재실행
U : 라인에 수정한 것으로 원복하고 싶을때
u : 작업 취소
[Ctrl]+r : 작업 재실행
```

5. 복사 및 붙여넣기 (command)
```
yy : 라인 복사
yw : 단어 복사
y^ : 커서 앞에 부터 라인 앞까지 복사
y$ : 커서 포함 라인 뒤까지 복사

P : 라인 복사시에는 커서의 윗라인에 붙여넣기 , 라인이 바뀌지 않는 복사시에 커서의 앞에 붙여넣기
p : 라인 복사시에는 커서의 아래에 붙여넣기, 커서의 뒤에 붙여넣기
```

6.저장하기 (extended)
```
:q  = 수정을 안했을시에 종료
:q! = 수정을 저장을 안하고 종료
:w  = 저장하기 
:x  = 저장후 종료
:x PATH/file명 = 다른이름으로 저장하기
```

7. 검색
```
/string = string 검색 (내림차순)
n, N = 다음, 이전 검색
```

8. 치환
```
:%s/A/B = 각 행의 첫 A를 B로 바꾸기
:%s/A/B/g = 문서 전체에서 A를 B로 바꾸기
:SP,EP s/A/B/g = SP 라인부터 EP라인까지 A를 B로 바꾸기
:1,5 s/A/B/g   1번행 부터 5번행까지의 범위에서 A를 B로 찾아 바꾸기
```

![:%s/A/B](https://user-images.githubusercontent.com/57117748/166706302-136a3589-9da5-40cf-85cb-10c27414daf8.jpg)
![:SP,EP s/A/B/g ](https://user-images.githubusercontent.com/57117748/166707110-2d3818f0-19ef-4efc-ab83-574a28f130ff.jpg)
![1,5 s/A/B/g](https://user-images.githubusercontent.com/57117748/166707307-284cc0cf-f478-4a78-b866-3977c27609d7.jpg)






# 파일 입출력
-Input / Output
	1. 표준입력 (0)
		- 채널이름 : stdin
		- 입력도구 : 키보드 

	2. 표준출력 (1)
		- 채널이름 : stdout
		- 출력도구 : 모니터

	1. 표준에러 (2)
		- 채널이름 : stderr
		- 출력도구 : 모니터

## 리다이렉션
- 프로세스의 입력 혹은 출력의 대상을 변경한다.
- 일반적으로 출력의 값을 파일로 저장한는 용도.
- 표준 입출력 및 표준 에러에 대한 처리가능

```
1>  : 표준출력 재지정 (덮어쓰기)
1>> : 표준출력 재지정 (이어쓰기)
2>  : 표준에러 재지정 (덮어쓰기)
2>> : 표준에러 재지정 (이어쓰기)   
&> 	: 표준출력&에러 재지정 (덮어쓰기) 
&>> : 표준출력&에러 재지정 (이어쓰기)   
```
![리다이렉션](https://user-images.githubusercontent.com/57117748/166707893-b6027e26-c6b4-42c2-8bb2-3ae227684548.jpg)
![리다이렉션](https://user-images.githubusercontent.com/57117748/166709279-051b6335-95b1-4d23-afd4-f76f0fd401a1.png)

# 파이프(Pipe)와 티(tee)

## Pipe (|)
- 선행 프로세스의 실행 결과를 표준출력이 아닌 후속 프로세스의 입력으로 전달
![Pipe 구조](https://user-images.githubusercontent.com/57117748/166710881-d96bcd76-1a5f-4ce1-8251-375ea01327b8.png)
![Pipe](https://user-images.githubusercontent.com/57117748/166711535-8c6ff0cd-1cfe-482c-999c-dd706658c42e.jpg)
![Pipe](https://user-images.githubusercontent.com/57117748/166712080-ee8a2a28-0578-4ce9-badb-56230d1fe7f2.png)

## tee
- 선행 프로세스의 실행 결과를 표준출력하고 또 후속 프로세스에게 전달
- 리다이렉션과 파이프 기능을 동시에 사용
![tee](https://user-images.githubusercontent.com/57117748/166712395-7d790af0-a391-44b3-8f61-b613c867ba3a.png)

# 아카이브 및 압축
## 아카이브
- 여러 파일을 하나의 파일로 묶어서 보관하는 방법
- 백업 및 복제 용이

1. tar
- 아카이브 파일 관리에 있어 오래된, 일반적인 명령
- 읽기 권한이 있어야 아카이브 가능
- 아카이브를 완료하면 사이즈가 더 커진다.

```
$ tar cf A.tar fileA.txt fileB.txt... 
$ tar cvf A.tar fileA.txt fileB.txt... 

fileA, fileB... 를 fileA.tar에 하나로 합친다.


$ tar xf A.tar
$ tar xvf A.tar 

fileA.tar에 있는 파일을 펼친다.


$ tar tf A.tar 
fileA.tar에 있는 파일목록 확인


option
-c(create) 새로운 묶음을 생성
-x(extract) 묶인 파일을 풀어줌
-t(list) 묶음을 풀기 전에 목록을 보여줌
-f(file) 묶음 파일명을 지정해줌
-v(visual) 파일이 묶이거나 풀리는 과정을 보여줌
```

2. 압축
- 파일 사이즈 크기를 줄이는 것

1. gzip z 	FileName.tar.gz 	가장 오래되고 속도가 빠름
2. bzip2 j 	FileName.tar.bz2 	고용량에 압축률이 gzip보다 좋음
3. xz J 	FileName.tar.xz 	가장 최근에 만들어 짐, 압축률이 좋음

```
1. compress
Unix 기반의 LZW 방식의 압축 도구로 압축 강도가 낮은 압축 방식

$ compress FILENAME
$ uncompress FILENAME


2.gzip
GNU zip의 준말로, 초기 유닉스 시스템에 쓰이던 압축 프로그램을 대체하기 위한
자유 소프트웨어

$ gzip FILENAME
$ gunzip FILENAME


3.bzip
줄리안 시워드가 개발한 오픈소스 압축 알고리즘 및 압축 소프트웨어로 gzip보다
압축율이 더 높은 방식

$ bzip2 FILENAME
$ bunzip2 FILENAME


4.xz
LZMA 방식의 압축 도구로 가장 최근에 나온 압축 방식으로 압축 효율이 다른 압축
방식에 비해 우수함

$ xz FILENAME
$ unxz FILENAME
```

* 위 두개의 것이 합쳐진것이 현재 우리가 사용하는 압축방식
```
아카이브와 압축 동시에 하기

$ tar cfz ArchiveName.tar.gz FileA.txt FileB.txt...		// FileA.txt FileB.txt... 를 아카이브화 후 gzip 으로 압축
$ tar cfj ArchiveName.tar.bzip2 FileA.txt FileB.txt...		// FileA.txt FileB.txt... 를 아카이브화 후 bzip 으로 압축
$ tar cfJ ArchiveName.tar.xz FileA.txt FileB.txt...		// FileA.txt FileB.txt... 를 아카이브화 후 xz 으로 압축

$ tar xfz ArchiveName.tar.gz	// ArchiveName.tar.gz 를  gzip 으로 압축해체
$ tar xfj ArchiveName.tar.bz 	// ArchiveName.tar.bzip2 를  bzip 으로 압축해체
$ tar xfJ ArchiveName.tar.xz	// ArchiveName.tar.gz 를  xz 로 압축해체
```

# 프로세스 제어
## 프로그램 -
- 소스코드가 빌드되어 실행가으한 실행 바이너리 파일로 주로 보조기억장치에 저장

## 프로세스 -
- 프로그램이 메모리에 로드되어 CPU 를 통해 실행되는 프로그램을 의미 프로세스는 프로그래과 달리 "실행 상태"를 가짐
	1. 부모 프로세스 : 자식 프로세스에게 파일을 복제하여 작업을 처리시키고 아웃풋을 받는다.
	2. 자식 프로세스 : 부모 프로세스에 의해 생성되어 작업 완료 후 자원해제

![화면 캡처 2022-05-05 003525](https://user-images.githubusercontent.com/57117748/166717696-6dcffc5a-8dcf-4066-912f-deae2561a527.png)

## 데몬
- 백그라운뎅서 동작하는 프로세스

## 쓰레드
- 프로세스 내에서 싱행되는 작업의 단위


## 작업 환경 foreground / background

- 포그라운드 : 동작 시에 화면 ( 터미널 ) 에 나타나는 프로세스
- 백그라운드 : 동작은 하지만 터미널을 따로 사용 가능

```
$ sleep 100& 	// 100 초동안 sleep 상태 & 를 붙이면 background 에서 프로세스 실행
& jobs 			// 현재 실행중인 작업에 대해 출력
&fg %N			// N번의 processID 를 가진(job에서 확인이 가능) background process 를 foreground 로 옮긴다.

```

![foreground / background](https://user-images.githubusercontent.com/57117748/166718141-101a3c10-50ec-4d64-a184-7557405381fe.png)

## 프로세스 상태 확인
1. $ ps 
	- 해당 시점에 터미널에서 실행되는 목록을 표시
	- -ep : 시스템 전체에 실행되는 프로세스 표시

2. $ top
	- windowsOS 의 작업관리자와 같이 현 프로세스의 전반적인 상태를 표시

3. $ uptime
	- OS 프로세스 실행 후 얼만큼의 시간이 흘렀는지 나타냄

* 부하평균 
	- 목적은 시스템 업무의 부하를 관리
	- 부하 평균갑을 통해 리소스 확인
	- 1분, 5분, 15분에 대한 값을 표시
	* 부하 평균값이 1을 넘으면 프로세스가 동작하면서 리소스 할당을 받기 위한 대기시간이 생긴다.

## 키워드
