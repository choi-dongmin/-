# DaemonSet
- 데몬셋(DaemonSet)은 연결된 노드에 파드를 '하나씩' 생성하는 컨트롤러
- nodeSelector를 이용한 노드 식별(DaemonSet.spec.template.nodeSelector)
- 노드가 추가되면 자동으로 컨트롤러는 하나의 파드를 배치, 노드가 제거되면 삭제된 파드를 다른 노드에 배치되지 않음
- apps 그룹의 v1을 사용

## DaemonSet 사용

1. DeamonSet No nodeSelector
- nodeSelector가 없는 DeamonSet 파일작성. 
```
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: testapp-daemon-notnodeselector
spec:
  selector:
    matchLabels:
      app: testapp-ds
  template:
    metadata:
      labels:
        app: testapp-ds
    spec:
      containers:
      - name: testapp
        image: ghcr.io/c1t1d0s7/go-myweb
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
...
``` 
```
$ kubectl get nodes

NAME           STATUS   ROLES    AGE    VERSION
kube-master1   Ready    master   3d1h   v1.18.10
kube-node1     Ready    <none>   3d1h   v1.18.10
kube-node2     Ready    <none>   3d1h   v1.18.10
kube-node3     Ready    <none>   3d1h   v1.18.10
```
```
Every 1.0s: kubectl get pods --show-labels                                                                               kube-master1: Thu Jun 23 08:50:04 2022
NAME                                   READY   STATUS    RESTARTS   AGE     LABELS
testapp-daemon-notnodeselector-5hdwq   1/1     Running   0          3m49s   app=testapp-ds,controller-revision-hash=6895fcbd55,pod-template-generation=1
testapp-daemon-notnodeselector-jht2q   1/1     Running   0          3m49s   app=testapp-ds,controller-revision-hash=6895fcbd55,pod-template-generation=1
testapp-daemon-notnodeselector-xnmvz   1/1     Running   0          3m49s   app=testapp-ds,controller-revision-hash=6895fcbd55,pod-template-generation=1
```

2. DeamonSet with nodeSelector
- java라는 레이블을 가진 노드들에게 파드를 하나씩 만들 컨트롤러 생성
```
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: testapp-ds-java
spec:
  selector:
    matchLabels:
      app: java
  template:
    metadata:
      labels:
        app: java
    spec:
      nodeSelector:
        node: java
      containers:
      - name: testapp
        image: ghcr.io/c1t1d0s7/go-myweb
        ports:
        - containerPort: 8080
        imagePullPolicy: Never

...
```

- python 라는 레이블을 가진 노드들에게 파드를 하나씩 유지 할 컨트롤러 생성
```
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: testapp-ds-python
spec:
  selector:
    matchLabels:
      app: python
  template:
    metadata:
      labels:
        app: python
    spec:
      nodeSelector:
        node: python
      containers:
      - name: testapp
        image: ghcr.io/c1t1d0s7/go-myweb
        ports:
        - containerPort: 8080
        imagePullPolicy: Never
...
```

- java, python 이라는 레이블을 가진 node 가 없음으로 파드 생성하지 않는다
```
$ kubectl get pods
No resources found in default namespace.
```

- 그러나 DaemonSet 은 작동중이다.
```
$ kubectl get daemonsets.apps --watch

NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
testapp-ds-java     0         0         0       0            0           node=java       4m32s
testapp-ds-python   0         0         0       0            0           node=python     24s
```

- nodess 확인
```
$ kubectl get nodes --show-labels
```

- nodes 에 레이블을 추가
```
$ kubectl label nodes [nodename] node=java
$ kubectl label nodes [nodename] node=java
$ kubectl label nodes [nodename] node=python
```


- nodes에 레이블을 추가 시켜 DaemonSet에 의해 파드가 생성
```
$ watch -n1 kubectl get pods --show-labels

Every 1.0s: kubectl get pods --show-labels                                                                               kube-master1: Thu Jun 23 09:35:37 2022
NAME                      READY   STATUS    RESTARTS   AGE     LABELS
testapp-ds-java-8nvqb     1/1     Running   0          5m19s   app=java,controller-revision-hash=7788b4f657,pod-template-generation=1
testapp-ds-java-ngz6w     1/1     Running   0          2m28s   app=java,controller-revision-hash=7788b4f657,pod-template-generation=1
testapp-ds-python-rr2p7   1/1     Running   0          2m16s   app=python,controller-revision-hash=7dc79fd8df,pod-template-generation=1
```

- DaemonSet 삭제
```
$ kubectl delete daemonsets.apps [DaemonSetName]

$ kubectl delete daemonsets.apps --all
daemonset.apps "testapp-ds-java" deleted
daemonset.apps "testapp-ds-python" deleted
```


