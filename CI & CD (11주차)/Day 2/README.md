# git Commands

## 파일삭제
- git 저장소의 파일을 삭제
- 삭제한 후에 add, commit을 거쳐야 버전 확정됨
```
$ rm [File]
$ git add .
$ git commit -m [MESSAGE]
```

## 파일 이름 변경
- git 저장소의 파일의 이름을 변경
- 수정한 후에 add, commit을 거쳐야 버전 확정됨
```
$ git mv [SRC] [DEST]

$ git mv a abc
$ git add .
$ git commit -m 'Rename a to abc'
[master a2adfc8] Rename a to abc
 1 file changed, 0 insertions(+), 0 deletions(-)
 rename a => abc (100%)
```

# Git 변경사항 확인하기

## Git diff
- 이전에 commit과 현재c(ommited 상태 전)을 `git diff`로 비교할 수 있다.
- a > ab로 치환
```
$ echo ab > a
$ git diff
diff --git a/a b/a
index e61ef7b..81bf396 100644
--- a/a
+++ b/a
@@ -1 +1 @@
-aa
+ab
```

- staged로 진행 후 비교 `git diff --staged`
```
$ git add .
$ git diff --staged
diff --git a/a b/a
index 7898192..81bf396 100644
--- a/a
+++ b/a
@@ -1 +1 @@
-a
+ab
```

## Git log -p
- Commit을 마친후에 이전 버전과 어떤 차이가 생겼는지 확인 가능 
- `git log -p` 를 활용 한다면 어떠한 변화가 있었는지 +,-로 나타 난다
- alpha/a ---  
- beta/a +++
- -a / +aa

```
$ git log -p
commit 640a0513f76c5b8a2bacd629adf7d9e962d55b8b (HEAD -> master)
Author: DongMin Choi <cdm5212@naver.com>
Date:   Tue Jul 5 23:32:59 2022 +0900

    new a2

diff --git a/a b/a
index 7898192..e61ef7b 100644
--- a/a
+++ b/a
@@ -1 +1 @@
-a
+aa
``` 

# Skip Staged
- 버전 확정 바로 전 단계인 Staged를 생략하고 싶다면 `git commit -a -m [Message]` 를 사용
	1. 추적되고 있는 파일에만 적용
	2. 위험성이 있어 잘 사용하지 않는 코드
```
$ git commit -a -m [MESSAGE] 
```

```
$echo abc > a
$ git commit -a -m 'modified a to abc' 	// add 하지않고 -a 옵션과 commit 같이 사용
[master 88fda49] modified a to abc
 1 file changed, 1 insertion(+), 1 deletion(-)
```

## Commit history 
- 기본적으로 Commit의 history 즉, 버전의 확인은 `git log`와 다양한 옵션의 조합으로 확인 가능하다.

1. 정보 옵션
```
--oneline	// 한 라인에 정보를 표시
--graph		// 관계도를 그림 형식으로 가시성을 높혀줌
-2 			// 최신 2개의 버전 
-p 			// 해당 버전 중 수정 내용 확인
--stat 		// 해당 커밋으 수정 파일 통계
```

2. 조회 제한조건
```
--since 	// 명시한 날짜 이후의 커밋만 검색한다.
----until 	// 명시한 날짜 이전의 커밋만 검색한다.
--grep		// 커밋 메시지 안의 텍스트를 검색한다.
```

