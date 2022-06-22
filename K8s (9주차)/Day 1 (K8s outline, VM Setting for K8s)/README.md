# 기본개념
- 컨테이너 : 애플리케이션들이 분리된 공간에서 리소스를 공유하는 시스템 그러나 리소스를 많이 사용하지 못하도록 제한한다
- Dokcer : 한 작업에 1개 노드가 1개의 컨테이너만 관리하는 플랫폼
- Dokcer Compose : 한 작업에 1개의 노드가 2개 이상의 컨테이너를 관리하는 플랫폼 
- K8s : 작업을 할 때 다수의 노드가 다수의 컨테이너를 관리하는 플랫폼

# 모놀리식 아키텍처
- 1개의 환경에서 모든 App이 실행되는 전통적인 통합 모델, 애플리케이션의 모든 기능이 하나로 통합 구현된 형태의 아키텍처 
- 오늘날에도 널리 사용되는 아키텍처로 대부분 단일 프로세스에서 실행되거나 몇몇 시스템에서 몇개의 프로세스로 실행됨

## 모놀리식 아키텍처 장/단점
1. 장점
	- 간단한 개발 및 배포 
	- 단순한 방식의 확장성
	- End to End 테스트가 용이(MSA의 경우 모듈을 모두 실행 해야함)

2. 단점
	- 코드의 품질 저하
	- 애플리케이션 속도의 저하
	- 애플리케이션 지속적 배포의 어려움(유지보수)
	- 애플리케이션 확장의 어려움
	- 컴포넌트별 개발의 어려움
	- 다양한 기술 적용의 어려움

## 모놀리식 아키텍처 결론
- 모놀리식 애플리케이션은 모든 것이 서로 강하게 밀 결합해 구성되어 단일 OS의 단일 프로세스로 실행되기 때문에 모든 것을 하나의 애플리케이션으로 개발 배치 관리 되어야 함
- 애플리케이션에 조그마한 추가 및 변경에서 전체를 재배포해야 하며 장기적으로 , 시스템이 점점 복잡해져서 결국 전체적인 품질 저하가 일어날 수 음

# 마이크로서비스 아키텍처
- 모놀리식 아키텍처가 크기가 커짐에 따라 발생하는 문제점을 극복하기 위해 마이크로서비스 라는 (Microservice) 기능적으로 세분화되고 독립적으로 작동하 는 방식을 사용.
- 세분화되고 독립적으로 작동하는 마이크로서비스 기반의 애플리케이션은 API를 통해 서로 다른 마이크로서비스와 통신하게 됨
- 여러 마이크로서비스 사이에는 일반적으로 동기 방식인 HTTP/RESTful API 또는 비동기 방식인 AMQP(Advanced Message Queue Protocol) 프로토콜을 이용하여 통신한다 때문에 . 각 마이크로서비스는 각 기능을 구현하기 가장 적합한 언어로 개발할 수 있음

## 마이크로서비스 아키텍처 장/단점
1. 장점
	- 지속적인 배포에 유리
	- 향상된 유지 보수성 각 서비스는 작기 때문에 이해하고 변경하기 쉬움
	- 테스트 용이성 각 서비스는 독립적으로 테스트 가능
	- 배포 효율성 각 서비스는 독립적으로 배포 가능
	- 독립적으로 개발 테스트 배포 및 확장
	- 개발에 생산성이 높고 배포 속도가 높음
	- 향상된 장애 격리(발생한 서비스에서만 영향을 받음)
	- 다양한 기술적용 가능

2. 단점
	- 분산시스템 설계에 따른 복잡성
	- 서비스 간 통신 메커니즘을 따로 구현
	- 서비스 간 상호작용 테스트(End to End 테스트)
	- 배포 및 관리 운영상의 복잡성
	- 증가된 리소스 소비 

# DevOps
- 애플리케이션을 개발 운영하는 경우 개발, 품질, 운영 등 전반에 걸친 통합을 통헤 환경, 기불, 접근방식 등을 의미
- 개발 팀에서 애플리케이션을 개발하는 환경과 운영 팀에서 개발 된 애플리케이션을 프로덕션 시스템에 배포해 운영하는 환경과의 차이 때문에 문제가 발생
- 기존보다 더 빠르게 제품을 혁신하고 관리
- 개발/운영 사이에 단절되었던 역할이 협업 및 모호화(출시시간 단축, 복구시간 개선, 유연한 대응)


