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

## kubectl run에서 `--generator=run-pod/v1` 없이 실행하면 Deployment가 생성될 수 있다는 의미

이 문장은 **Kubernetes/kubectl 버전 및 기본 동작 변화** 때문에, `kubectl run` 명령이 **Pod를 만들 수도 있고 Deployment를 만들 수도 있었던 역사적 이유**를 설명합니다.

---

## 1) `kubectl run`은 Pod도 만들고 Deployment도 만들었던 시기가 있었다

과거에는 같은 명령이라도 kubectl 버전/기본 generator 정책에 따라 결과가 달라질 수 있었습니다.

- 어떤 경우: `kubectl run` → **Deployment 생성**
- 다른 경우: `kubectl run` → **Pod 1개 생성**

이 혼란을 줄이기 위해 한때 `--generator` 옵션이 사용되었고, 그중 `run-pod/v1`은 **“Pod만 만들도록 강제”**하는 역할이었습니다.

---

## 2) `--generator=run-pod/v1`의 정확한 의미

예:
```bash
kubectl run mypod --image nginx --generator=run-pod/v1
```

의미:
- kubectl에게 **Deployment/ReplicaSet 같은 상위 컨트롤러를 만들지 말고**
- **v1 Pod 오브젝트를 직접 생성**하라고 강제합니다.

즉, 생성되는 리소스는:
- `Pod/mypod` (Pod만 생성)
- `Deployment` / `ReplicaSet` 없음

---

## 3) “옵션 없이 실행하면 Deployment가 생성될 수 있다”의 의미

예:
```bash
kubectl run myapp --image nginx
```

이때 kubectl 버전(특히 과거 버전)이나 당시 기본 정책에 따라:

- `Deployment/myapp`이 생성될 수 있고
- Deployment가 `ReplicaSet`을 만들고
- 최종적으로 `Pod`가 생성됩니다.

즉 “Pod가 뜨긴 하지만”, 그 Pod는 **직접 만든 Pod가 아니라 Deployment가 관리하는 Pod**일 수 있습니다.

---

## 4) 왜 이 차이가 중요한가?

### A. Deployment로 만들면 좋은 점(운영형)
- `kubectl scale deployment myapp --replicas=3` 처럼 **스케일링**이 쉬움
- `kubectl rollout undo deployment myapp` 같은 **롤백** 가능
- **롤링 업데이트/가용성 관리**에 유리
- 컨트롤러가 **원하는 상태(Desired State)** 를 유지하려고 지속적으로 관리

### B. Pod로만 만들면 특징(테스트/1회성)
- 스케일/롤링업데이트/롤백 같은 운영 기능이 없음
- “그냥 1회성 테스트 Pod” 느낌
- 실서비스/지속 운영은 보통 Deployment/StatefulSet 등을 사용

---

## 5) 요즘(현행) 기준으로 더 명확한 사용법

최근 흐름에서는 `--generator` 자체가 폐기/비권장 방향이었고, 의도를 명확히 나눠 쓰는 방식이 흔합니다.

- **Pod 1개만 띄워 테스트**
  ```bash
  kubectl run mypod --image=nginx --restart=Never
  ```

- **Deployment로 운영 형태로 띄우기**
  ```bash
  kubectl create deployment myapp --image=nginx
  ```

핵심은:
- “Pod만 만들겠다”면 Pod로 명확히,
- “운영/관리(스케일·업데이트·롤백)”가 필요하면 Deployment로 명확히 만드는 것.

---

## 6) 내 환경에서 실제로 무엇이 만들어졌는지 확인

```bash
kubectl get deploy,rs,pod | grep my
```

- `deploy/myapp`가 보이면 Deployment로 만든 것
- `pod/mypod`만 보이면 Pod만 만든 것

---
###
1) Deployment는 뭐냐?

Kubernetes API에 등록된 리소스 타입(kind) 중 하나

사람이 “원하는 상태”를 선언하면(예: nginx 3개, 이런 이미지로)
Kubernetes 컨트롤러가 그 상태를 계속 유지하도록 관리합니다.

즉 Deployment는 “프로그램/컨테이너”가 아니라,

etcd에 저장되는 선언적 설정 데이터(오브젝트)

이를 감시하고 동작시키는 건 kube-controller-manager 안의 Deployment 컨트롤러입니다.

2) “API 중 하나냐? 리소스냐?” → 둘 다 맞는 표현

API 관점: Deployment는 Kubernetes API의 apps/v1 그룹에 있는 오브젝트
예: apiVersion: apps/v1, kind: Deployment

리소스 관점: kubectl에서 deploy 또는 deployment로 다루는 리소스

kubectl get deployments
kubectl get deploy

