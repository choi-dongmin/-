# Ansible
- 코드형 인프라(IaC)를 위한 도구
- IaC : 코드를 이용해 인프라환경을 구성/관리하는 인프라 관리 자동화 도구.
- IaC의 필요성 : 과거에는 기존의 유지관리 방식으로 가능했지만 현대에 오면서 그 규모가 커지고 복잡해지면서 자동화를 지향하게 됨

## IaC의 장점
- 비용 및 시간의 절감
- 오류 및 보안 정책 위반 등의 위험요인 제거
- 텍스트 파일로 표현 시 버전 관리가 쉬움
- 자동으로 빌드/리빌드 가능
* 가상 시스템에 대한 프로그래밍 방식의 관리지원 = 개별 하드웨어를 수동으로 구성하고 업데이트할 필요가 없어짐 = 인프라에 "유연성”, 즉 반복성과 확장성을 부여

## IaC의 한계점
- 할 수 있는 일이 제한적이다
- 구조적 코딩 프로젝트에 비해 스크립트 작성은 느슨하고 약식으로 수행한다.
- 멱등성(Idempotence)의 부재

# Vagrant
- IaC의 도구중 하나로 Ansible이 지원하지 않는 템플릿 구조를 만드는것을 지원한다.

```
> vagrant up : vagrant 실행
> vagrant halt : vagrant 종료
> vagrant destroy : vagrant 삭제
```
# Ansible
- 오픈소스의 IT 자동화 도구이다
- 시스템 구성 및 배포를 제공한다
- 단순성과 사용의 용의성이 크다
- 보안과 신뢰성이 높다
- YAML 언어 사용으로 접근성이 좋다

## Ansible 장점
- SSH 를 이용해 Agentless 이다
- YAML 언어 사용으로 쉽고 접근성이 높다
- 멱등성이 있다
- 변수를 이용해 재사용성이 높다
- 다른 도구보다 간소화 되어 있다
- 다양한 플랫폼을 지원한다.(단 Control-loder은 리눅스만 지원)

## Ansible 단점
- 타 도구들에 비해 전문성이 부족하다
- DSL을 통한 로직 수행(수시로 문서를 확인해야 한다)
- 변수 등록으로 인한 복잡성
- 변수값 확인은 출력으로만 가능
- 입/출력, 구성파일에 일관성이 없다
- 때떄로 성능의 저하 확인

## 구성 
- YAML(Play book)으로 구성된 텍스트 파일
- 하나 이상의 작업내용(Play)의 집합

## 동작 
- 각 작업은 특정 인수와 함게 작은 코드 조각 모듈을 실행
- 시스템 특정 측면이 아닌 특정한 상태에 있는지 확인
- 특정한 상태가 아니면 상태 구성을 위해 다시 실행
- 작업 실패시 후에 구성된 다른 작업들도 중단 시킴
- 별도의 에이전트 없이 SSH를 통해 실행 가능

## 제어노드 / 관리노드
1. 제어노드
	- 설치 및 실행
	- 원격으로 관리하는 노드 제어
	- 프로젝트 파일의 사본 보관
	- Python v2.6 이상

2. 관리노드
	- 제어노드에 의해 모듈 설치 및 관리 실행
	- 인벤토리에 나열된 대상
	- 단독 혹은 그룹으로 관리 가능
	- Python v2.4 이상
	- SSH 필요