# Kubernetes
- 여러 컨테이너 오케스트레이션 도구 중 현재 가장 주목 받고 있는 도구는 쿠버네티스
- 구글에 의해 개발된 컨테이너 오케스트레이션 도구
- 쿠버네티스는 기본적으로 도커 플랫폼을 사용
* 오케스트레이션 : 다수의 시스템과 애플리케이션을 쉽게 설정하고 유지관리 할 수 있는 방식

## 쿠버네티스 제공하는 기능과 아닌 기능
1. 제공하는 기능
	- 컨테이너 플랫폼
	- 마이크로서비스 플랫폼
	- 이식성 있는 클라우드 플랫폼

2. 제공하지 않는 기능
	- CI/CD 파이프라인
	- 애플리케이션 레벨의 서비스
	- 로깅, 모니터링, 경고 솔루션


## K8s Architecture
- 쿠버네티스 클러스터는 여러 시스템의 연결로 구성
- 마스터 와 노드 구성요소로 분류할 수 있음
- 필요에 따라 추가 요소(애드온)가 있을 수 있음

## 마스터 
- 쿠버네티스 클러스터를 구성하기 위한 핵심 요소들의 모음
- 클러스터를 조정하기 위한 컨트롤 플레인을 제공함.
- 노드에 작업을 분배하는 스케쥴링 등 클러스터에서 전반적인 결정을 수행하고 대응이 필요한 클러스터 이벤트를 감지하고 이에 대응하는 역활을 수행
- 마스터의 기능 : API서버, etcd, 스케쥴러, 쿠버네티스 컨트롤러 관리자, 클라우드 컨트롤러 관리자

1. API 서버
	- API(Application Programming Interface)는 서로 다르게 구성되어 있는 기능들이 데이터를 주고 받기 위한 방식, 즉 인터페이스를 말함.
	- 쿠버네티스의 구성요소들은 마스터의 API 서버와 메세지를 주고 받게 되고 API 서버가 허브역활을 수행하여 모든 요청을 수신하며 명령을 전달하는 역활을 수행

2. etcd
	- 쿠버네티스 클러스터의 모든 정보 데이터를 저장하는 저장소
	- 데이터 저장시 'key - value' 형태로 데이터를 저장함.
	- 데이터 백업에 대하여 반드시 고려할 필요가 있음.

3. 스케쥴러(Scheduler)
	- 클러스터 내에서 생성되는 파드(컨테이너)를 감지하고 실행할 노드를 선택하는 역활을 수행하는 구성요소
	- 리소스 상태, 하드웨어/소프트웨어/정책상의 제약,  어피니티(Affinity) 등 다양한 기존에 따라 배치를 결정

4. 쿠버네티스 컨트롤러 관리자
	- API  서버를 통해 클러스터의 상태를 감시하고 , 필요한 상태를 유지하는 기능을 수행하는 구성요소
	- 종류
	1. 노드 컨트롤러 : 노드를 관리하며 노드가 다운되었을 때 알리고 대응
	2. 레플리케이션 컨트롤러 : 복제 컨트롤러를 사용하는 모든 오브젝트를 관리하며 알맞은 수의 파드를 유지하는 기능이 있음.
	3. 엔드포인트 컨트롤러 : 서비스와 파드를 연결
	4. 서비스 어카운트 및 토큰 컨트롤러 : 쿠버네티스의 네임스페이스, 계정, 토큰을 담당

5. 클라우드 컨트롤러 관리자
- 클라우드 환경에서 쿠버네티스 컨트롤러가 하는 역활을 수행함

## 노드(Worker Node , Minion Node)
- 쿠버네티스의 컨테이너가 동작하는 런타임 환경을 제공하며, 동작중인 파드를 유지하는 기능을 담당
-  노드의 역활 : 큐블릿, 프록시, 컨테이너 런타임
1. 큐블릿(kubelet) 
	- 쿠버네티스 클러스터의 각 노드에서 실행되는 에이전트로 마스터로부터 제공받는 파드의 구성 정보, 즉 노드가 수행하여야 할 작업을 전달받아서 컨테이너가 확실하게 동작하도록 보장하는 역활을 수행