# Job
- ReplicaSet 혹은 DaemonSet은 지속적인 작업에 초점을 맞춰 애플리케이션이 실행되고 완료되면 파드의 할 일이 끝난것으로 간주하고 파드를 종료시킨다.
- Job 컨트롤러는 임시작업(AD-HOC) 및 배치(Batch) 작업에 유용하게 사용된다.
- API는 batch 그룹의 v1 버전을 사용
- 지속적이지 않은 일회적인 작업이기 때문에 restartPolicy :OnFailure or Never

## Job 재시작 정책
- job.spec.template.spec.restartPolicy

1. Onfailure: 실패 시 재시작 (정상 종료 시 재시작하지 않음)
2. Never: 종료 또는 오류 시 절대 재시작 하지 않음

* Always는 오류가 생김

## Job 사용 

- job manifest file 생성
```
$ testapp-job.yaml
```
```
apiVersion: batch/v1
kind: Job
metadata:
  name: testapp-job
spec:
  template:
    metadata:
      labels:
        app: testapp-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: testapp
        image: busybox
        command: ["sleep", "20"]
```

- 
```
$ kubectl get pods
NAME                READY   STATUS              RESTARTS   AGE
testapp-job-bpp2z   0/1     ContainerCreating   0          6s
$ kubectl get jobs.batch
NAME          COMPLETIONS   DURATION   AGE
testapp-job   0/1           14s        14s

$ kubectl get pods
NAME                READY   STATUS      RESTARTS   AGE
testapp-job-bpp2z   0/1     Completed   0          25s
$ kubectl get jobs.batch
NAME          COMPLETIONS   DURATION   AGE
testapp-job   1/1           21s        29s
```

# Job completions (다중작업)
- completions를 통해 작업의 완료 개수 입력
- 작업을 여러 번 순차적으로 실행
- Job.spec.completions
- 즉, 하나의 파드가 생성되어 잡이 실행되고 완료되면, 두 번째 파드가 생성되고 잡이 완료되고, 마지막으로 세 번째 파드가 생성되고 잡이 완료되어야 모든 잡이 완료


## Job completions 사용

- Job completions Manifest 작성
```
---
apiVersion: batch/v1
kind: Job
metadata:
  name: testapp-job-comp
spec:
  completions: 3
  template:
    metadata:
      labels:
        app: testapp-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: testapp
        image: busybox
        command: ['sleep', '15']
...
```


- Job completions 동작 확인
```

$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
testapp-job-comp-67lb4   1/1     Running   0          15s

$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
testapp-job-comp-67lb4   0/1     Completed           0          22s
testapp-job-comp-xgb8l   0/1     ContainerCreating   0          1s

$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
testapp-job-comp-67lb4   0/1     Completed           0          44s
testapp-job-comp-xgb8l   0/1     Completed           0          23s
testapp-job-comp-zg8p9   0/1     ContainerCreating   0          2s

$ kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
testapp-job-comp-67lb4   0/1     Completed   0          66s
testapp-job-comp-xgb8l   0/1     Completed   0          45s
testapp-job-comp-zg8p9   0/1     Completed   0          24s
```
```
Every 1.0s: kubectl get pods --show-labels                                                                               kube-master1: Thu Jun 23 11:15:53 2022
NAME                     READY   STATUS      RESTARTS   AGE     LABELS
testapp-job-comp-67lb4   0/1     Completed   0          4m44s   app=testapp-job,controller-uid=9b3380cf-44e1-443d-9769-41cc9f3230fa,job-name=testapp-job-comp
testapp-job-comp-xgb8l   0/1     Completed   0          4m23s   app=testapp-job,controller-uid=9b3380cf-44e1-443d-9769-41cc9f3230fa,job-name=testapp-job-comp
testapp-job-comp-zg8p9   0/1     Completed   0          4m2s    app=testapp-job,controller-uid=9b3380cf-44e1-443d-9769-41cc9f3230fa,job-name=testapp-job-comp
```

# Parallelism Job (병렬작업)
- 다중작업은 순서대로 직렬 실행하지만, 이를 병렬형태로 동시에 실행하는 작업을 Parallelism Job이라고 한다.
- Default = 1
- job.spec.parallelism: 동시에 실행할 잡의 개수 지정

## Job parallelism 사용

- Job parallelism Manifest 작성
```
apiVersion: batch/v1
kind: Job
metadata:
  name: testapp-job-para
spec:
  completions: 3
  parallelism: 2
  template:
    metadata:
      labels:
        app: testapp-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: testapp
        image: busybox
        command: ['sleep', '10']
```

