# AD-HOC과 모듈

## AD-HOC 
- 하나의 Ansible 작업을 신속하게 실행하는 방법
- 신속한 테스트 및 변경에 유용하다
- Ansible 명령어 사용
- 호스트를 직접 지정 혹은 -i 옵션을 통해 인벤토리 파일을 지정
- -m 옵션으로 사용할 모듈 지정
- -a 옵션으로 추가 인수 입력
- 기본 문법
```
# ansible <parrern_goes_here> -m <module_name> -a <arguments>
```

## Ad-HOC command option
-i : inventory
-u : remote_user
-b : become
--become-method : become_method
--bavome-user : become_user
-K : become_ask_pass


## Ansible Module

## Module 사용 확인
- ansible-doc 으로 모듈 정보 및 사용법 확인 가능
	* ansible-doc -l 명령으로 모듈 목록을 확인
	* ansible-doc [module name] 명령을 사용해 모듈에 대한 자세한 문서 확인
	* ansible-doc -s [module name] 명령을 사용해 모듈에 대한 예제 확인
- command, shell, raw 위 3가지의 모듈은 멱등성을 보장하지 않는다(주로 테스트 용도)
- ping 모듈을 통해 시스템의 네트워크 연결을 체크할 수 있다
- 다양한 관리 모듈을 지원 (copy, fetch, template, file...)
- 다양한 패키지 관리 모듈 지원(package, yum, dnf, apt...)
- 사용자, 서비스, 파티셔닝 등 시스템 관리 모듈이 다양하다.

# Playbook
- AD-HOC 명령은 하나의 간단한 작업을 일회성 명령으로 실행한것
- Playbook 방식은 여러 개의 복잡한 작업을 쉽고 반복적으로 실행
- 동기식, 비동기식 작업 가능
- YAML 언어 사용
- 1개 이상의 Play 가 포함 되어 있어야 한다
- 문서 시작과 끝에 관습적으로 ---, ... 을 사용(생략가능)
- 목록의 항목은 하나의 대시와 공백으로 시작

## Playbook 구조
- 기본적으로 name, hosts, tasks 로 구성
1. name
	- 생략이 가능하지만 명시적으로 사용
	- 각 플레이를 구분하는 구분자

2. hosts
	- 각 플레이 별 작업 대상 지정
	- 개별 호스트 및 그룹 지정 가능

3. tasks
	- 실질적인 작업 내용을 지정
	- 각종 모듈을 사용

## Playbook 사용
- ansible-playbook 명령어를 통해 사용
- name 속성을 설정하면 플레이북의 실행  진행 상황을 더 쉽게 모니터링 가능
- 일반적인 ansible-playbook 작업은 멱등성을 가짐
- 플레이북을 여러번 실행하는것이 더 안전함
- 호스트가 이미 올바른 상태라면 변경되지 않음 (ok 상태)
- -v 명령은 추가적인 정보 표시
- --syntax-check 옵션으로 문법적 오류 검사 가능
- --check 를 통해 작업의 가용성 검사 가능
- 공백이 있더라도 따옴표 묶지 않음