2. 프록시 
	- 클러스터 내부 간 통신이나, 클러스터 외부에서 내부로 전달되는 통신 등 호스트 레벨의 네트워크 규칙을 구성하고 외부 연결을 파드에 포워딩하며 서비스의 추상화를 제공
3. 컨테이너 런타임
	- 각 노드에서 실제 컨테이너의 동작을 책임지는 구성요소(docker, containerd, CRI-O, rktlet, Kubernetes CRI)

## 애드온
- 쿠버네티스 클러스터에 추가할 수 있는 확장 기능을 제공하는 구성요소

1. 클러스터 DNS
	- 쿠버네티스 클러스터 내에 여러 오브젝트(파드, 컨테이너, 서비스 등)에 대한 DNS 레코드를 제공하는 구성요소
	- 주소 기반으로 오브젝트를 찾을 수 있음
	- 추가 확장 기능으로 분류되어 있지만 다른 애드온과는 다르게 거의 필수적인 애드온

2. 대시보드
	- 쿠버네티스 클러드터를 위한 웹 기반의 인터페이스를 제공하는 구성요소

3. 컨테이너 리소스 모니터링
	- 컨테이너들에 대한 리소스 사용량을 시계열 매트릭스를 사용해 데이터를 저장하고 열람하기 위한 인터페이스를 제공함

4 클러스터 로깅
	- 컨테이너 로그를 중앙 로그 저장소에 저장하고 관리하는 기능을 담당


## K8s API
- API Version : 같은 종류의 리소스라도 API 버전이 다른 경우 안정성이나 기술 지원의 수준의 차이가 존재함
- 같은 종류의 리소스라고 하더라도 서로 다른 API 버전이 존재할 수 있으며, API 버젼이 다르다는 것은 안정성이나 기술 지원의 수준이 다르다는 것을 의미

1. alpah
	- 이름에 alpha가 포함되어 앞으로도 소프트웨어 릴리스애서 예고없이 호환성이 없는 방법으로 변경될 가능성이 있는 버전 
	- 기본적으로 비활성화 되어있음
	- 다음 릴리즈 공지 없이 API 호환성이 깨지는 방식을 변경 될 수 있음
	- 버그의 위험이 높고 장기간 지원되지 않으므로 단기간 테스트 용도의 클러스터에서만 사용할길 권장함

2. beta
	- 이름에 beta가 포함되며 코드가 잘 테스트 되었고, 이 기능을 활성화해도 안전한 버전으로 기본적으로 활성화 되어 있음
	- 오브젝트에 대한 스키마나 문법이 다음 베타 또는 안정화 릴리스에서 호환되지 않을 수 있으나, 이전 가이드를 제공함
	- API 오브젝트의 삭제, 편집 또는 재성성이 필요할 수도 있으며 이런경우 애플리케이션의 다운타임이 필요할 수도 있음.

3. stable 
	- 버전 이름이 정수 타입의 버전으로 실제 운영 환경에서 사용할 수 있는 버전
	- 안정화 버전의 기능은 이후 여러 버전에 걸쳐서 소프트웨어 릴리스에 포함됨

## YAML 
- 기본적으로 Key:value 혹은 key : list 형태를 가짐
- UTF-8 / UTF-16 의 코드
- 공백 문자를 이용한 들여 쓰기로 계층 구조를 구분
- ---,... 시작과 끝에 첨부
- 확장자는 .yaml 혹은 .yml 
* 일반적으로 편의를 위해 vimre 설정 (~/.vimre)
```
﻿syntax on
autocmd FileType yaml setlocal ai ts=2 sw=2 sts=2 et autoindent
set cursorcolumn
```


## Manifest
- 쿠버네티스는 클러스터의 상태를 나타내기 위해 오브젝트 개체를 정의하여 사용
- 오브젝트르 생성할 때는 오브젝트의 기본정보, 오브젝트의 상세 스펙을 정확하게 제시 해야함.
- YAML 또는 JSON 문법으로 정의 할 수 있음.
- 오브젝트 정의시 필수 요구 필드 값 
	* apiVersion : 오브젝트를 생성하귀 위한 API 버전 지정	
	* kind : 오브젝트의 종류를 명시
	* metadata : name, label, namespace 등 기본적인 정보를 기술
	* spec : 오브젝터의 세부 상태를 정의