3) Deployment가 실제로 “무엇을 만들고 관리하냐?”

보통 구조는 이렇습니다:

Deployment → ReplicaSet → Pod → Container

Deployment는 “업데이트 전략/버전 관리/원하는 replica 수”를 정의

ReplicaSet이 “Pod 개수 유지”를 실제로 수행

Pod 안에 컨테이너가 들어감

4) “kube에서 제공하는 컨테이너인가?” → 아니다

컨테이너는 이미지(nginx:1.27 등) 로부터 런타임(containerd 등)이 실행합니다.

Deployment는 그 컨테이너를 직접 실행하는 게 아니라, “Pod를 이렇게 유지해”라고 지시/관리하는 리소스입니다.

5) 한 줄로 정리

Deployment = Kubernetes API 리소스(오브젝트)
컨테이너 = Pod 안에서 실제로 실행되는 프로세스(이미지 기반)
---

---
Deployment의 핵심 스펙 구조

Deployment는 크게 3덩어리입니다.

metadata: 이름/라벨 같은 식별 정보

spec: “원하는 상태” (진짜 핵심)

status: “현재 상태” (컨트롤러가 채움)

여기서는 spec 위주로 갑니다.

1) spec.replicas

Pod를 몇 개 유지할지(원하는 개수)

예: replicas: 3 → 항상 3개 Pod를 유지하려고 함

노드 죽거나 Pod 죽으면 다시 만들어서 맞춤

2) spec.selector

Deployment가 어떤 Pod들을 “내 것”으로 볼지 정하는 조건

보통 라벨 매칭으로 결정

매우 중요: spec.template.metadata.labels 와 일치해야 함

일치 안 하면 “관리할 Pod가 없음/엉뚱한 Pod 잡음” 같은 사고가 납니다.

예: matchLabels: { app: myapp }

3) spec.template

Deployment가 만들어낼 Pod의 설계도(템플릿)

3-1) spec.template.metadata.labels

“이 Pod에 어떤 라벨을 붙일지”

이 라벨이 곧 selector에 매칭되어야 합니다.

3-2) spec.template.spec

Pod 내부 동작 정의(컨테이너, 볼륨, 포트, 프로브 등)

여기 안에서 핵심은:

containers[]

name: 컨테이너 이름

image: 실행할 이미지 (nginx:1.27 등)

ports: 컨테이너 포트(문서/Service 타겟 용도)

env: 환경변수

resources: CPU/Memory requests/limits (스케줄링/제한)

livenessProbe / readinessProbe / startupProbe

readinessProbe: “서비스 트래픽 받아도 됨?”

livenessProbe: “죽었나? 재시작 필요?”

startupProbe: “부팅 오래 걸릴 때 초기 구간 보호”

volumes / volumeMounts

imagePullSecrets

nodeSelector / affinity / tolerations

serviceAccountName

운영 실무에선 “template이 곧 Pod 스펙”이라 보시면 됩니다.

4) spec.strategy

업데이트(배포) 방식을 정의

4-1) type

RollingUpdate (기본): 새 Pod를 조금씩 올리고, 기존 Pod를 조금씩 내림

Recreate: 기존 다 내리고 새로 올림(다운타임 가능)

4-2) rollingUpdate.maxSurge

업데이트 중 일시적으로 더 띄워도 되는 Pod 수

예: replicas=10, maxSurge=2 → 최대 12개까지 잠깐 늘릴 수 있음

4-3) rollingUpdate.maxUnavailable

업데이트 중 동시에 죽어도 되는(가용성 깨져도 되는) Pod 수

예: replicas=10, maxUnavailable=1 → 최소 9개는 살아있게 유지

5) spec.minReadySeconds

새 Pod가 Ready가 된 뒤 “몇 초 동안 안정적으로 Ready 유지해야 성공으로 인정할지”

트래픽 받자마자 죽는 문제 같은 걸 완화할 때 사용

6) spec.revisionHistoryLimit

롤백을 위해 옛 ReplicaSet(리비전)을 몇 개 남길지

값 낮추면 롤백 히스토리가 줄고 리소스는 덜 씀

7) spec.progressDeadlineSeconds

배포가 이 시간 내에 진전이 없으면 실패로 판단

CI/CD에서 배포 실패 감지에 중요

8) status 쪽에서 자주 보는 값

(직접 설정하는 게 아니라 관측용)

status.replicas: 현재 전체 Pod 수

status.readyReplicas: Ready인 Pod 수

status.updatedReplicas: 새 버전으로 업데이트된 Pod 수

status.conditions: Progressing/Available 등 상태

“가장 많이 쓰는” Deployment 예시 (미니)
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: web
        image: nginx:1.27
        ports:
        - containerPort: 80
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```
---

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