* 이외에도 많은 옵션이 있으니 [Pro Git Book](https://git-scm.com/book/ko/v2/Git%EC%9D%98-%EA%B8%B0%EC%B4%88-%EC%BB%A4%EB%B0%8B-%ED%9E%88%EC%8A%A4%ED%86%A0%EB%A6%AC-%EC%A1%B0%ED%9A%8C%ED%95%98%EA%B8%B0) 참고


# Commit 되돌리기

## amend
- stage 혹은 commit 된 상태로 되돌리기
- `--amend` 명령어 사용
- 즉, 최근의 버전을 수정, 메세지 변경, 의미있는 파일 누락 등 필요할때 사용
- commit은 되었지만 저장소에 올리기 전 즉, push 하기 전에 가능
```
$ git commit --amend
```

- Rename을 Renamed 로 변경
```
Renamed a to abc 	 

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Tue Jul 5 23:55:32 2022 +0900
#
# On branch master
# Changes to be committed:
#       renamed:    a -> abc
#
```

- 확인
```
$ git log --oneline
b763bf9 (HEAD -> master) Renamed a to abc
```

- 직전 커밋 파일 추가
```
$ git commit -m 'initial commit'
$ git add forgotten_file
$ git commit --amend
```

## resest  
- 현재 committed 상태를 과거의 버전으로 되돌리기.
- 다른 버전으로 commit 하는 방법이 아닌 과거의 커밋으로 다시 전개.
```
! 그러나 이 방법은 후에 다른 사람들과 협업(Push, Pull)을 할때 문제를 발생 시켜 좋은 방법이 아니다.

예를들어 A,B가 같은 저장소의 V3을 공유하다 A가 reset V2로 되돌린후 새롭게 V3를 커밋하고 Push 했다면 

1. Push 하는곳에서 오류가 발생 할 수 있고 2. Push가 되었다면 B에 저장된 V3는 무용지물이 된다

즉 DVCS를 제대로 이용할 수 없다. 
```

* 명령어
```
$ git reset HEAD~1	// 가장 최신의 버전으로 부터 1 전 버전으로 되돌리기
$ git reset HEAD^ 	// 가장 최신의 버전으로 부터 1 전 버전으로 되돌리기
$ git reset [HASH] 	// 가장 최신의 버전으로 부터 Hash 버전으로 되돌리기
```

* 옵션
```
$ --soft : 파일의 상태만 변경(내용은 변경 X), Default
$ --hard : 파일의 내용까지 모두 변경
```

- Ex

1. Soft
* `soft`를 사용할 때는 V5 까지 Committed 했을때 V4,5 가 서로 의미있는 형태라면 `soft`를 사용해 V3로 되돌린 후에 V4,5를 묶어서 V4로 Commit 한다
```
$ git log --oneline
b763bf9 (HEAD -> master) Renamed a to abc
88fda49 modified a to abc
d127ec4 modified a to ab
89ef6f4 new a

// Soft
$ git reset HEAD~3
Unstaged changes after reset:
D 		a

// Reset to 89ef6f4 
$ git log --oneline
89ef6f4 (HEAD -> master) new a

// No Change File
$ ls
abc
$ cat abc
abc
```

2. Hard
```
$ git log --oneline
b763bf9 (HEAD -> master) Renamed a to abc
88fda49 modified a to abc
d127ec4 modified a to ab
89ef6f4 new a

// Hard
$ git reset HEAD~3 --hard
HEAD is now at 89ef6f4 new a

// Reset to 89ef6f4 
$ git log --oneline
89ef6f4 (HEAD -> master) new a

// No Change File
$ ls
a
$ cat abc
a
```

## Revert
- 해당 커밋으로 Cherry Pick 하게 되지만 그 기록을 남겨야 할때 `git revert`를 사용한다.
```
$ git revert [Hash]
``` 

# 원격 저장소
- Git 원격 저장소(Git server repositroy)와 git local 저장소에 연결하여 파일을 Fatch, Push 형태로 관리
- 협업을 위한 필수 Tool

## 원격저장소 추가/제거/확인

- 확인  
```
$ git remote 
$ git remote -v
```

- 추가(먼저 원격 저장소를 만들어야 한다)
```
// ssh 를 이용한 방식. local, http, ssh를 사용하지만 현재 PW를 이용한 방식은 막혀서 효율성이 떨어짐
$ git remote add [origin : Nick] git@github.com:[choi-dongmin : ID]/[git-test : RepoName].git 	

$ git remote -v
origin  git@github.com:choi-dongmin/git-test.git (fetch)
origin  git@github.com:choi-dongmin/git-test.git (push)
```

- Key 등록
```
$ ssh-keygen 	// 키생성 후 해당 원격 Repo ID로 로그인 후 Setting 에서 SSH Key 등록으로 등록

$ cat ~/.ssh/id_rsa.pub 	// pub 확장자가 공개키 형태 확인
 							// ssh-rsa 에서 부터 키의 마지막(호스트 이름 앞의 문자)까지 복사후 등록
```
1. Setting 확인
![화면 캡처 2022-07-06 012456](https://user-images.githubusercontent.com/57117748/177374163-6f8a2dab-c5dc-41ca-91e7-7f29d718db8a.png)

2. Setting 에서 SSH 키 설정 진입
![화면 캡처 2022-07-06 012522](https://user-images.githubusercontent.com/57117748/177374192-94cc68fe-30f0-459e-ba03-50a5792989e6.png)

3. 키 등록 화면 진입
```
![화면 캡처 2022-07-06 012532](https://user-images.githubusercontent.com/57117748/177374251-f53fed8f-2c76-40dd-8cd4-83fd81114343.png)

4. 등록 
- 상단에 키 이름
- 하단에 공개키 값
```
![화면 캡처 2022-07-06 012541](https://user-images.githubusercontent.com/57117748/177374301-3275df23-e768-4c6b-a48e-69fd6d62a073.png)

- 제거
```
$ git remote -v
origin  git@github.com:choi-dongmin/git-test.git (fetch)
origin  git@github.com:choi-dongmin/git-test.git (push)

$ git remote rm origin

$ git remote -v
```

## 원격 저장소에 가져오기 / 올리기

1. 원격 저장소에 가져오기
- 연결된 원격 저장소의 버전을 현재 로컬 저장소로 가져오기
- fetch 방식은 원격 저장소를 로컬로 그냥 가져오고
- pull 방식은 원격 저장소를 로컬로 가져와 합치기(merge)
```
$ git fetch [origin : REMOTE] 	// 가져오기
$ git pull [origin : REMOTE] [master : LOCAL] // 가져오기 & 합치기
```

2. 원격 저장소에 올리기 
- 로컬 저장소의 버전을 원격 저장소에 동기화 
- 명령어는 `git push` 명령어 사용
```
git push [origin : REMOTE] [master : LOCAL]
```

* 만약 Local과 Remote가 같은 파일을 수정하고 더 나아가 같은 라인을 수정한다면 Push를 하려고 한다면 거부를 당한다. (Auto Mergy 불가, 수동우로 수정)


# Git Collaborator and RP
- GIT을 다른 사람과 협업하는 데 유용한 도구로 사용할 수 있다
- pull은 오픈 소스의 경우, 자유롭게 할 수 있지만
- push는 저장소에 대한 권한이 없다면 실행을 할 수 없다
- Push의 권한을 주는것을 Collaborator
- 자신의 저장소로 Fork하여 수정을 거쳐 Pull requests을 하는것이 PR

1. Collaborator 
- git remote 저장소에 setting 에서 Collaborator 항목에서 Add People
- USER를 찾고 그에 맞는 권한을 설정(Push)
- 해당 USER가 이메일을 통해 동의하고 인증한다면 그 유저는 Push 권한을 가짐

2. PR  
1. PR 할 저장소에 버전 확인 후 Fork 진행하여 내 저장소로 forking(원격저장소 우측 상단)

2. local과 remote 저장소를 연결하여 `git pull`, `git fetch`를 이용해 동기화

3. loacl 에서 modified, staged, commited 진행

4. 연결된 remote 저장소에 Push = 수정한 Loacl 을 원격 저장소에 동기화

5. 원격 저장소에 Pull Requests / New Pull Requests 버튼으로 Fork 해온 저장소에게 PR 



# Tag
- Git도 Tag를 붙여 배포 할 수 있다
- 보통 Version을 0.0.0 식으로 나타낸다.
- 현재 Head가 바라보는 버전에 Tag를 지정한다
```
$ git tag 0.1.0 -m 'v1 0.1.0'

$ git log --oneline
7c99af1 (HEAD -> master, tag: 0.1.0) add a:abc
```

# Branch
- branch 란 협업을 하는데 있어 독립적인 작업을 하기 위한 개념이다. 
- 각각의 branch에서 동시에 개별적인 작업들을 수행한다.
- 여러 사람이 동시에 작업하면서서로 영향을 미치지 않게 한다.
- 진행 끝난후에는 메인 브랜치로 Mergy 하여 변경사항을 저장 할 수도 있다.

* Local(master) / Remote (origin/master) 각각이 다른 branch

1. main Branch : 배포 할 수 있는 수준의 안정적인 Branch
2. Topic Branch : 기능 추가, 버그 수정과 같은 단위의 작업을 위한 Branch (이 브런치는 수시로 생성/삭제)

![Branch](data:image/png;base64,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)

## branch 확인
```
$ git branch

$ git branch
* master
```

## branch 생성
```
$ git branch [NAME]
```

## branch 전환
```
$ git checkout [NAME]

$ git checkout -b [NAME] // 생성, 전환
```

## branch 병합(merge)
- Merge 는 단순히 병합 될 다른 branch의 결과물만을 흡수

1. 병합의 모체가 될 branch 이동
```
$ git branch master
```

2. branch 병합
```
$ git merge [NAME]
```

## branch rebase 
- rebase는 병합과는 다르게 해당 branch의 모든 커밋을 가져옴 즉, 모든 Log를 가져옴 
```
$ git rebase [NAME]
```

## 출처
[branch 이미지 출처](https://www.atlassian.com/git/tutorials/using-branches)