# Vagrant를 통한 VM 배포
1. Vagrant download
	- 홈페이지에서 Vagrant 설치 후 시스템 재부팅
	[Vagrant](https://www.vagrantup.com/)

2. Vagrant file 구성
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "control-node" do |config|
    config.vm.box = "centos/7"
    config.vm.network "private_network", ip: "192.168.56.100"
    config.vm.hostname = "control-node"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.name = "control-node"
    end
  end

  config.vm.define "host1" do |config|
    config.vm.box = "centos/7"
    config.vm.network "private_network", ip: "192.168.56.110"
    config.vm.hostname = "host1"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "host1"
    end
  end

  config.vm.define "host2" do |config|
    config.vm.box = "centos/7"
    config.vm.network "private_network", ip: "192.168.56.120"
    config.vm.hostname = "host2"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "host2"
    end
  end

  config.vm.define "host3" do |config|
    config.vm.box = "centos/8"
    config.vm.network "private_network", ip: "192.168.56.130"
    config.vm.hostname = "host3"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "host3"
    end
  end

  config.vm.define "host4" do |config|
    config.vm.box = "ubuntu/focal64"
    config.vm.network "private_network", ip: "192.168.56.140"
    config.vm.hostname = "host4"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "host4"
    end
  end

  # Hostmanager plugin
  config.hostmanager.enabled = true
  config.hostmanager.manage_guest = true

end

```

- Ruby 로 이루어진 Vagrant VM 구성 파일
- ||의 안쪽의 글자는 다음 행들의 이름이 된다.
- control-node와 host1~4 까지 만든다
- 각 VM 시스템 구성, OS 및 네트워크를 지정 


2. plugin 설치
```
> vagrant  plugin  install  vagrant-hostmanager
```

3. 가상머신 배포
```
> vagrant  up Vagrantfile		// Vagrant File의 경로에서 vagrant up을 통해 VM을 배포한다
```

# Ansible 사용 

1. 호스트에서 제어노드에 원격 접속
```
> vagrant ssh control-node 		// Vagrant의 경로에서 Vagrant File에서 지정한 제어노드에 ssh를 통해 진입 
```
![화면 캡처 2022-06-07 123426](https://user-images.githubusercontent.com/57117748/172344449-e225798c-cd5a-4820-ae1a-f562b29d1841.png)


2. 패키지 설치 (제어노드)
```
$ sudo yum install epel-release -y
$ sudo yum install -y ansible
``` 
![화면 캡처 2022-06-07 124239](https://user-images.githubusercontent.com/57117748/172344542-d4c23756-0fe4-46b2-ae2b-b3cce352aac5.png)


3. ssh 접속 설정 (모든 시스템)
```
$  vi /etc/ssh/sshd_config
```
```
> PasswordAuthentication yes
```
```
$ sudo systemctl restart sshd
```

![화면 캡처 2022-06-07 140336](https://user-images.githubusercontent.com/57117748/172344960-1c69efd5-0512-4e86-8aa6-c701277a7fb3.png)
![화면 캡처 2022-06-07 140429](https://user-images.githubusercontent.com/57117748/172345007-7ccb03d7-19b3-4d59-8474-17cd7c971944.png)

4. 관리 대상(호스트)주소를 등록 (제어노드)
	- 기존 파일에 내용이 없을시에 추가
```
$ sudo vi /etc/hosts	
```
```
192.168.56.100  control-node
192.168.56.110  host1
192.168.56.120  host2
192.168.56.130  host3
192.168.56.140  host4
```

5. 관리 사용자 설정
	- 사용자 생성(모든 시스템)
	```
	$ sudo useradd ansible
	$ sudo passwd ansible
	```

	- sudo 설정 추가(모든 시스템)
	```
	$ sudo vi /etc/sudoers.d/ansible
	```
	```
	ansible ALL=(ALL) NOPASSWD: ALL
	```

	- ssh 키 설정(제어 노드)
	```
	$ ssh-keygen -t rsa -b 2048
	$ ssh-copy-id ansible@hosts	// host1~4 각각의 호스트 마다 복사
	```

6. ubuntu 설정
	- host4(ubuntu) 에서는 자동으로 home 디렉토리에 계정 디렉토리가 만들어 지지 않아 직접 만들어주고 권한을 지정해야 한다.
	- Permission denie 오류
	```
	ansible@host4's password:
	Could not chdir to home directory /home/ansible: No such file or directory
	sh: 1: cd: can't cd to /home/ansible
	mkdir: cannot create directory ‘.ssh’: Permission denie
	```
	- 해결 (host4 또는 다른 ubuntu OS에서)
	```
	$ sudo mkdir /home/ansible
	$ sudo chown ansible:ansible /home/ansible
	$ sudo chmod 755 /home/ansible
	```

7. 전체 확인(제어노드)
```
$ ssh ansible@host1 "sudo id"
$ ssh ansible@host2 "sudo id"
$ ssh ansible@host3 "sudo id"
$ ssh ansible@host4 "sudo id"
```
![화면 캡처 2022-06-07 150843](https://user-images.githubusercontent.com/57117748/172347992-a8f2cf95-6237-40b8-bb6a-99feb0739367.png)


## 인벤토리 파일
- ansible을 통해 작업할 대상(호스트)를 지정하는 파일
- 호스트네임/IP주소를 이용해 설정
- 각각 단일 호스트 혹은 그룹으로 묶어서 설정
- 작업 실행시 해당 호스트 또는 그룹으로 작업 대상 지정
- INI / YAML  형식으로 사용
- /etc/ansible/hosts 파일이 기본
- 따로 만들경우 ansible.cfg(구성파일)에 경로를 지정
- Inventory 파일 예시
	```
	host1

	[web]
	host2
	host3

	[db]
	host4

	[centos]
	host1
	host2
	host3

	[ubuntu]
	host4

	[seoul]
	host1
	host2

	[busan]
	host3
	host4

	[korea:children]
	seoul
	busan
	```
	![화면 캡처 2022-06-07 184750](https://user-images.githubusercontent.com/57117748/172350648-f18b73b2-85ed-4271-9cac-c0b84e03e218.png)
	![화면 캡처 2022-06-07 184735](https://user-images.githubusercontent.com/57117748/172350655-cf7c30b9-6c27-4673-a2be-ba2ae41fbd80.png)


## 구성파일(ansible.cfg)
- 기본적으로는 /etc/ansible/ansible.cfg 파일 사용(낮은 우선순위)
- 현재 작업디렉토리에 ansible.cfg 파일이 있으면 해당 파일 사용(높은 우선순위)
- 구성파일 옵션
	1. remote_user : 접속할 사용자 이름
	2. ask_pass : 접속 시 패스워드 입력 유무 
	3. become : 접속 후 권한 상승 여무	// root 사용자 권한 상승
	4. become_ask_pass : 권한 상승 시 패스워드 입력 유무
	5. inventory : 인벤토리파일 경로 지정
- 구성파일 예시
	```
	[defaults]
	inventory = inventory
	remote_user = ansible
	ask_pass = false

	[privilege_escalation]
	become = true
	become_ask_pass = true
	```
	![화면 캡처 2022-06-07 185414](https://user-images.githubusercontent.com/57117748/172352016-857ef4b6-630c-4d5e-80de-59f7972cfeb4.png)


# 연습
1. /etc/ansible/hosts 파일의 내용을 수정해보기
```	
$ sudo vi /etc/ansible/hosts
```
```
host1
host2
```
![화면 캡처 2022-06-07 205852](https://user-images.githubusercontent.com/57117748/172373780-b578d22f-0622-4c6c-a123-251726d0c582.png)

2. --list-hosts 옵션으로 확인해보기
![화면 캡처 2022-06-07 210259](https://user-images.githubusercontent.com/57117748/172374449-93ca3b6f-43c7-4f44-9e22-0de1e8562d9f.png)

3. /etc/ansible/hosts 파일의 내용을 수정
	-  host1 ~ host4 를 한줄로 설정해보기
	![화면 캡처 2022-06-07 210750](https://user-images.githubusercontent.com/57117748/172375316-a7f0d1db-78af-4161-93af-60b93363841c.png)

	- webserver 라는 그룹으로 host1, host2 만 설정하기
	![화면 캡처 2022-06-07 210758](https://user-images.githubusercontent.com/57117748/172375359-4dbf7cea-bd35-478e-baf2-73d6a6f69d7f.png)

	- application 라는 그룹으로 host3 설정 추가
	![화면 캡처 2022-06-07 210806](https://user-images.githubusercontent.com/57117748/172375384-d7808905-ee3a-45ec-9844-f251921332e4.png)

	- database 라는 그룹으로 host4 설정 추가
	![화면 캡처 2022-06-07 210815](https://user-images.githubusercontent.com/57117748/172375403-a4a66e14-cdeb-436d-808b-da322030ec27.png)

	- was 라는 상위 그룹의 구성원으로 webserver, application, database 그룹들 설정하기
	![화면 캡처 2022-06-07 210940](https://user-images.githubusercontent.com/57117748/172375518-907c5fd6-d857-4a47-b197-51b17982337b.png)

4. 구성파일 작성 연습
	- defaults 섹션에서 remote_user 항목을 vagrant 로 설정, ask_pass 항목을 true 로 설정
	![화면 캡처 2022-06-07 212420](https://user-images.githubusercontent.com/57117748/172378043-7e7b63b8-7d29-47fb-9e9c-1d0f2230d276.png)
	![화면 캡처 2022-06-07 212405](https://user-images.githubusercontent.com/57117748/172378065-52ae66a2-585a-401d-b4bf-3d03163edd90.png)

	- ansible -m command -a id host1 명령어로 동작 확인
	![화면 캡처 2022-06-07 212711](https://user-images.githubusercontent.com/57117748/172378547-5826c95c-f31c-4c75-83a4-b44a5ddb1ee0.png)

	- 권한상승 설정 추가 /etc/ansible/ansible.cfg 파일에서 privilege_escalation 에서 become 을 true 로 become_ask_pass를 true 로 설정 한 후 다시 확인
	![화면 캡처 2022-06-07 212950](https://user-images.githubusercontent.com/57117748/172379067-6406db93-36cd-4b3f-9626-ff6bbecf0252.png)
	![화면 캡처 2022-06-07 213123](https://user-images.githubusercontent.com/57117748/172379333-8653ea42-a8fe-4821-b7b5-d6c69aaf1697.png)


5. 별도의 인벤토리 파일 작성
	- 작업디렉토리 생성 (first_demo)

	- 해당 디렉토리 안에 inventory 라는 이름으로 파일 생성
	![화면 캡처 2022-06-07 211325](https://user-images.githubusercontent.com/57117748/172376146-f114687c-8460-44f2-b5c8-8734fa5a5d34.png)

	- web 그룹은 host1 ~ host3
	![화면 캡처 2022-06-07 211559](https://user-images.githubusercontent.com/57117748/172376557-878a951b-cf06-4b2b-ae2e-ff730bb4510d.png)

	- db 그룹은 host4
	![화면 캡처 2022-06-07 211616](https://user-images.githubusercontent.com/57117748/172376620-edb1051f-7b98-4d58-af3d-8a3460274b43.png)

	- servers 상위그룹은 web/db 그룹 포함
	![화면 캡처 2022-06-07 211631](https://user-images.githubusercontent.com/57117748/172376683-8edec349-5e48-4a2b-8394-8ac4c19d8dcb.png)

6. 별도로 구성파일 생성
	-  3번 연습에서 사용한 동일한 작업디렉토리에 ansible.cfg 파일 생성

	- 인벤토리파일을 해당 디렉토리의 파일로 지정해보기
	![화면 캡처 2022-06-07 213903](https://user-images.githubusercontent.com/57117748/172381369-84f0f64b-8136-4471-974c-0b9c3bd617f9.png)
	![화면 캡처 2022-06-07 213824](https://user-images.githubusercontent.com/57117748/172381269-b051e0f7-38b9-49ac-ab41-4a45e9ba4bd0.png)

	- ansible 사용자로 접속 시 패스워드 입력 없이 작업 가능하게 설정해보기
	![화면 캡처 2022-06-07 214200](https://user-images.githubusercontent.com/57117748/172381442-8cf5f980-2004-4dd4-8850-f4ba860b664b.png)
	![화면 캡처 2022-06-07 214105](https://user-images.githubusercontent.com/57117748/172381501-7f505785-5bb6-4f25-a201-fb2ab4d0c2f6.png)

	- 작업 시 권한상승 후 root 사용자로 작업하도록 설정 추가, 권한 상승 시 패스워드 입력 없이 가능하도록 설정 추가
	![화면 캡처 2022-06-07 214329](https://user-images.githubusercontent.com/57117748/172381989-0c100037-c3d1-4ac2-969e-2aafc301e1da.png)
	![화면 캡처 2022-06-07 214408](https://user-images.githubusercontent.com/57117748/172382000-cdadf366-bdb1-47dd-9c0d-66199fa24ead.png)


## 키워드
- IaC : Infrastructer as a Code 코드형 인프라환경 구성/배포/관리, 인프라 환경이 복잡해지고 커지면서 자동화 즉 코드형 인프라가 대두 되었다. 
- Ansible : IaC 도구 중 하나로 오픈소스이며 YAML 언어를 사용해 가독성이 뛰어나다. 호스트에게 따로 설치할 필요가 없는 Agentless 형식이다. 멱등성을 보장한다 즉, 상태를 지정하면 그에 맞게 설정한다 그로인해 여러번 실행해도 결과가 동일하다.
- 제어노드 : 제어노드에서 ansible 을 패키지 형식으로 설치한다 Inventory에 호스트의 리스트를 만들어서 사용한다. 호스트는 모든 OS가 가능하지만 제어노드의 OS는 리눅스만 가능하다.
- 관리노드(호스트) : 호스트는 기본적인 OS SSH Python2.4 이상의 기본 구성이 있어야 한다.
- Inventory : 관리할 대상의 IP/호스트네임의 리스트 파일이다. YAML or INI 텍스트 형식이며 단일/그룹 단위의 호스트 지정 할 수 있다. 기본 경로는 /etc/ansible/hosts 이며 보통은 다른 경로의 파일로 생성해 구성파일에서 경로를 지정한다.
- 구성파일(ansible.cfg) : Ansible 작업 실행 시 필요한 설정, 구성을 저장하는 파일이다. 접속할 사용자, 권한, 인벤토리 등을 설정할 수 있으며 기본경로는 /etc/ansible/ansible.cfg이며 보통은 우선순위가 높은 ./ansible.cfg 로 만들어 관리한다
