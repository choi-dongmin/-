# DevOps
- DevOps는 Development + Operation 의 합성어
- Development: 프로세스 혹은 App을 구성하기 위한 code 구현
- Operation: Computing Engin에 올려 서비스를 제공하는 것으로 지속적 유지와 지속적인 관리가 필요
- 개발에서부터 운영까지 그 전과정의 프로세스 관리 및 운영(넓은 범주로 인해 자동화 필수)
- Monitoring, maintenance 

```
  ----------------------------->

dev : 시작 --------------> ops : 끝

  <-----------------------------
``` 

* 결국 그 넓은 범주로 인해 각각의 DevOps는 하는 일이 차이가 크다.
* 그러나 결국 DevOps는 개발자가 개발에 집중 할 수 있도록 개발, 운영 사이에 발생하는 프로세스 자동화를 지원한다.
* 자동화를 위해서는 CI/CD가 필요(CI/CD는 DevOps에 일부분)
* CI : Continue Integration : 지속적 통합
* CD : Continue Deployment : 지속적 배포


# Software Developement Method

## Waterfall Model
- S/W 개발의 하나의 방법론
- 기본적으로 절차적으로 진행되며 한번 진행된 절차는 거스르기 힘들다
- Project 기간이 굉장히 길다

![Waterfalls Model](https://upload.wikimedia.org/wikipedia/commons/thumb/e/e2/Waterfall_model.svg/1200px-Waterfall_model.svg.png?20131021150906)

1. Reqirements 
	- Client의 Spec, Demands, Needs 등 클라이언트와 원하는 요구사항을 결정하는 단계

2. Design
	- 클라이언트와 결정한 요구사항 등을 디자인 및 구성 설계

3. Implementation
	- 해당 디자인 및 설계에 따라 구현하는 단계

4. Verification
	- 구현된 S/W를 검증하는 단계로 QA 

5. Maintenance
	- 계속해서 유지 및 보수 


## Agile
- 현재 주목받는 S/W 개발 방법론
- 선순환의 구조 형태로 당장 구현 가능한 S/W 빠르게 구현한다.
- 한번의 개발이 끝난후 다시 선순환의 구조로 개발된다(Sprint 단위)
- Version의 형태

![Agile Model](https://sp-ao.shortpixel.ai/client/to_auto,q_lossless,ret_img,w_1024/https://www.soldevelo.com/blog/wp-content/uploads/Agile-software-dev-1-1024x476.jpeg)


# Git 개요
- 버전관리(VSC : Version Control System) : 파일에 대한 버전 관리를 별도의 시스템에 따라 관리
1. 쉽게 RollBack 가능
2. 어디서 문제를 일으켰는지도 추적 가능
3. 시간에 따라 수정 내용을 비교 가능 

## Local VSC
![Local VSC](https://git-scm.com/book/en/v2/images/local.png)

## CVCS
![CVCS](https://git-scm.com/book/en/v2/images/centralized.png)

- 중앙집중식 버전 관리 
- 다른 개발자와 함께 작업해야 하는 경우 생기는 문제를 해결하기 위해 CVCS(중앙집중식 VCS)가 개발
- 중앙 서버에서 파일을 받아서 사용하는 방식
- 누가 무엇을 하고 있는지 알 수 있고 VCS 하나를 관리하기가 훨씬 쉽다
- 그러나 Origin VCS에 결함이 발생하면 아무것도 할 수 없는 상태가 된다
- 즉, 중앙집중화로 인한 위험도가 증가한다

## DVCS
![DVCS](https://git-scm.com/book/en/v2/images/distributed.png)

- 분산 버전 관리 시스템
- 저장소를 히스토리와 더불어 전부 복제한다. 서버에 문제가 생기면 이 복제물로 다시 작업을 시작한다
- 클라이언트 중에서 아무거나 골라도 서버를 복원할 수 있으며 Clone은 모든 데이터를 가진 진정한 백업이다.
- 진정한 고가용적 버전관리


## Git의 3가지 관리 상태
- Git은 파일을 Modified, Staged, Committed 이렇게 세 가지 상태로 관리한다.
- Git 버전관리에 매우 중요한 내용 

! Committed, Modified, Staged](https://git-scm.com/book/en/v2/images/areas.png)


1. Modified
	- Working Directory 에서 생성, 수정, 삭제를 한 파일이 Stage에 보내지 않은 상태
	- 즉, 수정 파일을 Add 하지 않은 상태
	- git add 명령으로 Staged 상태로 진입

2. Staged 
	- 수정한 파일이 Working Diractory 에서 Staging Area로 넘어온 상태
	- 곧 Commit 할 것이라고 표시하는 상태
	- 실제 서버와 유사한 환경에서 상태 테스트(Staging)
	- 마지막 확인, 검증
	- git commit 명령을 통한 committed 상태로 진입

3. Committed
	- 버전으로 확정되어 로컬 데이터베이스에 안전하게 저장 됐다는 것을 의미
	- Message 작성 필요(주석 및 설명)
	- 버전을 Commit 시에는 의미있는 단위로 버전을 확정해야 한다.


# Git을 이용한 서버관리

## zsh 및 oh my zsh 설치

1. zsh 설치
- Ubuntu 패키지 업데이트
```
$ sudo apt update
```

- zsh 설치
```
$ sudo apt install -y zsh
```

- 기본 shell 바꾸기
```
$ sudo cash -s /usr/bin/zsh
```

- Oh My Zsh
```
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

- powerlevel10K 설치
```
$ git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```

- Theme 변경 (11번째)
```
vi ~/.zshrc
```
```
11 ZSH_THEME="powerlevel10k/powerlevel10k"
```

- windows Terminal 에서 fonts 설정

[fonts 다운](https://github.com/romkatv/powerlevel10k#fonts)

![화면 캡처 2022-07-04 204603](https://user-images.githubusercontent.com/57117748/177148490-3c4462ba-f441-4bb2-8e42-d1227f3c496d.png)

- 다운 받은 글꼴 설치
![화면 캡처 2022-07-04 204903](https://user-images.githubusercontent.com/57117748/177148902-9e227b9c-0014-40df-9615-8d1e28bf0467.png)

- windows Terminal 에 적용
```
터미널에서 ctrl + shift + , 
```
```
https://raw.githubusercontent.com/romkatv/dotfiles-public/aba0e6c4657d705ed6c344d700d659977385f25c/dotfiles/microsoft-terminal-settings.json

해당 URL 정보 복사 붙여넣기 후 재접속 
```

- oh my zsh 개인 설정
![화면 캡처 2022-07-04 205334](https://user-images.githubusercontent.com/57117748/177149601-cc0c2bfe-07ba-470b-88c5-b5f197f978cf.png)

- zsh-syntax-highlighting Plugin 설치
```
$ git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
```
vi ~/.zshrc
```
```
80 plugins=(git zsh-syntax-highlighting)
```

- zsh-autosuggestions
```
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```
```
vi ~/.zshrc
```
```
80 plugins=(git zsh-syntax-highlighting zsh-autosuggestions)
```

## Git 초기설정
1. /etc/gitconfig : 모든 사용자와 모든 저장소에 적용되는 설정을 선언

2. ~/.gitconfig : 특정 사용자(현재 사용자)에게만 적용되는 설정(공동작업시 사용자 메일, 이름 등 지정)

3. .git/config : Git Directory의 특정 저장소에 적용


- Git 사용자 정보 설정
```
$ git config --global user.name "[사용자 이름]"
$ git config --global user.email [사용자 메일]
```

- Git text Editor Setting
```
$ git config --global core.editor vim
```

- 설정된 항목 리스트 확인
```
$ git config --list
```

## Git 저장소 및 서버관리
- Git 저장소 생성 
- .git = git DB

1. 빈 디렉토리 에서 설정 
```
$ mkdir ~/git-test
$ cd git-test

$ git init
$ ls -al
total 12
drwxrwxr-x 3 vagrant vagrant 4096 Jul  4 13:32 .
drwxr-xr-x 6 vagrant vagrant 4096 Jul  4 13:32 ..
drwxrwxr-x 7 vagrant vagrant 4096 Jul  4 13:32 .git 	// .git 파일 생성 확인
```

2. 기존에 버전 관리중이던 저장소에서 가져오기
```
$ git clone [URL]

$ git clone http://github.com/kubernetes-sigs/kubespray.git
								   (ID) 	   (Repo Name)

$ git clone http://github.com/kubernetes-sigs/kubespray.git abc  // abc 라는 디렉토리를 만들어 저장
``` 

- git status
```
$ git status
```
```
$ echo "hello world" > hello.txt

$ git status
On branch master

No commits yet

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        hello.txt

nothing added to commit but untracked files present (use "git add" to track)
```

- git stage
```
$ git add [File]
```
```
$ git add hello.txt

$ git status
On branch master

No commits yet

Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
        new file:   hello.txt
```

- git commit
```
$ git commit 	// 에디터로 연결되어 정의
$ git commit -m [MESSAGE]
```
```
$ git commit hello.txt -m "say hello"
[master (root-commit) d86dd4a] say hello
 1 file changed, 1 insertion(+)
 create mode 100644 hello.txt

$ git status
On branch master
nothing to commit, working tree clean
```

- git log
```
$ git log
```
```
$ git log


commit d86dd4a149d00c0a2c476ee175ea5fccb83c36eb (HEAD -> master)
Author: DongMin Choi <abcd@efgh.com>
Date:   Mon Jul 4 13:48:19 2022 +0000

    say hello
```

## 출처
[Waterfall Model 이미지 출처](https://commons.wikimedia.org/wiki/File:Waterfall_model.svg) , [Agile Model 이미지 출처](https://www.soldevelo.com/blog/is-agile-always-the-best-solution-for-software-development-projects) , [VSC 이미지 출처](https://git-scm.com/book/ko/v2/%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0-%EB%B2%84%EC%A0%84-%EA%B4%80%EB%A6%AC%EB%9E%80%3F)
