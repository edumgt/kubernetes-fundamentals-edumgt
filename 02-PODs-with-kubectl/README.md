# Kubernetes - Pod 다루기

## Step-01: Pod 소개
- **Pod란?**
  - Kubernetes에서 컨테이너를 실행하는 최소 단위입니다.
  - 하나 이상의 컨테이너가 **같은 네트워크/스토리지 네임스페이스**를 공유합니다.
- **멀티 컨테이너 Pod란?**
  - 사이드카 패턴(로그 수집, 프록시 등)처럼 같은 생명주기를 갖는 컨테이너를 묶어 실행합니다.

## Step-02: Pod 데모

### 워커 노드 상태 확인
- Kubernetes 워커 노드가 준비 상태인지 확인합니다.
```
# 워커 노드 상태 확인
kubectl get nodes

# 워커 노드 상태를 상세 정보와 함께 확인
kubectl get nodes -o wide
```

### Pod 생성
- Pod를 생성합니다.
```
# 템플릿
kubectl run <desired-pod-name> --image <Container-Image> --generator=run-pod/v1

# 예시: Pod 이름과 컨테이너 이미지 지정
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0 --generator=run-pod/v1
```
- **중요:** `--generator=run-pod/v1` 옵션 없이 실행하면 Deployment가 생성될 수 있습니다.
- **중요:**
  - Kubernetes 1.18 이후 `kubectl run` 명령이 단순화되었습니다.
  - 최신 버전에서는 아래와 같이 `--generator` 없이도 Pod만 생성됩니다.
```
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0
```

### Pod 목록 확인
```
# Pod 목록 조회
kubectl get pods

# pods의 축약어는 po
kubectl get po
```

### Pod 목록(상세) 확인
- `-o wide` 옵션으로 Pod가 실행 중인 노드 정보를 함께 확인할 수 있습니다.
```
kubectl get pods -o wide
```

### 백그라운드에서 일어나는 일
1. Kubernetes가 Pod 생성 요청을 처리합니다.
2. 필요한 Docker 이미지를 레지스트리에서 가져옵니다.
3. Pod 내부에 컨테이너를 생성합니다.
4. 컨테이너가 실행됩니다.

### Pod 상세 정보 확인
- 장애 진단 시 유용합니다. 이벤트 로그를 통해 원인을 추적할 수 있습니다.
```
# Pod 이름 확인
kubectl get pods

# Pod 상세 조회
kubectl describe pod <Pod-Name>
kubectl describe pod my-first-pod
```

### 애플리케이션 접근
- 기본적으로 Pod는 클러스터 내부에서만 접근 가능합니다.
- 외부 접근을 위해 **NodePort Service**를 생성합니다.
- Service는 Kubernetes 핵심 개념으로, 네트워크 접근을 추상화합니다.

### Pod 삭제
```
# Pod 이름 확인
kubectl get pods

# Pod 삭제
kubectl delete pod <Pod-Name>
kubectl delete pod my-first-pod
```

## Step-03: NodePort Service 소개
- **Service란?** Pod의 동적인 IP를 안정적으로 접근하기 위한 네트워크 추상화입니다.
- **NodePort Service란?**
  - 모든 노드의 특정 포트를 열어 외부에서 접근 가능하게 합니다.
  - 학습/테스트용으로 유용하지만, 실제 운영에서는 LoadBalancer나 Ingress를 더 많이 사용합니다.

## Step-04: 데모 - Service로 Pod 외부 노출
- NodePort Service로 애플리케이션을 외부에서 접근하도록 설정합니다.
- **Ports**
  - **port:** Service가 내부에서 사용하는 포트
  - **targetPort:** 컨테이너의 실제 포트
  - **nodePort:** 노드에서 외부 접근에 사용하는 포트
```
# Pod 생성
kubectl run <desired-pod-name> --image <Container-Image> --generator=run-pod/v1
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0 --generator=run-pod/v1

# Pod를 Service로 노출
kubectl expose pod <Pod-Name> --type=NodePort --port=80 --name=<Service-Name>
kubectl expose pod my-first-pod --type=NodePort --port=80 --name=my-first-service

# Service 정보 확인
kubectl get service
kubectl get svc

# 워커 노드 Public IP 확인
kubectl get nodes -o wide
```
- **애플리케이션 접근 예시**
```
http://<node1-public-ip>:<Node-Port>
```

- **targetPort에 대한 중요 메모**
  - `targetPort`를 명시하지 않으면 기본적으로 `port`와 동일하게 설정됩니다.
```
# 아래 명령은 서비스 포트(81)와 컨테이너 포트(80)가 달라 접근 실패 가능
kubectl expose pod my-first-pod --type=NodePort --port=81 --name=my-first-service2

# targetPort 지정
kubectl expose pod my-first-pod --type=NodePort --port=81 --target-port=80 --name=my-first-service3

# Service 정보 확인
kubectl get service
kubectl get svc

# 워커 노드 Public IP 확인
kubectl get nodes -o wide
```
- **애플리케이션 접근 예시**
```
http://<node1-public-ip>:<Node-Port>
```

## Step-05: Pod와 상호작용

### Pod 로그 확인
```
# Pod 이름 확인
kubectl get po

# Pod 로그 출력
kubectl logs <pod-name>
kubectl logs my-first-pod

# 로그 스트리밍
kubectl logs -f my-first-pod
```
- **중요 메모**
  - 추가 옵션은 공식 문서의 **Interacting with running Pods** 섹션을 참고하세요.
  - 로그 확인은 트러블슈팅에 매우 중요합니다.
  - **Reference:** https://kubernetes.io/docs/reference/kubectl/cheatsheet/

### Pod 내부 컨테이너 접속
- **컨테이너 내부에 접속해 명령 실행**
```
# Nginx 컨테이너에 접속
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it my-first-pod -- /bin/bash

# 컨테이너 내부 명령 예시
ls
cd /usr/share/nginx/html
cat index.html
exit
```

- **단일 명령 실행**
```
kubectl exec -it <pod-name> env

# 예시
kubectl exec -it my-first-pod env
kubectl exec -it my-first-pod ls
kubectl exec -it my-first-pod cat /usr/share/nginx/html/index.html
```

## Step-06: Pod/Service YAML 출력 확인
### YAML 출력
```
# Pod 정의 YAML 출력
kubectl get pod my-first-pod -o yaml

# Service 정의 YAML 출력
kubectl get service my-first-service -o yaml
```

## 트러블슈팅 체크리스트
- **Service 접근 실패 시 포트 매핑 확인**
  - `port`와 `targetPort`가 불일치한데 `targetPort`를 생략하면 기본적으로 `port`와 동일하게 설정됩니다.
  - 컨테이너 포트와 다르면 접근이 실패할 수 있으니, Service YAML에서 `targetPort`를 확인합니다.
```
kubectl get service my-first-service -o yaml
kubectl describe pod my-first-pod
```

## Step-07: 정리(Clean-Up)
```
# default 네임스페이스의 모든 객체 확인
kubectl get all

# Service 삭제
kubectl delete svc my-first-service
kubectl delete svc my-first-service2
kubectl delete svc my-first-service3

# Pod 삭제
kubectl delete pod my-first-pod

# 최종 확인
kubectl get all
```