- parallelism 확인
```
$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
testapp-job-para-mlgjd   0/1     ContainerCreating   0          4s
testapp-job-para-tbh85   0/1     ContainerCreating   0          4s
$ kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
testapp-job-para-mlgjd   1/1     Running   0          14s
testapp-job-para-tbh85   1/1     Running   0          14s

$ kubectl get pods
NAME                     READY   STATUS              RESTARTS   AGE
testapp-job-para-kx2jl   0/1     ContainerCreating   0          2s
testapp-job-para-mlgjd   0/1     Completed           0          17s
testapp-job-para-tbh85   0/1     Completed           0          17s
$ kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
testapp-job-para-kx2jl   1/1     Running     0          9s
testapp-job-para-mlgjd   0/1     Completed   0          24s
testapp-job-para-tbh85   0/1     Completed   0          24s
$ kubectl get pods
NAME                     READY   STATUS      RESTARTS   AGE
testapp-job-para-kx2jl   0/1     Completed   0          18s
testapp-job-para-mlgjd   0/1     Completed   0          33s
testapp-job-para-tbh85   0/1     Completed   0          33s
vagrant@kube-master1:~/test$
```
```
Every 1.0s: kubectl get pods --show-labels    

NAME                     READY   STATUS              RESTARTS   AGE   LABELS
testapp-job-para-mmb75   0/1     Completed           0          16s   app=testapp-job,controller-uid=d783b9cf-3afb-4b3b-8537-385086354bf0,job-name=testapp-job-para
testapp-job-para-pcs4h   0/1     Completed           0          16s   app=testapp-job,controller-uid=d783b9cf-3afb-4b3b-8537-385086354bf0,job-name=testapp-job-para
testapp-job-para-ttmfh   0/1     ContainerCreating   0          0s    app=testapp-job,controller-uid=d783b9cf-3afb-4b3b-8537-385086354bf0,job-name=testapp-job-para
```

# Cron Job
- 일회성인 Job 컨트롤러를 주기적으로 반복해 실행하고자 할 때 사용한다.
- Linux의 Crontab과 유사
- 아직 베타 기능이며 크론잡 API 역시 batch 그룹을 사용하며 버전은 v1beta1을 사용 오브젝트 종류는 CronJob
- cronjob.spec.schedule : 반복 지정 작업 Cron 형식으로 작성


## Cron Job 사용

- Cron Job manifest 작성
```
---
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: testapp-cron
spec:
 schedule: "* * * * *"
 jobTemplate:
   spec:
     template:
       metadata:
         labels:
           app: testapp-cron
       spec:
         restartPolicy: OnFailure
         containers:
         - name: testapp
           image: busybox
           command: ['sleep', '20']
...
```

- 확인
```
$ kubectl get pods
NAME                            READY   STATUS              RESTARTS   AGE
testapp-cron-1655984040-cjm98   0/1     ContainerCreating   0          4s
$ kubectl get pods
NAME                            READY   STATUS    RESTARTS   AGE
testapp-cron-1655984040-cjm98   1/1     Running   0          7s
$ kubectl get pods
NAME                            READY   STATUS      RESTARTS   AGE
testapp-cron-1655984040-cjm98   0/1     Completed   0          71s
testapp-cron-1655984100-56w8f   1/1     Running     0          11s
$ kubectl get pods
NAME                            READY   STATUS      RESTARTS   AGE
testapp-cron-1655984040-cjm98   0/1     Completed   0          87s
testapp-cron-1655984100-56w8f   0/1     Completed   0          27s


$ watch -n1 kubectl get jobs --show-labels

Every 1.0s: kubectl get jobs --show-labels                                                                                                                                kube-master1: Thu Jun 23 11:38:20 2022
NAME                      COMPLETIONS   DURATION   AGE     LABELS
testapp-cron-1655984100   1/1           26s        1m16s   app=testapp-cron,controller-uid=ceacda10-7ae8-4ad2-9963-8bfbd1c256a0,job-name=testapp-cron-1655984100
testapp-cron-1655984160   1/1           27s        1m49s   app=testapp-cron,controller-uid=a7f1cbb9-0015-43c2-9fe0-1c6696d82566,job-name=testapp-cron-1655984160

$ kubectl get cronjobs.batch
NAME           SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
testapp-cron   * * * * *   False     2        26s             1m55s
```

- crontab 종료
```
$ kubectl delete cronjobs.batch [cronjobname]

$ kubectl delete cronjobs.batch --all
cronjob.batch "testapp-cron" deleted
```

## 키워드
- DaemonSet : 마스터와 연결된 노드들에게 노드 셀렉터에 따라서 파드를 하나씩 유지하는 컨트롤러.

- nodeSelector: nodes를 레이블을 이용해 식별하게 해주는 항목. DaemonSet.spec.template.nodeSelector

- Job: 지속적이지 않은 일회성 작업 즉, Job은 파드(컨테이너)의 애플리케이션이 실행이 완료/종료되는 것에 초점을 맞춘 컨트롤러

- Job Completions: Job을 실행할때 이전 작업이 완료되면 순차적으로 작업을 실행한다.
Job.spec.completions  

- Job Paralleism : Completions을 기반으로 순차적 실행하지만 한번에 많이 실행 하고 싶다면 paralleism 을 이용해 병렬으로 처리한다. Job.spec.paralleism  

- Cron Job: 일회적인 작업(Job)을 반복,주기적으로 실행 할 수 있도록 해주는 컨트롤러
 cronjob.spec.schedule에 cron Time 형식으로 작성