# Playbook 연습
0. vim 구성 및 YAML설정
- vim 편집기 사용(yum -y install vim)
- vim 구성파일 설정 
```
~/.vimrc
```
```
syntax on
autocmd FileType yaml setlocal ts=2 sts=2 sw=2 et ai
```
- ts : tab 버튼을 누를시 공백 수
- sw : 자동 들여쓰기와 함께 사용할 공백수
- et : 공백문자를 탭 대신 공백으로 사용
![화면 캡처 2022-06-08 182250](https://user-images.githubusercontent.com/57117748/172581603-c19db889-2b3b-483a-a38b-6cd76c4cfa9b.png)


## User 연습
- 호스트에 User 생성
```
# vim user.yaml
```
```
  - name: User Config
    hosts: host1
    tasks: 
      - name: user create
        user:
          name: playbook_user
          uid: 3000
          group: ansible
          shell: /bin/sh
```
```
$ ansible-playbook user.yaml
```

- 호스트 User 정보 수정
```
  - name: User Config
    hosts: host1
    tasks: 
      - name: user create
        user:
          name: playbook_user
          uid: 3000
          group: ansible
          shell: /bin/sh
      - name: user modify
        user:
          name: playbook_user
          uid: 4000
```
```
$ ansible-playbook user.yaml
```
- 호스트 User 삭제
```
  - name: User Config
    hosts: host1
    tasks: 
      - name: user create
        user:
          name: playbook_user
          uid: 3000
          group: ansible
          shell: /bin/sh
      - name: user modify
        user:
          name: playbook_user
          uid: 4000
      - name: user delete
      	user:
      	  name playbook_user
      	  state: absent
```
```
$ ansible-playbook user.yaml
```
![화면 캡처 2022-06-08 141825](https://user-images.githubusercontent.com/57117748/172584157-0e19ecf6-2d64-4894-b741-66b10ec37e52.png)
![화면 캡처 2022-06-08 141850](https://user-images.githubusercontent.com/57117748/172584983-cc7f070e-82d7-4fb7-ade3-7ea54eb41f85.png)

## 호스트에 group 생성 및 수정
```
$ vim group.yaml
```
```
  - name: Group config
    hosts: host1
    tasks: 
      - name: Create group
        group:
          name: test
      - name: Modify group
        group:
          name: test
          gid:2000
      - name: Delete group
      	group:
      	  name: test
      	  state: absent
```
```
$ ansible-playbook group.yaml
```
![화면 캡처 2022-06-08 184936](https://user-images.githubusercontent.com/57117748/172587489-9cd5db7b-79ba-438d-96fc-a2028da199ed.png)
![화면 캡처 2022-06-08 185317](https://user-images.githubusercontent.com/57117748/172588153-3ccdc885-24fa-4ee9-97c7-790cddba5889.png)


## 호스트의 디스크 관리
- state: info 를 사용해 해당 dev의 정보를 출력하는것으로 대체
```
  - name: Parted Module Test
    hosts: localhost
    tasks:
      - name: Partition State
        parted: 
          state: info
          device: /dev/sda
        register: result
      - name: Print Device Info
      	debug:
      	  var: result
```
```
$ ansible-playbook parted.yaml
```
![화면 캡처 2022-06-08 190630](https://user-images.githubusercontent.com/57117748/172590681-d371595d-7822-4416-aec1-96c1fd1535cd.png)
![화면 캡처 2022-06-08 190650](https://user-images.githubusercontent.com/57117748/172590684-2b59c6ec-86b4-4ef4-a3ed-faf285756b68.png)

## 파일관리 copy / file / lineinfile

- copy : 제어노드의 파일을 호스트로 복사(src, dest 필수)
```
  - name: copy from control-node to host1
    hosts: host1
    tasks:
      - name: user.yaml file copy
        copy:
          src: /home/vagrant/first_demo/user.yaml
          dest: /tmp/user.yaml
      - name: user2.yaml file copy
      	copy:
      	  src: /home/vagrant/first_demo/user.yaml
      	  dest: /tmp/user2.yaml
      	  owner: ansible
      	  group: vagrant
      	  mode: 0644
```
```
$ ansible-playbook copy.yaml
```

![화면 캡처 2022-06-08 151135](https://user-images.githubusercontent.com/57117748/172595615-ac7074a0-ac4f-497d-8d8e-e1f0ceacd83d.png)
![화면 캡처 2022-06-08 151713](https://user-images.githubusercontent.com/57117748/172595172-b60447ed-4882-4ddd-862c-eb48891ce5cf.png)
![화면 캡처 2022-06-08 151809](https://user-images.githubusercontent.com/57117748/172595179-fa4f9d0f-abc1-4362-90d5-60f28322b244.png)


- file : 호스트의 파일에 대한 속성 수정
```
  - name: file module
    hosts: host1
    tasks:
      - name: file permmision chagne
        file:
          path: /tmp/user2.yaml
          owner: root
          group: root
      - name: file delete
        file:
          path: /tmp/user2.yaml
          state: absent
      - name: create directory
        file:
          path: /tmp/dirA
          state: directory
```
```
$ ansible-playbook file.yaml
```

- lineinfile : 해당 파일의 라인의 값을 새로운 값으로 대체
```
  - name: lineinfile module test
    hosts: host1
    tasks:
      - name: file text remove
        lineinfile:
          path: /tmp/user.yaml
          regexp: Delete
          state: absent
      - name : file text edit
        lineinfile:
          path: /tmp/user.yaml
          regexp: Create
          line: test edit
      - name : file text add
        lineinfile:
          path: /tmp/user.yaml
          insertafter: uid
          line: new line added
```
```
$ ansible-playbook lineinfile.yaml
```

![화면 캡처 2022-06-08 165549](https://user-images.githubusercontent.com/57117748/172597382-fac6c781-c04c-4518-af83-4d9925eb3a16.png)
![화면 캡처 2022-06-08 165653](https://user-images.githubusercontent.com/57117748/172597431-d5cb658d-3605-436b-8868-2c7e077f50cf.png)



## 호스트의 Service를 관리

- yum httpd 패키지 설치
```
  - name: yum config
    hosts: host1
    tasks:
      - name : yum install
        yum:
          name: httpd
          state: latest
``` 
![화면 캡처 2022-06-08 205349](https://user-images.githubusercontent.com/57117748/172609996-decc64ce-c5d8-4452-9f07-f5fae3f5484f.png)

- httpd 시작 
```
  - name : service start and enable
    hosts: host1
    tasks:
      - name: httpd service start
        service:
          name: httpd
          state: started
          enable: yes
```
![화면 캡처 2022-06-08 205708](https://user-images.githubusercontent.com/57117748/172610575-8391a303-7ebc-40f7-85d9-0a2bd8acf9ee.png)


- firewall 해체
```
  - name: firewall setting config
    hosts: host1
    tasks:
      - name: firewalld service start
        service:
          name : firewalld
          state: started
          enabled: yes
      - name: use firewall
        firewalld:
          service: http
          immediate: yes
          permanent: yes
          state: enabled
```
![화면 캡처 2022-06-08 164300](https://user-images.githubusercontent.com/57117748/172612920-27b27e8e-5105-402b-bedb-70d19487b4bd.png)
![화면 캡처 2022-06-08 164707](https://user-images.githubusercontent.com/57117748/172612947-86851092-da8b-4415-b039-ec37d44a67d6.png)


# 변수 및 팩트

# Ansible 변수
- 파일 전체에서 재사용할 수 있는 값을 저장하는 데 사용할 수 있는 변수를 지원
- 변수에 포함할 수 있는 값
	* 생성할 사용자
	* 설치할 패키지
	* 다시 시작할 서비스
	* 제거할 파일
	* 인터넷에서 검색할 아카이브
- 변수 이름 지정
	* 문자로 시작
	* 문자, 숫자, 밑줄만 포함
- 변수범위
	1. 전역범위 : 명령줄 또는 Ansible 구성에서 설정한 변수로 어느곳에서나 사용가능
	2. 플레이범위 : 플레이 및 관련구조에서 설정한 변수로 플레이 내에서 사용가능
	3. 호스트범위 : 인벤토리, 팩트 수집 또는 호스트 그룹 및 개별 호스트에서 설정한 변수
	- 우선순위 : 전역 > 플레이 > 호스트

## 플레이북 변수
- 플레이북 시작 위치에 있는 vars 블록에 변수를 배치 혹은 외부 파일에 플레이북 변수를 정의
- 변수의 이름을 따옴표와 중괄호로 묶어 사용
![화면 캡처 2022-06-08 172139](https://user-images.githubusercontent.com/57117748/172617852-422542e9-3bab-49c3-bbb0-7e8828d332c9.png)
![화면 캡처 2022-06-08 172252](https://user-images.githubusercontent.com/57117748/172617859-84211f30-7197-448e-85ff-7a3711e8ff9f.png)

## 호스트 변수 
- 개별 호스트 변수가 그룹 변수 보다 우선순위 높음
- 인벤토리 파일에 직접 설정이 가능하지만 권장되지 않는다.
- 작업디렉토리에 group_vars 및 host_vars의 두 개 디렉토리 생성 권장
- 호스트 이름 및 그룹 이름으로 파일 생성 후 변수 정의

## 배열을 변수로 사용
- 배열 형식의 변수 사용 가능
- 사용 시 .(점)표기법 혹은 대괄호 표기법을 사용

## 등록된 변수를 사용해 명령 출력 캡쳐
- register 문을 사용해 명령 출력을 캡쳐

# 팩트
- Ansible facts는 호스트에서 자동으로 검색한 변수
- 호스트의 정보를 변수로 저장한 것
- 일반적으로 첫 작업전 setup 모듈을 자동으로 실행(팩트수집)
```
팩트 			 				변수
짧은 호스트 이름 					ansible_facts['hostname']
정규화된 도메인 이름 				ansible_facts['fqdn']
기본 IPv4 주소(라우팅 기반) 		ansible_facts['default_ipv4']['address']
네트워크 인터페이스 목록 			ansible_facts['interfaces']
/dev/vda1 디스크 파티션 크기 		ansible_facts['devices']['vda']['partitions']['vda1']['size']
DNS 서버 목록 					ansible_facts['dns']['nameservers']
현재 실행되고 있는 커널 버전 		ansible_facts['kernel']
```

## 팩트 수집 끄기
- gether_facts 키워드를 no: 로 설정해서 비활성화
- 비활성화하면 플레이 속도 증가, 관리노드의 부하가 감소

## 사용자 지정 팩트 만들기
- 제어 노드는 각 관리 노드에 로컬로 저장된 사용자 지정 팩트를 생성 가능
- 관리 노드에서 setup 모듈이 실행될 때 수집되는 표준 팩트 목록으로 통합
- 관리 노드에서 플레이의 동작 조정에 사용 가능한 임의 변수를 ansible에서 제공
- INI 파일로 또는 Json을 사용해 형식이 지정된 정적 파일에 정의
- 관리 노드의 /etc/ansible/facts.d 디렉토리에 있는 파일 및 스크립트에서 로드
- 파일 또는 스크립트는 이름이 .fact 으로 끝나야 사용가능
- setup 모듈에 의해 ansivle_fact.ansible_local 변수에 저장

## 매직 변수
- 팩트가 아니거나 setup 모듈을 통해 구성되지 않지만, Ansible에 의해 자동 설정
- 특정 관리 호스트에 고유한 정보를 얻는 데 유용
- 예약된 변수
- 매직 변수 종류
	1. hostvars
	2. group_names
	3. groups
	4. inventory_hostname


# 연습문제
1. user 모듈의 사용자 이름을 변수화 시켜라
```
---
  - name: User config
    hosts: host1
    vars:
      - username: test
    tasks:
      - name: Create user
        user:
          name: "{{username}}"
          uid: 2000
          group: ansible
      - name: Modify user
        user:
          name: "{{username}}"
          uid: 3000
      - name: Delete user
        user:
          name: "{{username}}"
          state: absent
...
```
![화면 캡처 2022-06-08 215024](https://user-images.githubusercontent.com/57117748/172620596-f1f6c3ef-b9ef-43d3-90fd-7ba77033fe41.png)

2. yum 모듈에서 패키지를 변수화 하라
```
---
  - name: yum config
    hosts: host1
    vars:
      - package: httpd
    tasks:
      - name: yum install
        yum:
          name: "{{package}}"
          state: latest
...
~             
``` 
![화면 캡처 2022-06-08 215210](https://user-images.githubusercontent.com/57117748/172620952-cd75e724-9ae1-4399-8bc5-cd803762e51c.png)


3. service 모듈에서 서비스 이름 변수화 하라
```
---
  - name: service start and enable
    hosts: host1
    vars:
      - service_name: httpd
    tasks:
      - name: httpd service start
        service:
          name : "{{service_name}}"
          state: started
          enabled: yes
...
```
![화면 캡처 2022-06-08 215410](https://user-images.githubusercontent.com/57117748/172621498-2e2d40da-398f-430f-b44e-1b5ad4a88e75.png)

4. 사용자 설정 실습 (사용자/그룹 이름은 변수로 사용)
	- user 모듈을 이용해서 사용자 생성 (testuser , UID : 2022)
	- group 모듈로 그룹 생성 (study  , GID : 2030)
	- user 모듈을 이용해서 사용자 설정 변경 (testuser 사용자의 기본그룹을 study 로 변경 / ansible 사용자의 보조그룹에 study 를 추가)

```
---
  - name: User config
    hosts: host1
    vars:
      - user_name: testuser
      - group_name: study
    tasks:
      - name: Create user
        user:
          name: "{{user_name}}"
          uid: 2022
      - name: Create group
        group:
          name: "{{group_name}}"
          gid: 2030
      - name: Modify user testuser
        user:
          name: "{{user_name}}"
          group: "{{group_name}}"
      - name: Modify user ansible
        user:
          name: ansible
          groups: "{{group_name}}"
...                                 
```

5. web 구성
- 기본 ansible.cfg, inventory
```
[defaults]
inventory = inventory
remote_user = ansible
ask_pass = false

[privilege_escalation]
become = true
become_ask_pass = false
```
```
[web]
host[1:3]

[db]
host4

[servers:children]
web
db
```

- 플레이북에서 호스트 지정 시 web 그룹 지정
- setting.yaml 파일에서 변수 불러오기
- 패키지이름/서비스이름 변수로 지정
- copy / yum / service / firewalld 모듈을 이용
- 컨트롤노드에서 index.html 파일을 만들어놓고 copy 모듈을 이용해서 각 호스트에 복사

- 위 조건들에 맞는 플레이북 작성 및 실행
![화면 캡처 2022-06-09 004935](https://user-images.githubusercontent.com/57117748/172661155-dcd4bf18-be68-4c46-bc4b-7edda91ae12d.png)
![화면 캡처 2022-06-09 005157](https://user-images.githubusercontent.com/57117748/172661701-1b50da98-b322-49d9-bfb9-7954098efb0b.png)

- curl 명령어를 통해서 web 그룹의 호스트들에 접속해서 확인해보기
![화면 캡처 2022-06-09 005244](https://user-images.githubusercontent.com/57117748/172661924-1908f069-c9a2-4239-9095-c9dba273b088.png)


* 패키지 다운로드시 오류
- 관리노드에서 패키지 실행 오류 
```
$Error: Cannot find a valid baseurl for repo: appstream

$Errors during downloading metadata for repository 'C6.10-base' 
```

- 해결 (각 관리 노드마다 적용)
```
$ sudo vi /etc/resolv.conf
```
```
# Generated by NetworkManager
nameserver 8.8.8.8
~                        
```

6. 호스트 파티셔닝
- 가상머신 종료 후 디스크 1개 추가
![화면 캡처 2022-06-09 005953](https://user-images.githubusercontent.com/57117748/172663560-a050350a-916e-42bc-ae8f-a2bce169ba29.png)


- 해당 가상머신에 대한 파티셔닝 작업을 앤서블로 진행 (parted 모듈을 사용)
- 1번~3번 파티션은 1G 크기로 primary 종류 , 4번 파티션은 나머지 전체 크기로 extended
![화면 캡처 2022-06-09 114800](https://user-images.githubusercontent.com/57117748/172753675-3ee25cdd-16c7-4478-aed6-64a5bba5af63.png)


## 키워드
- AD-HOC : 단발성으로 Ansible에게 명령을 실행할때 사용한다. 명령어 구문
- Playbook: 단발성이 명령이 아닌 다발적, 연속적으로 쓰기위해 또 복잡한 명령을 위헤 playbook 파일에 YAML 형식으로 사용하며 기본 구성으로 name, hosts, tasks 로 구성
- YAML : 가시성이 좋은 문법구조로 쉽고 단순하다 그러나 들여쓰기에 민감하다
- 변수 : 코드의 재사용성을 높이고 사용자의 효율성, 편의성을 위해서 사용한다. 명령어 사용 시 옵션으로 지정, 플레이북에 지정, 인벤토리에서 지정, 별도의 파일에서 지정 등의 방식이 있다.
 

