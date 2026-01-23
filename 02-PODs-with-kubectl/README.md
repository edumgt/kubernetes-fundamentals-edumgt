# Kubernetes - Pod 다루기

## Step 01: Pod 소개
- **Pod란?**
  - Kubernetes에서 컨테이너를 실행하는 최소 단위입니다.
  - 하나 이상의 컨테이너가 **같은 네트워크/스토리지 네임스페이스**를 공유합니다.
- **멀티 컨테이너 Pod란?**
  - 사이드카 패턴(로그 수집, 프록시 등)처럼 같은 생명주기를 갖는 컨테이너를 묶어 실행합니다.

## Step 02: Pod 데모

### 2-1. 워커 노드 상태 확인
```bash
# 워커 노드 상태 확인
kubectl get nodes

# 워커 노드 상태를 상세 정보와 함께 확인
kubectl get nodes -o wide
```

### 2-2. Pod 생성
```bash
# 템플릿
kubectl run <desired-pod-name> --image <container-image> --generator=run-pod/v1

# 예시: Pod 이름과 컨테이너 이미지 지정
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0 --generator=run-pod/v1
```

**중요 메모: `kubectl run` 동작 변화**
- 과거 버전에서는 `kubectl run`이 **Deployment** 또는 **Pod**를 만들 수 있었습니다.
- 이를 명확히 하기 위해 `--generator=run-pod/v1`가 사용되었고, 이는 **Pod만 생성**하도록 강제합니다.
- 최신 버전(1.18+)에서는 `kubectl run`이 기본적으로 Pod만 생성하며, `--generator`는 비권장입니다.

```bash
# 최신 버전에서 Pod만 생성
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0
```

**의도를 명확히 하는 권장 흐름**
```bash
# Pod 1개 테스트 목적
kubectl run mypod --image=nginx --restart=Never

# 운영 형태(Deployment)
kubectl create deployment myapp --image=nginx
```

**어떤 리소스가 생성되었는지 확인**
```bash
kubectl get deploy,rs,pod | grep my
```

### 2-3. Pod 목록 확인
```bash
# Pod 목록 조회
kubectl get pods

# pods의 축약어는 po
kubectl get po

# 상세 정보 확인
kubectl get pods -o wide
```

### 2-4. Pod 상세 정보 확인
```bash
# Pod 이름 확인
kubectl get pods

# Pod 상세 조회
kubectl describe pod <pod-name>
kubectl describe pod my-first-pod
```

### 2-5. 애플리케이션 접근
- Pod는 기본적으로 클러스터 내부에서만 접근 가능합니다.
- 외부 접근을 위해 **Service(NodePort)**를 생성합니다.

### 2-6. Pod 삭제
```bash
kubectl delete pod <pod-name>
kubectl delete pod my-first-pod
```

## Step 03: NodePort Service 소개
- **Service란?** Pod의 동적인 IP를 안정적으로 접근하기 위한 네트워크 추상화입니다.
- **NodePort Service란?**
  - 모든 노드의 특정 포트를 열어 외부에서 접근 가능하게 합니다.
  - 학습/테스트용으로 유용하지만, 실제 운영에서는 LoadBalancer나 Ingress를 더 많이 사용합니다.

## Step 04: 데모 - Service로 Pod 외부 노출

### 4-1. Pod 생성
```bash
kubectl run <desired-pod-name> --image <container-image> --generator=run-pod/v1
kubectl run my-first-pod --image stacksimplify/kubenginx:1.0.0 --generator=run-pod/v1
```

### 4-2. Pod를 Service로 노출
```bash
kubectl expose pod <pod-name> --type=NodePort --port=80 --name=<service-name>
kubectl expose pod my-first-pod --type=NodePort --port=80 --name=my-first-service
```

### 4-3. Service 정보 확인
```bash
kubectl get service
kubectl get svc

# 워커 노드 Public IP 확인
kubectl get nodes -o wide
```

**애플리케이션 접근 예시**
```
http://<node1-public-ip>:<node-port>
```

### 4-4. targetPort 중요 메모
- `targetPort`를 명시하지 않으면 기본적으로 `port`와 동일하게 설정됩니다.
- 컨테이너 포트와 다르면 접근이 실패할 수 있으니, `targetPort`를 지정합니다.

```bash
# 서비스 포트(81)와 컨테이너 포트(80)가 달라 접근 실패 가능
kubectl expose pod my-first-pod --type=NodePort --port=81 --name=my-first-service2

# targetPort 지정
kubectl expose pod my-first-pod --type=NodePort --port=81 --target-port=80 --name=my-first-service3

# Service 정보 확인
kubectl get service
kubectl get svc
```

**애플리케이션 접근 예시**
```
http://<node1-public-ip>:<node-port>
```

## Step 05: Pod와 상호작용

### 5-1. Pod 로그 확인
```bash
# Pod 이름 확인
kubectl get po

# Pod 로그 출력
kubectl logs <pod-name>
kubectl logs my-first-pod

# 로그 스트리밍
kubectl logs -f my-first-pod
```

**추가 참고**
- 공식 문서: https://kubernetes.io/docs/reference/kubectl/cheatsheet/

### 5-2. Pod 내부 컨테이너 접속
```bash
# Nginx 컨테이너에 접속
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it my-first-pod -- /bin/bash

# 컨테이너 내부 명령 예시
ls
cd /usr/share/nginx/html
cat index.html
exit
```

**단일 명령 실행**
```bash
kubectl exec -it <pod-name> env

# 예시
kubectl exec -it my-first-pod env
kubectl exec -it my-first-pod ls
kubectl exec -it my-first-pod cat /usr/share/nginx/html/index.html
```

## Step 06: Pod/Service YAML 출력 확인
```bash
# Pod 정의 YAML 출력
kubectl get pod my-first-pod -o yaml

# Service 정의 YAML 출력
kubectl get service my-first-service -o yaml
```

## 트러블슈팅 체크리스트
- **Service 접근 실패 시 포트 매핑 확인**
  - `port`와 `targetPort`가 불일치한데 `targetPort`를 생략하면 기본적으로 `port`와 동일하게 설정됩니다.
  - 컨테이너 포트와 다르면 접근이 실패할 수 있으니, Service YAML에서 `targetPort`를 확인합니다.

```bash
kubectl get service my-first-service -o yaml
kubectl describe pod my-first-pod
```

## Step 07: 정리(Clean-Up)
```bash
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