## 오브젝트 관리
1. 명령형 명령어
	- kubectl 명령어를 실행 시 오브젝트 생성에 필요한 정보를 인수 또는 옵션을 사용하여 전달하는 방식
2. 명령형 오브젝트 구성 
	- 오브젝트에 대한  세부 사항을 YAML/JSON 포맷으로 작성하고(메니페스트) , kubectl 명령은 작성된 파일을 참고하여 실행하는 방식
3. 선언형 오브젝트 구성 
	- 특정 디렉토리에 오브젝트 파일을 배치하는 방식, kubectl  명령은 특정 디렉토리에 배치된 오브젝트 파일을 참고하여 오브젝트를 관리

# VM 설치 및 K8s 사용을 위한 환경 설정
## Kubeadm
- 노드 하나하나에 직접 환경설정하는 방식

## kubespray
- ansible을 이용한 구축

0. vagrant 설치
![vagrant.com](https://www.vagrantup.com/downloads.html)

1. 플러그인 설치
- cmd 혹은 Pwoer Shell
```
vagrant plugin install vagrant-hostmanager  
vagrant plugin install vagrant-disksize
vagrant plugin list
vagrant box add ubuntu/bionic64	// 해당 Vagrnat File Nodes가 ubuntu 일때
vagrant box list
```

2. 특정 디렉토리 생성 후 이동
```
mkdir c:\kube
cd kube  
```

3. Vagrantfile 작성
- 예시
	1. 1개의 마스터 노드와 3개의 워크 노드
	2. 리소스와 네트워크 설정
	3. 가상머신은 버츄어 박스
	4. OS는 ubuntu
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.define "kube-master1" do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "kube-master1"
      vb.cpus = 2
      vb.memory = 3072
    end
    config.vm.hostname = "kube-master1"
    config.vm.network "private_network", ip: "192.168.56.11"
    config.disksize.size = "30GB"
  end
  config.vm.define "kube-node1" do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "kube-node1"
      vb.cpus = 2
      vb.memory = 3072
    end
    config.vm.hostname = "kube-node1"
    config.vm.network "private_network", ip: "192.168.56.21"
    config.disksize.size = "30GB"
  end
  config.vm.define "kube-node2" do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "kube-node2"
      vb.cpus = 2
      vb.memory = 3072
    end
    config.vm.hostname = "kube-node2"
    config.vm.network "private_network", ip: "192.168.56.22"
    config.disksize.size = "30GB"
  end
  config.vm.define "kube-node3" do |config|
    config.vm.box = "ubuntu/bionic64"
    config.vm.provider "virtualbox" do |vb|
     vb.name = "kube-node3"
      vb.cpus = 2
      vb.memory = 3072
    end
    config.vm.hostname = "kube-node3"
    config.vm.network "private_network", ip: "192.168.56.23"
    config.disksize.size = "30GB"
  end

  # Hostmanager plugin
  config.hostmanager.enabled = true
  config.hostmanager.manage_guest = true

  # Enable SSH Password Authentication
  config.vm.provision "shell", inline: <<-SHELL
    sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
    sed -i 's/archive.ubuntu.com/ftp.daum.net/g' /etc/apt/sources.list
    sed -i 's/security.ubuntu.com/ftp.daum.net/g' /etc/apt/sources.list
    systemctl restart ssh
  SHELL
end
```

4. Vagrant 명령어 사용
- 새로만든 특정 디렉토리에서 Vagrant up
```
> vagrant up
```

- VM 상태확인
```
> vagrant status
```

- VM 접속
```
> vagrant ssh kube-master1@192.168.56.11
```
5. VM 스냅샷 생성

6. SSH 키 복사
- 마스터 노드에서 명령어 사용(192.168.56.11)
- keygen 을 아용한 ssh 키파일 생성
```
$ ssh-keygen  
```

- 키파일 각 노드로 복사하기(비밀번호 없는 ssh를 위해)
```
$ ssh-copy-id vagrant@localhost  
$ ssh-copy-id vagrant@kube-node1 
$ ssh-copy-id vagrant@kube-node2
$ ssh-copy-id vagrant@kube-node3 
```

7. python3, pip, git 설치
```
$ sudo apt update  
$ sudo apt install -y python3 python3-pip git
```

8. kubespray 배포 및 디렉토리 변경
- kubespray Git repository 클론
```
$ git clone --single-branch --branch v2.14.2 https://github.com/kubernetes-sigs/kubespray.git 
```

- 배포 후 생선된 디렉토리로 변경
```
$ cd kubespray
```

9. requirements.txt 파일에서 의존성 확인 및 설치
```
$ sudo pip3 install -r requirements.txt  
```

10. 인벤토리 준비 및 수정
- 인벤토리 준비
```
$ cp -rfp inventory/sample inventory/mycluster  
```

- 인벤토리 수정 (:%d 를 통한 전체삭제 후 아래 코드 붙어넣기)
```
$ vi inventory/mycluster/inventory.ini
```
```
[all]  
kube-master1	ansible_host=192.168.56.11 ip=192.168.56.11 ansible_connection=local
kube-node1      ansible_host=192.168.56.21 ip=192.168.56.21
kube-node2      ansible_host=192.168.56.22 ip=192.168.56.22
kube-node3      ansible_host=192.168.56.23 ip=192.168.56.23

[all:vars]  
ansible_python_interpreter=/usr/bin/python3

[kube-master]  
kube-master1 

[etcd]  
kube-master1  

[kube-node]  
kube-node1  
kube-node2  
kube-node3  

[calico-rr]  

[k8s-cluster:children]  
kube-master  
kube-node  
calico-rr
```

11. 파라미터 확인 및 변경
```
$ vi inventory/mycluster/group_vars/k8s-cluster/addons.yml
```
```
metrics_server_enabled: true 	// 이와 같이 수정
ingress_nginx_enabled: true		// 이와 같이 수정
```

12. ansible 통신 가능 확인
-  각 호스트별로 엔서블로 통신 가능한지 확인 밑에 내용같이 나오면 설치 준비 완료
- fail 결과가 있을 시 인벤토리 파일 내용에서 빠진 것이 없는지 확인하고 각 노드에 키 기반 로그인 설정 빼먹었는지 확인
```
$ ansible all -i inventory/mycluster/inventory.ini -m ping
```

13. 플레이북 실행
- 호스트 시스템 사양, VM 개수, VM 리소스 및 네트워크 성능에 따라 20분~40분 소요
```
$ ansible-playbook -i inventory/mycluster/inventory.ini cluster.yml --become
```

14. 자격증명 가져오기
```
$ mkdir ~/.kube
$ sudo cp ~root/.kube/config ~/.kube
$ sudo chown vagrant:vagrant -R ~/.kube
```

15. kubectl 명령 자동완성
```
$ kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl
$ exec bash
```

## 키워드
- 모놀리식 아키텍쳐 : 1개의 환경에서 모든 App이 실행되는 전통적인 통합 모델, 애플리케이션의 모든 기능이 하나로 통합 구현된 형태의 아키텍처

- 마이크로서비스 아키텍쳐 : 모놀리식 아키텍쳐의 단점을 보완하기 위해 기능적으로 세분화되고 독립적으로 모듈화된 아키텍쳐 방식

- Devops : 개발과 운영사이에서 존재하는 차이를 줄이기 위해 개발, 품질, 운영팀의 결합을 강조하는 운영 방식

- 오케스트레이션 : 다수의 시스템과 애플리케이션을 쉽게 설정하고 유지관리 할 수 있는 방식을 의미

- 쿠버네티스 : 

- 마스터 : 쿠버네티스 클러스터를 구성하귀 위한 핵심 요소들의 모음, 노드에 작업을 분배, 전반적인 이벤트 감지 결정 등의 역할을 실행

- 노드 : 컨테이너가 동작하는 런타임 환경을 제공하며, 동작중인 파드를 유지

- 애드온 : 쿠버네티스의 확장 기능 요소들

- K8s API : alpah, beta stable 로 이루어지며 순차적으로 더 안정화된 애플리케이션을 제공한다. 

- Manifest :  클러스터의 상태를 나타내기 위해 오브젝트 개체를 정의하기 위해 YAML 또는 JSON 문법으로 쓰여진 파일. 필수 값으로 apiVersion, kind, metadata, spec 이 있다.

- Kubeadm : 쿠버네티스 환경을 각각의 노드마다 설치하고 연결하는 방식

- Kubespray : ansible을 이용한 자동화 코드로 한번에 다수의 노드에게 쿠버네티스 환경을 연결한다.