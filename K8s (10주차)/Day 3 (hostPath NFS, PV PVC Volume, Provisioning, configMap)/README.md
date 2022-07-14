# hostPath (NFS)
- 쿠버네티스 클러스터 노드(호스트)의 파일 시스템을 제공. '노드 내' 파일공유 지원하며 '경로 지정'이 가능한 볼륨
- 파드가 중지되더라도 데이터가 유지
- 파드가 재시작 시 다른 노드에서 동작하면 데이터 접근이 불가능
- 보안에 좋지않음
- 모든 노드에서 동일한 데이터를 제공해야 모든 파드가 동일한 데이터 사용

## NFS를 이용한 방식으로 hostPath 구성
- 네트워크를 이용한 볼륨 연결 방식
- NFS 서버를 구성하고 해당 서버에 대한 연결 설정 진행
- 사용하는 모든 파드가 동일한 데이터 사용
- 어떤 노드에서든 생성 가능
- NFS 서버 기능을 제공하는 파드 / 다른 시스템 이 필요

[NFS](https://github.com/choi-dongmin/K-digital_k8s/tree/main/Semi-Project%20(5%EC%A3%BC%EC%B0%A8)/Day%203%20(Web%20Server%20%EC%97%B0%EA%B2%B0%2C%20NFS%20%EC%84%9C%EB%B2%84%20%EC%97%B0%EA%B2%B0%2C%20DB%20%EC%97%B0%EB%8F%99%2C%20%EB%A1%9C%EB%93%9C%20%EB%B0%B8%EB%9F%B0%EC%84%9C%20)#nfs) 


1. NFS 서버 구성(컨트롤 플레인)
- apt update
```
$ sudo apt update
```

- nfs-kernel-server 패키지 설치
```
$ sudo apt install nfs-kernel-server -y
```

- nfs-kernel-server 서비스 활성화
```
$ sudo systemctl restart nfs-kernel-server.service
```

- 공유할 디렉토리 생성
```
$ mkdir /share
```

- /etc/exports 파일에 설정
```
$ sudo vi /etc/exports 
```
```
/share  *(rw,sync,no_root_squash,subtree_check)
```

- exportfs 명령어 실행
```
$ sudo exportfs -r
```

- 방화벽 설정 
```
$ sudo iptalbes -A INPUT -p tcp --dport 2049 -j ACCEPT
$ sudo iptalbes -A INPUT -p udp --dport 2049 -j ACCEPT
```

2. 모든 nodes에서 nfs 설정
- apt update
```
$ sudo apt update -y
```

- nfs-common 설치
```
$ sudo apt install nfs-common -y
```

- 공유 디렉토리 설정
```
$ sudo mkdir /node1_share
```

- 마운트 설정
```
$ sudo vim /etc/fstab
```
```
192.168.56.11:/share    /node1_share    nfs     rw,sync,sec=sys 0       0
```
- 마운트 
```
$ sudo mount -a
```

3. hostPath 설정 (컨트롤 플레인)
- nfs 공유 디렉토리에 index.html 만들고 저장
```
$ echo "its work" > /share/index.html
```

- svc manifest 설정
```
---
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc-hp-nfs
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: myapp-rs-hp-nfs
...
```

- rs manifest 설정
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rs-hp-nfs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp-rs-hp-nfs
  template:
    metadata:
      labels:
        app: myapp-rs-hp-nfs
    spec:
      nodeName: kube-node1 
      containers:
      - name: web-server
        image: nginx:alpine
        volumeMounts:
        - name: web-content
          mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
      volumes:
      - name: web-content
        hostPath:
          type: Directory
          path: /node1_share
...
```

- 상태 확인
```
$ kubectl get all,endpoints
NAME                        READY   STATUS    RESTARTS   AGE
pod/myapp-rs-hp-nfs-j8qtm   1/1     Running   0          17s
pod/myapp-rs-hp-nfs-x444z   1/1     Running   0          17s

NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
service/kubernetes         ClusterIP      10.233.0.1     <none>           443/TCP        23h
service/myapp-svc-hp-nfs   LoadBalancer   10.233.6.205   192.168.56.200   80:30260/TCP   17s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/myapp-rs-hp-nfs   2         2         2       18s

NAME                         ENDPOINTS                             AGE
endpoints/kubernetes         192.168.56.11:6443                    23h
endpoints/myapp-svc-hp-nfs   10.233.101.134:80,10.233.101.135:80   18s
```

- nfs 연결 확인
```
$ curl 192.168.56.200
its work
```

# PV, PVC
1. PersistentVolume : PV
	- 볼륨 종류 중 하나
	- 스토리지에 대한 지식이 필요
	- 파드 / 컨트롤러 정의 시 직접 지정
	- 파드의 라이프사이클과 스토리지의 라이프 사이클이 동일
	- 쿠버네티스 클러스터와 별개로 관리하는 스토리지 사용
	- 스토리지를 어디에 얼마나 할당할 것인가 설정
	- 일반적으로 관리자가 설정

2. PersistentVolumeClaim : PVC
	- PV를 파드에 연결할 때 사용하는 요청서
	- 스토리지 크기 및 접근제어 등의 설정을 정의
	- 요청 시 PV 사이즈 보다 크게 요청을 할 시에 연결이 안됨
	- 실제 스토리지(PV)의 스펙을 몰라도 사용할 수 있게 해주는 구성

- PV, PVC
![화면 캡처 2022-06-29 112212](https://user-images.githubusercontent.com/57117748/176473312-9c7d9efe-b309-4ef2-b9b1-f0880ea0bb97.png)


- Binding
![화면 캡처 2022-06-29 112245](https://user-images.githubusercontent.com/57117748/176475491-cb8d1392-4037-4348-9c82-7941bb98b1c6.png)

[K8s 볼륨](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)
## Life Cycle
- PV 와 PVC 연결 시 레이블 사용 가능
- PVC 를 제거하면 정책에 따라 PV 도 제거
- 기본적으로 PV 는 데이터를 영구저장
- PVC 에서 요청하는 사이즈 조정 가능

1. Provisioning(배포)
- pv를 원한는 크기로 맞게 설정하는 단계
- 정적인 방식(Static)과 동적인 방식(Dynamic) 가능
- Static Provisioning : 미리 만들어두고 요청 시 알맞은 볼륨 선택, 수동으로 모든것을 연결
- Dynamic Provisioning : Storage Class 를 통한 프로비저닝, 바인딩 자동화

2. Binding(연결)
- PV와 PVC , 그리고 파드 간의 연결을 하는 단계
- PVC 요청 시 알맞은 PV가 없으면 실패(대기 상태)
- PV 와 PVC 는 서로 1대1 관계

3. Using(사용)
- 파드와 PVC (PV) 가 연결된 상태
- 사용 중인 상태
- 모든 파드가 종료된 후에 삭제 가능
- 임의로 삭제할 수 없도록 보호되고 있다

4. Reclaim(반환)
- 사용이 끝나고 (모든 파드가 종료) PVC 를 삭제하면서 PV 에 대한 처리 방식 결정
1. Retain : 수동으로 PV 를 삭제 (쿠버네티스 오브젝트만 제거, 스토리지의 데이터는 유지), 데이터가 필요 없을 경우 당 스토리지에서 직접 제거
2. Delete : PV, Storage를 삭제, Default 값으로 지정되어 있다.
3. Recycle : PV 및 데이터 삭제 후 PVC로 재사용 가능 (중단 예정)

## 영구 볼륨의 유형
- [플러그인 목록](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes)
- 지원되는 PV 플러그 인 
```
awsElasticBlockStore- AWS 엘라스틱 블록 스토어(EBS)
azureDisk- 애저 디스크
azureFile- Azure 파일
cephfs- CephFS 볼륨
csi- 컨테이너 스토리지 인터페이스(CSI)
fc- 파이버 채널(FC) 스토리지
gcePersistentDisk- GCE 영구 디스크
glusterfs- Glusterfs 볼륨
hostPath- HostPath 볼륨(단일 노드 테스트 전용, 다중 노드 클러스터에서는 작동하지 않음, local대신 볼륨 사용 고려)
iscsi- iSCSI(IP를 통한 SCSI) 스토리지
local- 노드에 마운트된 로컬 저장 장치.
nfs- NFS(네트워크 파일 시스템) 스토리지
portworxVolume- Portworx 볼륨
rbd- 라도스 블록 디바이스(RBD) 볼륨
vsphereVolume- vSphere VMDK 볼륨
```

- 지원 되지만 향후 삭제될 예정의 PV 플러그인
```
cinder- Cinder(OpenStack 블록 스토리지)( v1.18에서 더 이상 사용되지 않음 )
flexVolume- FlexVolume( v1.23에서 더 이상 사용되지 않음 )
flocker- 플로커 스토리지( v1.22에서 더 이상 사용되지 않음 )
quobyte- 할당량 볼륨( v1.22에서 더 이상 사용되지 않음 )
storageos- StorageOS 볼륨( v1.22에서 더 이상 사용되지 않음 )
```

## 접근 허용 (Access Modes)
- [접근허용 문서](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

1. ReadWriteOnce(RWO)
	- 단수의 노드에서 읽기, 쓰기로 마운트 된다
	- 같은 노드에서 실행되는 파드의 경우 여러 포드가 볼륨에 접근이 가능하다

2. ReadOnlyMany(ROX)
	- 볼륨이 다수의 노드에서 일기 전용으로 마운트가 가능하다

3.ReadWriteMany(RWX)
	- 볼륨은 다수의 노드에서 읽기-쓰기로 마운트될 수 있습니다.

4. ReadWriteOncePod(RWOP)
	- 볼륨은 단일 Pod에 의해 읽기-쓰기로 마운트될 수 있습니다.
	- 전체 클러스터에서 하나의 포드만 해당 PVC를 읽거나 쓸 수 있도록 하려면 ReadWriteOncePod 액세스 모드를 사용합니다.
	-  Kubernetes 버전 1.22+

## Static Provisioning 사용

0. 컨트롤 노드에서 git Clone 을 통한 데이터 설치
```
git clone https://github.com/dybooksIT/kubernetes-book.git
```

1. PV 템플릿 (퍼시스턴트 볼륨 템플릿)
```
$ cd kubernetes-book/volume/

$ kubectl create -f pv-hostpath.yaml 
```

2. PVC 템플릿 (퍼시스턴스 볼륨 클레임 템플릿)
```
$ kubectl create -f pvc-hostpath.yaml
```

3. 레이블로 PVC와 PV를 연결하기
```
$ kubectl create -f pv-hostpath-label.yaml -f pvc-hostpath-vable.yaml
```

4. 파드에서 PVC 를 볼륨으로 사용하기 
```
$ kubectl create -f deployment-pvc.yaml 
```

## Dynamic Provisioning 사용
- Rook Ceph를 사용한 동적 PersistentVolume
- 스토리지를 구성할 각 노드에 사용하지 않는 블록 스토리지가 있어야 함
- 디스크가 없을 경우 수동 추가 (노드별 10G)

1. rook ceph 설치
- git을 사용하여 rook 설정 다운로드
```
$ git clone --single-branch --branch release-1.5 https://github.com/rook/rook.git 	// git clone 을 통한 설치
$ cd rook/cluster/examples/kubernetes/ceph/ 	// 작업위치 변경
```

- rook operator 설치 및 확인
```
$ kubectl create -f crds.yaml -f common.yaml -f operator.yaml
$ kubectl get pod --namespace rook-ceph
```

- rook ceph 클러스터 설치
```
$ kubectl create -f cluster.yaml // 일반 사양, 리소스에 여유가 있을때
$ kubectl create -f cluster-test.yaml // 단일 노드, 여유없으면
```

- 설치완료 확인
```
$ watch -n 1 kubectl get pod --namespace rook-ceph
```

- 클러스터 상태 체크용 리소스(toolbox) 
```
$ kubectl create -f toolbox.yaml // 클러스터 상태를 확인하도록 만든 toolbar yaml 실행
$ kubectl exec -it deploy/rook-ceph-tools --namespace rook-ceph -- bash // 진입하여 확인
# ceph status //확인
$ kubectl delete -f toolbox.yaml  // 완료후 필요 없을시 삭제
```

2.  Block 장치용 Storage Class 생성
- Storage Class 실행 및 확인
```
$ kubectl create -f csi/rbd/storageclass.yaml // 일반
$ kubectl create -f csi/rbd/storageclass-test.yaml // 테스트용 단일
$ kubectl get storageclasses.storage.k8s.io // get 을 통한 확인
```

- Block 장치 스토리지 동작 확인
```
$ kubectl create -f csi/rbd/pod.yaml -f csi/rbd/pvc.yaml  // Storage Class 이름을 통한 연결힐 파드 생성 
$ kubectl get po,pv,pvc  //  확인
$ kubectl delete -f csi/rbd/pod.yaml -f csi/rbd/pvc.yaml  // 삭제
```

3. 파일 기반 스토리지 스토리지클래스 생성
- 파일 시스템 생성
```
$ kubectl create -f filesystem.yaml // 일반
$ kubectl create -f filesystem-test.yaml // 단일
$ kubectl get storageclasses.storage.k8s.io // 스토리지 클래스 확인

$ kubectl create -f csi/cephfs/storageclass.yaml
```

- 파일 기반 스토리지 동작 확인
```
$ kubectl create -f csi/cephfs/pod.yaml -f csi/cephfs/pvc.yaml
$ kubectl get po,pv,pvc
$ kubectl delete -f csi/cephfs/pod.yaml -f csi/cephfs/pvc.yaml
```

[storage class](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/)

[ceph-rdb](https://kubernetes.io/ko/docs/concepts/storage/storage-classes/#ceph-rbd)

[동적볼륨프로비저닝](https://kubernetes.io/ko/docs/concepts/storage/dynamic-provisioning/
)
[ceph-filesystem-crd](https://rook.io/docs/rook/v1.2/ceph-filesystem-crd.html)

# Labels
- 파드 등에 설정하여 컨트롤러/서비스의 셀렉터로 연결 및 구성
- 다수의 레이블 설정 가능(셀렉터 또한 다수 레이블 가능)
- 노드에 레이블을 설정해 파드를 배치할 수 있다.
- 카나리배포 방식과 같은 특수한 설정을 위한 사용도 가능

* 배포방식
1. 롤링 업데이트(Rolling Update)
- 인스턴스를 하나씩 새로운 버전으로 교체해 나가는 방법
- 배포 및 롤백 시간이 오래 걸리지만 배포를 위한 시스템 자원 적게 사용됨

![롤링 업데이트](https://postfiles.pstatic.net/MjAyMTExMjhfNzYg/MDAxNjM4MDk4ODY5NTk2.lCJ-3ZdlzjjhuAJ0XuEsQFzks63pfUtRcrsTnEdIm6kg.7vPjp_90jE5GkMF6czaPHfBamauoqMDR-vJHdRz2zjog.PNG.koys007/image.png?type=w773)

2. 블루-그린 업데이트(Blue-Green Update)
- 구 버전 = 블루, 새로운 버전 = 그린
- 새로운 버전의 서버를 구성한 뒤에 배포 시점에 트래픽을 일제히 새로운 버전으로 전환 시킴.
- 하나의 버전만 배포 상태가 되기 때문에 버전 문제를 방지 할 수 있음.
- 문제가 발생시 빠르게 롤백 가능함
- 시스템 자원이 두배로 필요함
![블루-그린 업데이트](https://postfiles.pstatic.net/MjAyMTExMjhfMjI2/MDAxNjM4MDk4OTAyMzUx.PbMTGuKmwqenjBVLv5UH0wafEnKnMnT1PLEz5h-yC-gg.-zXKWFZJIvJom9oy1oG-3zJWpX_-pcU7irqRwDt1af4g.PNG.koys007/image.png?type=w773)

3.카나리 업데이트(Canary Update)
- 카나리아가 일산화 탄소 및 유독가스등에 매우 민감하여 적은 양을 들이켜도 죽는다는 점을 착안한 방법으로 카나리아 배포 방법은 위험을 빠르게 감지할 수 있는 배포 방법
- 구 버전의 서버와 새로운 버전의 서버를 구성한 뒤에 일부 트래픽이나 특정한 사용자들에게만 새로운 버전을 사용하도록함 (AB테스트)
-  오류와 성능을 모니터링하는데 유용함
![카나리 업데이트](https://postfiles.pstatic.net/MjAyMTExMjhfMTEx/MDAxNjM4MDk4ODQ0MDYx.517xhJqd3-TSfBzw6T8ASf61rm_2aBsyXrIasUpw8ngg.cs3HOOmFrKkJ7kb0MYwA3XrcWEDVAWu_br74emOSBzYg.PNG.koys007/image.png?type=w773)
