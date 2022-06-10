# 변수
- 재사용성과 효율성을 높이기 위해 변수를 설정 한다.
- 전체 범위의 변수를 설정하기 위해선 명령어에서 -e 옵션ㄴ으로 변수 설정한다.
- 플레이북에서 vars 혹은 vars_files로 설정한다(플레이 변수)
- 인벤토리 또는 host_var / group_vars로 설정한다.

* 명령어를 통한 전역 변수 사용
![화면 캡처 2022-06-09 115821](https://user-images.githubusercontent.com/57117748/172803329-f9d1dfbf-3647-4046-a74d-ae3124e8198c.png)

* 인벤토리 내에서 설정한 호스트/그룹 변수
![화면 캡처 2022-06-09 120116_LI](https://user-images.githubusercontent.com/57117748/172803653-0b3842cc-5b66-4e21-a781-73bfea921ee3.jpg)

* 배열변수
![화면 캡처 2022-06-09 121730](https://user-images.githubusercontent.com/57117748/172803894-1788facc-eb30-497a-8070-3351b25a97c6.png)


# 팩트
- 각 호스트의 고유 정보를 변수로 저장
- setup을 통해 자동으로 ansible에서 검색
- gether_facts: no/false 를 통해 비활성 가능(ansible 성능 증가)

* gether_facts: no
![화면 캡처 2022-06-09 122800](https://user-images.githubusercontent.com/57117748/172804120-dc6be379-be2f-455b-8b9e-f392b4177592.png)


# 작업제어
- 조건문, 반복문, 핸들러를 이용해 playbook의 작업 효율을 증가시킨다.

## 조건문(when)
- when 구문을 사용하는 조건문
- 해당 모듈에 대한 플에이의 실행 여부 결정한느 조건
- 모듈과 같은 수준의 들여쓰기 사용
- 논리연산자(and, or) 사용이 가능
 
* 기본 문법 사용
```
---
- name: condition using
  hosts: localhost
  vars:
  	- vars01: 1
  tasks:
  	- name: task one
  	  debug:
  	  	msg: task one clear
  	  when: "vars01 == 1"
  	- name: task two
  	  debug:
  	  	msg: task two clear
  	  when: "vars01 == 0"
#	  when : true
#	  when: ansible_facts.['hostname'] == 'host1'	// 해당 호스트들에서 팩트 호스트이름의 정보가 host1 일떄만 실행
...
```
![화면 캡처 2022-06-09 142015](https://user-images.githubusercontent.com/57117748/172808246-de8a5a3e-0fc1-452e-a5c9-441f54f799a0.png)

![화면 캡처 2022-06-09 142039](https://user-images.githubusercontent.com/57117748/172808250-a43394fa-3636-4dda-a45f-c7a11b123496.png)


## 조건문 연습
1. 호스트명에 따라 host1 또는 host3 일 때 사용자 생성
	- ansible_facts.hostname 으로 호스트 이름 확인 가능
```
---
- name: condition practice
  hosts: all
  tasks:
    - name: make user
      user:
        name: condition_practice
      when: ansible_facts.hostname == "host1" or ansible_facts.hostname == "host3"
...                                           
```
![화면 캡처 2022-06-09 151241](https://user-images.githubusercontent.com/57117748/172816267-24a67045-d5e1-46ee-8ddd-b406588ea146.png)

![화면 캡처 2022-06-09 151301](https://user-images.githubusercontent.com/57117748/172816279-50186407-da97-43d9-9e20-1ee44dcb73a0.png)

2. 메모리 크기가 1024 이상인 경우 /etc/hosts 파일을 /tmp 로 복사
	- ansible_facts.memfree_mb 로 메모리 확인
```
---
- name: condition practice2
  hosts: all
  tasks:
    - name: copy practice
      copy:
        src: /etc/hosts
        dest: /tmp
      when: ansible_facts.memfree_mb >= 1024
...
```
![화면 캡처 2022-06-09 171103](https://user-images.githubusercontent.com/57117748/172816933-d6919990-3abc-4717-9138-c436e482525c.png)

![화면 캡처 2022-06-09 171119](https://user-images.githubusercontent.com/57117748/172816936-507daff1-e351-43a1-bcc2-4f13a972f801.png)

## 반복문(loop)
- loop 구문을 사용하는 반복문
- 모듈의 반복실행 
- 반복적인 작업에 효율성을 위한 반복문
- item 이라는 반복문 전용 변수를 사용함
- 배열의 형태로 변수 사용 가능
- 중첩이 가능하다

* 반복문의 기본사용
```
---
- name: test loop
  hosts: localhost
  tasks:
  	- name: use debug
  	  debug:
  	  	msg: "{{item}}"
  	  loop:
  	  	- var01
  	  	- var02
...
```
![화면 캡처 2022-06-09 180403_LI](https://user-images.githubusercontent.com/57117748/172811566-cdb10c84-1671-4ca5-97e5-a8ee30e44137.jpg)


* 반복문의 배열변수 활용
```
---
- name: loop test with array var
  hosts: localhost
  vars:
    array_vars:
      name: testuser3
      groups: study

  tasks:
    - name: add several users
      user:
        name: "{{ item.name }}"
        state: present
        groups: "{{ item.groups }}"
      loop:
        - {name: 'testuser1', groups: 'wheel'}
        - {name: 'testuser2', groups: 'root'}
        - "{{ array_vars }}"
...
```
![화면 캡처 2022-06-09 182801](https://user-images.githubusercontent.com/57117748/172815120-ed9b81a5-6cfc-453c-abac-d6ebac8bf713.png)

![화면 캡처 2022-06-09 183007](https://user-images.githubusercontent.com/57117748/172815121-78ca12f5-8597-4576-bd5c-70aaae3d4bb5.png)

## 반복문 연습
1. httpd , mod_ssl , mod_wsgi 패키지를 설치하세요
```
---
- name: loop practice
  hosts: localhost
  tasks:
    - name: use the loop
      yum:
        name: "{{ item }}"
        state: latest
      loop:
        - httpd
        - mod_ssl
        - mod_wsgi
...
```
![화면 캡처 2022-06-09 152328](https://user-images.githubusercontent.com/57117748/172817215-ce14c96a-d114-4c1e-8d01-39a6d2df2cc7.png)

![화면 캡처 2022-06-09 152340](https://user-images.githubusercontent.com/57117748/172817217-a48120b4-3218-4572-9221-b0e42b0ec4a8.png)

2. 다음 조건의 사용자들을 만드세요.
	- 이름 : loop_user01 , UID : 1455
	- 이름 : loop_user02 , UID : 1456
	- 이름 : loop_user03 , UID : 1457
```
---
- name: loop practice2
  hosts: localhost
  tasks:
  	- name: make user loop
  	  user:
  	  	name: "{{ item.name }}"
  	  	uid: "{{ item.uid }}"
  	  loop:
  	  	- {name: 'loop_user01', uid: '1455' }
  	  	- {name: 'loop_user02', uid: '1456' }
  	  	- {name: 'loop_user03', uid: '1457' }
...
```
![화면 캡처 2022-06-09 152903](https://user-images.githubusercontent.com/57117748/172817965-58d2aa41-8432-4426-a9b5-34a161e25d46.png)

![화면 캡처 2022-06-09 152917](https://user-images.githubusercontent.com/57117748/172817971-525afc21-e11d-4f8c-b14a-3cb12e9105b6.png)

## Handlers
- 특정 작업의 결과가 changed 일 때 해당 해당 구문 실행
- 서비스의 재시작, 재부팅 등에 이용
- notify 구문으로 핸들러를 호출
- 핸들러의 이름을 통해 구문(이름이 겹치면 안됌)
- tasks 가 끝나고 핸들러가 실행
- handler를 호출시 항상 handler 섹션에서 지정한 순서대로 작동(2를 호출시 1,2/ 3을 호출시 1,2,3)

* 핸들러 기본 사용
```
---
- name: test handlers
  hosts: host1
  tasks:
  	- name: copy to config file
  	  copy:
  	  	src: index.html
  	  	dest: /var/www/html/
  	  notify: service_restart
  handlers:
  	- name: service_restart
  	  service:
  	  	name: httpd
  	  	state: restarted
...
```
![화면 캡처 2022-06-09 155835](https://user-images.githubusercontent.com/57117748/172820019-a921db5f-48be-4532-86ad-c8180dd0ab70.png)


## 예외처리
- ansible은 작업을 실시 했을때 반환 코드를 통해 평가하는데 리턴값이 0 반환시 실패 1 반환시 성공

1. ignore_errors : 오류 발생 시 건너뛰기
```
---
- name: test ignore_errors
  hosts:
  tasks:
  	- debug:
  		msg: abcd
  	- debug:
  		msg: "{{ msg }}"
  	  ignore_errors: yes
  	- debug:
  		msg: efg
...
```
![화면 캡처 2022-06-09 162114](https://user-images.githubusercontent.com/57117748/172821147-558e6ff2-4d33-47cc-9561-c6929a0e0889.png)

![화면 캡처 2022-06-09 162144](https://user-images.githubusercontent.com/57117748/172821195-3043494e-50f7-461d-a004-ad6327902f6d.png)

2. failed_when / chaged_when : 상태에 대한 별도 조건 지정
![화면 캡처 2022-06-09 162538](https://user-images.githubusercontent.com/57117748/172821698-361a9562-4554-472d-a158-fce16dc93386.png)


3. force_handlers : 작업 중 오류가 발생해도 호출한 핸들러는 꼭 실행하도록 하는 구문
```
---
- name:
  hosts: localhost
  tasks:
  	- debug:
  		msg: abcd
  	  failed_when: true
...
```
![화면 캡처 2022-06-09 162756](https://user-images.githubusercontent.com/57117748/172822015-fa411e1c-c480-46bc-87e7-21ae091fe964.png)

![화면 캡처 2022-06-09 162809](https://user-images.githubusercontent.com/57117748/172822046-b274b0ba-36f0-4b72-b202-4c016c1210fe.png)

## 블록처리
- 여러 모듈을 하나의 논리적인 작업으로 분류 시킨것이 block
- rescue: block이 실패했을때 실행
- always: block의 성공여부와는 상관없이 무조건 실행

![화면 캡처 2022-06-09 164159_LI](https://user-images.githubusercontent.com/57117748/172823181-d597ea69-754f-48df-8da7-cc1d79d9dc0e.jpg)

# 실습
1. 팩트를 사용해서 조건식을 만들기
	- copy 모듈로 /etc/hosts 파일을 복사하기, 전체 대상 중 host2는 제외
```
---
- name: practice 
  hosts: all
  tasks:
  	- name: copy /etc/hosts
  	  copy:
  	  	src: /etc/hosts
  	  	dest: /etc/hhosts
  	  when: ansible_facts.hostname != "host2"
...
```
![화면 캡처 2022-06-09 231209](https://user-images.githubusercontent.com/57117748/172871520-e112c033-448d-4390-9382-09c9d716d062.png)


2. 운영체제 종류 확인 후 yum 모듈 사용하기 전체 대상 중 CentOS 종류이면서 버전이 "7" 인 경우 targetcli 패키지 설치해보기
	- ansible_distribution : 종류확인
	- ansible_distribution_major_version : 버전확인

```
---
- name: practice 
  hosts: all
  tasks:
  	- name: yum targetcli for centOS7
  	  yum:
  	  	name: targetcli
  	  	state: latest
  	  when: ansible_distribution == "CentOS" and ansible_distribution_major_version == "7"
...
```
![화면 캡처 2022-06-09 231220](https://user-images.githubusercontent.com/57117748/172871410-aa98a22f-5805-4bc8-82e4-1951f59b1121.png)

![화면 캡처 2022-06-09 232704](https://user-images.githubusercontent.com/57117748/172871413-8eedba58-ed4a-4ec5-a6f1-2f68219c56e5.png)

3. 변수 host_vars / group_vars 로 설정하기
	- invantory
```
[web]
host[1:2]

[db]
host3
host4

[servers:children]
web
db
```
	- 각 호스트 별로 os_type 이라는 변수 설정(host{1..3} = : centos / host4 = ubuntu)
	- 각 그룹 별로 service_type 이라는 변수 설정(web = webserver / db = database)

4. 3을 이용한 변수 사용
	- os_type 이 centos 인 경우에만 file 모듈로 /etc/ansible/facts.d 디렉토리 생성하기
```
---
- name: vars practice
  hosts: all
  tasks:
    - block:
      - name: make facts.d dir with file
        file:
          path: /etc/ansible/facts.d
          state: directory
	  recurse: yes
      when: os_type == "centos"
...
```
![화면 캡처 2022-06-10 012251](https://user-images.githubusercontent.com/57117748/172896652-aa481362-f1ae-467a-be73-c41a8dd0f840.png)

![화면 캡처 2022-06-10 012303](https://user-images.githubusercontent.com/57117748/172896661-04f61485-e3b2-4fc5-b0c1-59a3bd4d33af.png)
	- service_type 이 database 인 경우에만 copy 모듈로 /etc/ssh/sshd_config 파일을 /tmp 로 복사하기
```
---
- name: vars practice2
  hosts: all
  tasks:
  - name: copy /etc/ssh/sshd_config to /tmp
    copy:
      src: /etc/ssh/sshd_config
      dest: /tmp
    when: service_type == "database"
...
```

* 실행시 vagrant 사용자는 sshd_config 파일에 대한 권한이 없다 그래서 ansible.cfg 구성파일에서 ask_pass 항목을 yes로 하고 실행

![화면 캡처 2022-06-10 013016](https://user-images.githubusercontent.com/57117748/172898075-adedc39a-adbf-4c0b-8d2d-5ffa9159ba0b.png)

![화면 캡처 2022-06-10 013100](https://user-images.githubusercontent.com/57117748/172898103-e6e5272d-43ef-41c8-bd46-d1f6edd4c813.png)

5. 핸들러 쓰기
	- lineinfile 모듈로 설정파일 수정 /etc/httpd/conf/httpd.conf 파일에서 Listen 80 -> Listen 192.168.56.110:80
	- 위 작업(lineinfile모듈)이 실행되면 restart service 라는 핸들러 호출하기
	- restart service 핸들러에서는 service 모듈로 서비스 재시작. 서비스 이름은 svc_name 이라는 변수로 지정하기.
```
---
- name: practice3
  hosts: all
  vars:
    svc_name: httpd
  tasks:
    - name: modify httpd.conf
      lineinfile:
        path: /etc/httpd/conf/httpd.conf
        regexp: ^Listen
        line: Listen 192.168.56.110:80
      notify: restart service
  handlers:
    - name: restart service
      service:
        name: "{{ svc_name }}"
        state: restarted
...
```
![화면 캡처 2022-06-10 014602](https://user-images.githubusercontent.com/57117748/172901046-e456ae9a-e46c-4bfe-9a9c-937400106e38.png)
