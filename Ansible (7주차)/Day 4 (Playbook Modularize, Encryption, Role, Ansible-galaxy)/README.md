# Playbook modularize
- 작업이 진행되면서 필연적으로 작업의 내용이 길어지고 복잡해지는데 모듈화를 통해 조금더 유지보수를 효율적이게 한다.
- 변수, 작업, 플레이북 파일 등을 모듈화가 가능하다.

## 변수 파일의 모듈화
- vars_files 키워드를 사용하여 플레이북에서 불러온다.
- 인번토리, 플레이북 파일과 동일한 경로에 디렉토리 및 파일을 준비한다.
- group_vars, host_vars 를 통해 엔진에서 자동으로 로드 할 수 있다.


## 블레이북 파일의 모듈화
- import_playbook 지시문을 사용한다.
- 마스터 플레이북에서 가져오는 순서대로 플레이 실행


## 작업 파일의 모듈화
- tasks 블록에 대한 내용만 파일로 따로 저장가능
- 마스터 플레이북의 tasks 섹션에서 import_tasks / include_tasks 를 사용해 불러온다.
- import와 indluce의 가져오기 방식의 차이점
```
항목 					include 		import
재사용 형식 				동적 			정적
처리 시점 				모듈 실행 시 	플레이북 파싱 시 전처리
반복문 사용 				가능 			불가능
태그 및 작업 목록 확인 	불가능 			가능
핸들러 알림 				통째로 호출 		파일 내의 이름 호출 가능
작업파일 				가능 			가능
플레이북 				불가능 			가능
변수파일 				가능 			불가능
```
## tasks 모듈화 사용
1. 웹서버 서비스를 위한 task 모듈 작성
	- yum 을 이용한 service 패키지 설치 task
	```
	---
	# install_package.yaml
	#
	- name: package install
	  yum:
	  	name: "{{ service.package }}"
	    state: latest
	...
	```
	![화면 캡처 2022-06-10 181058](https://user-images.githubusercontent.com/57117748/173033675-5bbe5ca1-4338-4425-94b7-5b81d339e08d.png)


	- 설치한 서비스 시작 및 활성화 task
	```
	---
	# active_service_httpd.yaml
	#
	- name: active_httpd
	  service:
	    name: "{{ service.package }}"
	    state: started
	    enabled: yes
	...
	```
	![화면 캡처 2022-06-10 181121](https://user-images.githubusercontent.com/57117748/173034305-f1944e36-bea1-47a8-88a1-ea1170586acf.png)


	- 방화벽 해제 task
	```
	---
	# firewall_setting_httpd.yaml
	#
	- name: firewalld restart
	  service:
	    name: firewalld
	    state: restarted
	    enabled: yes
	- name: use firewall for service
	  firewalld:
	    service: "{{ service.name }}"
	    immediate: yes
	    permanent: yes
	    state: enabled
	...
	```
	![화면 캡처 2022-06-10 181159](https://user-images.githubusercontent.com/57117748/173034613-07dd71da-f97a-46fa-b39e-51d3981e2e71.png)

2. 마스터 플레이북에서 tasks 사용
	- 변수를 마스터 플레이북에서 지정 가능
	```
	---
	- name: webserver_setting_module
	  hosts: localhost
	  vars:
	    - service:
	        package: httpd
	        name: http
	  tasks:
	    - import_tasks: install_package.yaml
	    - import_tasks: active_service.yaml
	    - import_tasks: firewall_setting_service.yaml
	...
	```
	![화면 캡처 2022-06-10 181229](https://user-images.githubusercontent.com/57117748/173035115-36a7da2f-b982-46bc-be33-5f69945c8c56.png)

# 암호화
- 평문으로 만든 파일을 암호화 하는것
- 구성파일을 제외한 파일들이 암호화가 가능하다(플레이북, 인벤토리, 변수파일 등)
- ansible-vault 명령어를 통한 관련 작업 수행
- 암호화된 플레이북을 실행시 --vault-id [ID]@prompt [Filename] 으로 실행하며 ID 는 제외 가능하다.
- 플레이북에 포함된 (작업파일/변수/인벤토리..) 중 하나라도 암호화 된 경우에는 --vault-id 를 사용
- 문자열만 암호화도 가능
- 단일 패스워드 / 다중 패스워드 설정 가능
	1. 단일 패스워드 : 하나의 패스워드로 다중의 사용자가 모든 파일에 접근
	2. 다중 패스워드 : 각각의 개인 팀 파일 마다 지정된 패스워드로 파일로 접근

## ansible-vault command
```
- create : 새로운 암호화 파일 생성
- view : 암호화 된 파일을 cat 명령어와 같이 출력
- edit : 파일을 편집
- encrypt : 기존 파일을 암호화
- decrypt : 암호화 파일을 해제
- rekey : 암호화 파일의 패스워드 변경
- --vault-id : playbook 파일 실행
```

## 암호화 사용하기
1. ansible-vault create [filename]
	- 암호문을 만드는 명령어 
	- 처음에 패스워드를 설정
	- view 를 통한 출력
	```
	$ ansible-vault create test.yaml	// 패스워드 설정 필요
	$ cat test.yaml	// 암호문 출력
	$ ansible-vault view test.yaml	// 암호문을 평문으로 정상 출력
	```
	![화면 캡처 2022-06-10 121606](https://user-images.githubusercontent.com/57117748/173038572-f743d96d-d1bf-4e1c-8cd7-149d538c6fd8.png)


2. ansible-vault encrypt / decrypt [filename]
	- encrypt 를 통한 암호화
	- decrypt 를 통한 해체
	```
	$ ansible-vault encrypt test.yaml
	$ ansible-vault decrypt test.yaml
	```
	![화면 캡처 2022-06-10 122003](https://user-images.githubusercontent.com/57117748/173040283-3dcada47-279f-4331-a157-3df9cc0f0fa6.png)


3. ansible-vault rekey [filename]
	- rekey를 통한 패스워드 재설정
	```
	$ ansible-vault rekey test.yaml
	```
	![화면 캡처 2022-06-10 122119](https://user-images.githubusercontent.com/57117748/173040864-bc6db3ee-003e-4567-8fea-1e8b7cea800d.png)


4. ansible-vault --vault-id [IP/hostname]@prompt [파일경로]
	- 현 운영체제에서 사용시 ID 부분 제외가능
	```
	$ ansible-vault --vault-id @prompt test.yaml
	```
	![화면 캡처 2022-06-10 122848](https://user-images.githubusercontent.com/57117748/173042130-1be5310d-7051-432f-af00-79c22b7867a5.png)


# 역할(role)
- 모든 코드를 다른 플레이북에 복사하는것은 쉽지 않다
- 작업, 변수, 파일, 템플릿 및 기타 리소스를 디렉토리로 패킹화
- 디렉토리를 복사해서 해당 역할을 프로젝트 간 복사
- 다른 사람들과 서로 공유
- 역할 관리 및 공유시 ansivle-galaxy 명령어를 사용
- 디렉토리의 구조로 이루어진 하나의 작업 단위

## 역할의 구조
- 최상위 디렉토리는 역할 자체에 이름을 정의
- 하위 디렉토리는 각 파일에 목적에 따라 정의
- 다른 사람들과 서로 공유 가능

```
하위 디렉터리 		기능
defaults 			역할 변수의 기본값이 저장 (main.yml)
files 				역할 작업에서 참조한 정적 파일
handlers 			역할의 핸들러 정의 (main.yml)
meta 				역할에 대한 정보 (main.yml)
tasks 				역할의 작업 정의 (main.yml)
templates 			역할 작업에서 참조한 Jinja2 템플릿
tests 				역할을 테스트하는 데 사용할 수 있는 인벤토리와 test.yml 플레이북이 포함
vars 				역할의 변수 값을 정의 (main.yml)
```


## 역할생성(role crate)
```
$ ansible-galaxy role init [파일경로 및 이름]
```
![화면 캡처 2022-06-10 144444](https://user-images.githubusercontent.com/57117748/173043817-a07f18ae-abce-49cc-92ea-bf5ae115cae1.png)

## 작업의 우선순위
- 기본 
```
플레이북의 tasks -> 플레이북의 handler
```

- 역할 추가시
```
역할의 tasks  -> 플레이북의 tasks -> 역할의 handler -> 플레이북의 handler
```

- pre_tasks 항목 추가 시
```
플레이북 pre_tasks 항목 -> 역할의 tasks -> 플레이북의 tasks -> 역할의 handler -> 플레이북의 handler
```

## ANSIBLE GALAXY
- 여러 Ansible 관리자와 사용자가 작성한 공용 Ansible 역할 라이브러리
- 검색 기능을 제공
- ansible-galaxy 명령으로 자신의 Git 리포지토리로 관리 가능
- 키워드, 작성자, 플랫폼 및 태그를 사용하여 관련 역할 검색 가능
- ANSIBLE GALAXY 명령
```
$ ansible-galaxy search : 역할 검색
$ ansible-galaxy info : 역할에 대한 세부 정보 표시
$ ansible-galaxy install : 역할 다운로드 및 설치
$ ansible-galaxy list : 로컬 시스템에 있는 역할을 나열
$ ansible-galaxy remove : 로컬 시스템의 역할 제거
```

## 최종 작업의 디렉토리 구조
```
├── ansible.cfg
├── group_vars
│   └── group1
├── host_vars
│   └── host1
├── inventory
├── playbook.yaml
├── play.yaml
├── roles
│   └── role01
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
├── tasks.yaml
└── vars
    └── name.yaml
```