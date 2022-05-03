# Linux 파일 및 디렉토리 관리
- 디렉토리란 파일을 포함 할 수있는 특수한 파일 (Ex 폴더)
- 파일 데이터를 가지고 있는 객체로 저장장치에 저장된 객체

## ls -l 의 구조

- ls 디렉토리 파일 목록 확인

![ls -l 의 구조](https://user-images.githubusercontent.com/57117748/166424526-f447e645-24c4-457c-a0dd-ab4e15ec6a34.jpg)

* 왼쪽부터 
	1. 파일의 정보 및 퍼미션
	2. Hard Link 의 수
	3. 소유자의 정보
	4. 소유자의 그룹
	5. 파일의 크기 
	6. 마지막 수정 날짜(Last Modified Type)
	7. 파일의 이름

## 파일 내용 확인 명령어

1. cat 
	- txt 파일의 내용을 확인하는 명령어.

```
$ cat FileName
```

![Command cat](https://user-images.githubusercontent.com/57117748/166424912-3a78bd85-02ea-40c0-8b99-b83378874362.jpg)

2-1. more
	- txt 파일의 내용을 페이지 단위로 출력
```
 $ more FileName
```

![Command more](https://user-images.githubusercontent.com/57117748/166424157-164e47f2-772b-48cf-8903-316ac73c3184.png)

2-2. less 
	- txt 파일의 내용을 페이지 단위로 출력, more 과 차이점은 less 는 마지막 페이지를 확인 후 자동으로 Shell 로 이동
```
$ less FileName
```

3-1. head 
	- 파일을 가장 앞부분을 n만큼 출력한다 (n의 default 값은 10)

```
$ head FileName
$ head -n 5 FileName
```
![Command head](https://user-images.githubusercontent.com/57117748/166426616-14d451fa-e5b9-4b53-ac31-2913a8ae6211.png)

3-2. tail
	- 파일을 가장 뒷부분을 n만큼 출력한다 (n의 default 값은 10)

![Command tail](https://user-images.githubusercontent.com/57117748/166426764-8207dd61-d088-444f-8fda-2af57a30d39e.png)

## 파일의 생성 및 삭제, 옮기기

1-1. mkdir
	- 디렉토리를 만드는 명령어

```
$ mkdir NewFileName
```

![Command mkdir](https://user-images.githubusercontent.com/57117748/166427694-f5430ceb-a7c4-46d6-ae8b-58deb701ccd8.png)

1-2. rmdir
	- 디렉토리를 제거하는 명령어

```
$ rmdir FileName
$ rmdir -f FileName : root 모드에서 파일 삭제시 확인절차 생략.
```

![Command rmdir](https://user-images.githubusercontent.com/57117748/166427696-4bdcdad9-3fe3-4882-9a65-5b3fa9c4ee1c.png)

2-1. toudch
	- bin 파일을 생성하는 명령어

```
$ touch NewFileName
```

![Command touch](https://user-images.githubusercontent.com/57117748/166428184-6e06e56d-bbc8-44e9-8f88-3cad7375dc7e.png)

2-2. rm
	- 파일을 제거하는 명령어
	- root 모드에서는 제거 명령을 다시한번 확인한다.

```
$ rm NewFileName
$ rm -f FileName : root 모드에서 파일 삭제시 확인절차 생략.
```

![Command rm](https://user-images.githubusercontent.com/57117748/166428187-0bb25cb9-07db-4f37-b905-ea4090a71945.png)

3. cp
	- 원본과 데이터가 같은 사본 파일을 만드는 명령어

```
$ cp OriginalFileName ./NewFileName or DirPath/NewFileName
$ cp -r OriginalDirName DirPath/NewDirName : 폴더를 통쨰로 복사
```

![Command cp](https://user-images.githubusercontent.com/57117748/166430143-18f890b0-0497-40d8-bc00-e1a20e190318.png)

4. mv
	- 파일을 다른 경로로 이동시키는 명령어

```
$ mv FileName or DirName Path
```

![Command mv](https://user-images.githubusercontent.com/57117748/166431138-1dbb9f9d-7870-408d-ab08-ce4c2b37704f.png)

## 다중 명령

- 파일의 생성 및 삭제 등의 명령어는 다중 명령이 가능하다.

```
$ mkdir NewFileName1 NewFileName2 NewFileName3...
$ rmdir NewFileName1 NewFileName2 NewFileName3... 
$ touch NewFileName1 NewFileName2 NewFileName3... 
$ rm NewFileName1 NewFileName2 NewFileName3... 
$ mv NewFileName1 NewFileName2 NewFileName3... Path 
```

![다중 명령](https://user-images.githubusercontent.com/57117748/166431966-8e0a486a-3307-488d-9909-cca7f4518828.png)

## 파일 및 디렉토리 검색

1. locate 검색
	- 데이터 베이스 내에서 검색
	- 검색 속도가 빠름
	- 파일 시스템 구조 변경시 검색 불가
	- 파일 이름으로만 검색 가능
	- 최상위 경로로 부터 검색하고자 하는 조건을 갖춘 모든 파일 출력 

```
$ locate FileName
```
![Command locate](https://user-images.githubusercontent.com/57117748/166435699-76fa0e4e-0db0-4b04-857b-009dfcd9cc25.png)

2. find
	- 직접 접근하여 검색
	- 검색 시간이 오래 걸림
	- 여러 조건으로 검색이 가능
	- 검색과 동시에 추가 작업 가능
	- 최상위 경로로 부터 검색하고자 하는 조건을 갖춘 모든 파일 출력 

```
$ find Path Expression [Action]  : 경로, 조건, 실행 후 추가 작업

Expression
1. -name : 이름
2. -type : 파일 종류
3. -perm : 파일 권한
4. -user : 파일 소유자
5. -size[+|-] : 파일 사이즈

Action
1. -print : 검색 후 출력
2. -ls : 검색 후 자세한 정보 확인
3. -exec {Command} : 검색 후 파일에 대한 특정 명령 실행
3. -ok {Command} : 검색 후 파일에 대한 특정 명령 실행, 확인 절차 추가
```

![Command find](https://user-images.githubusercontent.com/57117748/166436819-5c723d5e-9c09-4c6d-98ef-7426bd595ad7.png)

# Link File

## 링크 파일이란?
	- 하드와 심볼릭(소프트 링크)로 구분된다.
	- 파일은 데이터에 접근할 때 i-node 테이블울 통해 접근한다.
	- i-node : 파일의 정보를 담은 table
	- 디스크 공간을 공유하는 개념으로 효율적 관리가 가능하다.
	- 파일은 하드링크 1개 폴더는 2개.

1. Hard Link File
	- 파일이 물리적으로 저장공간에 저장된 주소를 가르키는 링크
	- 원본 파일과 i-node 가 동일하다.
	- 하드링크는 원본파일과 구분 할 수가 없다.
	- 원본 파일을 삭제해도 하드링크를 통해 접근이 가능
	- 하드 링크 변환시 원본도 변환된다 그 반대도 마찬가지.
	- 가르킬수 있는 대상 : 파일

```
$ ln OriginalFileName HardLinkFileName

```

![Hard Link File](https://user-images.githubusercontent.com/57117748/166433732-4eca0996-f336-4bb0-9ebf-142f102cf17e.png)

2. Symbolic Link File
	- 파일이 저장된 논리적인 경로를 가르키는 링크
	- 그렇기 때문에 복사본과 달리 저장공간을 차지 하지 않는다.
	- 원본 파일과 i-node 가 다르다.
	- 그렇기 때문에 심볼릭링크는 원본과 링크파일을 구별할 수 있음.
	- 원본 삭제시 링크파일로 접근 불가
	- 소프트 링크 변환시 원본도 변환된다 그 반대도 마찬가지.

```
$ ln -s OriginalFileName HardLinkFileName
```

![Symbolic Link File](https://user-images.githubusercontent.com/57117748/166434100-fcf5026e-d719-467a-b76c-e3398f52cb6d.png)


* 즉 하드링크는 같은 물리적 주소를 공유하기 때문에 같은 i-node 를 가지고, 소프트 링크는 논리적 주소(경로)를 가르키기 때문에 다른 i-node 를 가진다.

hard link file 은 집 주인의 아들
copy file 은 집 주인 제자
soft link file 은 집 주인 객식


# 텍스트 편집기 (vi)

## vimm edior 
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

## 키워드 