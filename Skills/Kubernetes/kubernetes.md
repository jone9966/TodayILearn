# 쿠버네티스
https://kubernetes.io/ko/docs/home/
## 쿠버네티스란
쿠버네티스(k8s)는 컨테이너 오케스트레이션 도구로, 여러 대의 물리적 서버에서 실행되는 컨테이너들을 관리한다.
쿠버네티스는 여러 서버들을 클러스터로 구성하여 각 서버에서 실행되는 프로그램(컨테이너)를 하나의 서버에서 관리할 수 있도록 한다.
이 때 클러스터를 관리하는 서버를 마스터 노드(Control plane)이라 하고 각 컨테이너가 실행되는 서버들을 워커 노드 라고 한다.
- 마스터 노드
    - kubectl: 쿠버네티스의 상태를 확인하고 원하는 상태를 요청하는 역할
    - kube-apiserver: 외부와 통신하는 프로세스로 kubectl로부터 명령을 받아 실행
    - kube-controller-manager: 컨트롤러를 통합 관리 및 실행
    - kube-scheduler: 파드를 워커 노드에 할당
    - cloud-controller-manager: 클라우드 서비스와 연동해 서비스를 생성
    - etcd: 클러스터 관련 정보를 관리하는 데이터 베이스
- 워커 노드
    - kube-let: 마스터 노드의 스케줄러와 연동하여 워커 노드에 파드를 배치 및 실행
    - kube-proxy: 네트워크 통신의 라우팅 메커니즘


## 오브젝트
쿠버네티스는 크게 오브젝트와 컨트롤러라는 개념으로 나뉜다.
오브젝트는 사용자가 쿠버네티스에 바라는 상태(desired state)이고 컨트롤러는 오브젝트가 원래 설정된 상태를 잘 유지 할 수 있게 관리하는 역할을 한다.

- kubectl을 통해 쿠버네티스의 상태를 확인하고 요청할 수 있다.
```
kubectl apply -f [파일명 또는 URL]
kubectl get [TYPE]
kubectl describe [TYPE]/[NAME]
kubectl delete [TYPE]/[NAME]
kubectl logs [POD_NAME]
kubectl exec [-it] [POD_NAME] -- [COMMAND]
```

### 네임스페이스
네임스페이스는 쿠버네티스 클러스터를 여러 가상 클러스터로 격리하기 위해 존재하는 논리적인 분리 단위이다.
```
kubectl get ns
kubectl create ns <name>
kubectl delete ns <name>
kubectl config set-context --current --namespace=<name>
```

### 파드
파드는 쿠버네티스에서 생성하고 관리할 수 있는 가장 작은 컴퓨팅 단위로, 하나 이상의 컨테이너로 구성된다.
공통된 명세 또는 정의를 통해 구동되는 컨테이너는 파드내의 컨테이너는 스토리지와 네트워크를 공유한다.
```
kubectl get pods --namespace=<ns-name>
kubectl run <pod-name> --image=<docker-image> --namespace=<ns-name>
kubectl apply -f <yaml-file>
kubectl edit pods <pod-name>
kubectl exec -it <pod-name> /bin/bash
kubectl delete pod <pod-name>
```

### 서비스
서비스는 파드 집합에서 실행중인 애플리케이션을 네트워크에 노출시키기 위한 추상화 방법이다.
파드가 생성되는 노드는 고정적이지 않기 때문에 파드 내부의 네트워크 역시 생성될 때마다 변하게 된다.
따라서 쿠버네티스는 파드가 외부와 통신할 수 있도록 클러스터 내부에서 고정적인 IP를 갖는 서비스를 사용한다.
```
kubectl get svc --namespace=<ns-name>
kubectl create -f <yaml-file>
kubectl apply -f <yaml-file>
kubectl edit svc <svc-name>
kubectl exec -it <pod-name> /bin/bash
kubectl delete pod <pod-name>
```

## 컨트롤러
### ReplicaSet
Pod을 정해진 수만큼 복제하고 관리
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: echo-rs
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```
    

### Deployment
ReplicaSet을 이용하여 Pod을 업데이트하고 이력을 관리하여 롤백Rollback 하거나 특정 버전revision
으로 돌아갈 수 있다.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```