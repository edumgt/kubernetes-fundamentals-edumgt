# Kubernetes Fundamentals
 
### 분류
---
Kubernetes에서 Node / Namespace(NS) / Service / Pod를 “무엇이고, 왜 필요한지, 서로 어떻게 연결되는지” 중심으로 정리한 것입니다.
---

### 추가 설명
---
이 문서는 Kubernetes 기본 개념을 빠르게 훑기 위한 요약본이며, 이후 실습 파트에서 실제 명령과 매니페스트로 연결됩니다.
운영 관점에서는 권한(RBAC)과 모니터링/로그 수집까지 함께 고려해야 전체 흐름을 이해하기 쉽습니다.
---

### Node
```
클러스터를 구성하는 실제 머신(가상/물리) 입니다. Pod가 여기 위에서 실행됩니다.

종류

Control Plane Node: API 서버, 스케줄러, 컨트롤러 등 “클러스터 관리”

Worker Node: 실제로 Pod(컨테이너) 를 실행

Node가 하는 일

컨테이너 런타임(containerd 등)로 컨테이너 실행

kubelet이 “내 노드에 떠야 할 Pod”를 받아서 실행/상태보고

자주 쓰는 명령

kubectl get nodes -o wide
kubectl describe node <node>
```
### Namespace (NS)
```
클러스터 안의 리소스를 논리적으로 구분하는 “가상 폴더/테넌트” 입니다.

왜 쓰나?

팀/프로젝트/환경(dev, stage, prod) 분리

리소스 이름 충돌 방지(같은 이름의 Service/Pod도 NS가 다르면 가능)

RBAC 권한 분리, ResourceQuota(자원 할당 제한) 적용

특징

대부분 리소스는 Namespace에 속함 (Pod, Service, Deployment 등)

일부는 클러스터 전역(예: Node, PersistentVolume 등)

자주 쓰는 명령

kubectl get ns
kubectl get all -n <ns>
```

### Pod
```
Kubernetes에서 배포/실행의 최소 단위입니다. “컨테이너 1개 이상”을 묶은 단위.

Pod 안에는?

컨테이너(들)

네트워크/스토리지/환경변수 설정

핵심 특징

Pod는 IP를 하나 받음(컨테이너들이 그 IP를 공유)

Pod 안의 컨테이너들은 localhost로 서로 통신 가능

Pod는 “쉽게 죽고 다시 생기는” 존재(일회성에 가까움)

운영에서는 보통 Pod를 직접 만들기보다

Deployment/StatefulSet/DaemonSet 같은 컨트롤러가 Pod를 생성/유지

자주 쓰는 명령

kubectl get pod -n <ns> -o wide
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns>
kubectl exec -it <pod> -n <ns> -- sh
```

### Service
```
Pod는 IP가 바뀌기 쉽습니다(재생성/스케줄링).
Service는 Pod들을 “고정된 주소(가상 IP/DNS)”로 묶어주는 추상화입니다.

왜 필요?

Pod가 바뀌어도 클라이언트는 항상 같은 주소로 접근

로드밸런싱(여러 Pod로 트래픽 분산)

Service가 연결하는 대상은?

selector(라벨)로 Pod를 선택해서 “Endpoint”를 구성

결과적으로 Service → (Endpoints) → Pod 로 연결

주요 타입

ClusterIP (기본): 클러스터 내부 전용 고정 IP

NodePort: 모든 Node의 특정 포트를 열어 외부에서 접근 가능

LoadBalancer: 클라우드 로드밸런서를 붙여 외부 IP 제공

Headless (clusterIP: None): 로드밸런싱 없이 Pod 개별 DNS 제공(StatefulSet에 자주 사용)

자주 쓰는 명령

kubectl get svc -n <ns>
kubectl describe svc <svc> -n <ns>
kubectl get endpoints -n <ns>
```
---
### 한 장으로 연결 관계
```
Namespace: 리소스들이 들어가는 “논리 공간”

Node: Pod가 실제로 실행되는 “물리/가상 머신”

Pod: 컨테이너가 실제로 돌아가는 “최소 실행 단위”

Service: 바뀌는 Pod들을 “고정 주소로 묶어주는 진입점”

흐름 예시:

Deployment가 Pod 여러 개 생성

스케줄러가 Pod를 적절한 Node에 배치

Service가 라벨로 Pod를 선택해서 고정 주소 제공

클라이언트는 Service로 접근 → 내부적으로 Pod로 분산
```

---

## 전체 목차

| 번호 | 학습 내용 |
| ---- | --------- |
| 1. | Kubernetes 아키텍처 |
| 2. | kubectl로 Pod 다루기 |
| 3. | kubectl로 ReplicaSet 다루기 |
| 4. | kubectl로 Deployment 다루기 |
| 5. | kubectl로 Service 다루기 |
| 6. | YAML 기본 문법 |
| 7. | YAML로 Pod 생성 |
| 8. | YAML로 ReplicaSet 생성 |
| 9. | YAML로 Deployment 생성 |
| 10. | YAML로 Service 생성 |

## 선언형(Declarative) vs 명령형(Imperative)
- Pods
- ReplicaSets
- Deployments
- Services

> **추가 설명**
> - 명령형은 `kubectl run`, `kubectl expose`처럼 즉시 실행 가능한 명령으로 리소스를 만드는 방식입니다.
> - 선언형은 YAML 매니페스트로 원하는 상태(desired state)를 정의하고 `kubectl apply -f`로 적용합니다.
> - 이 레포는 두 방식을 모두 다루며, 실제 운영 환경에서는 선언형 방식을 권장합니다.

## Docker 이미지 목록

| 애플리케이션 | Docker 이미지 |
| ----------- | ------------ |
| Simple Nginx V1 | stacksimplify/kubenginx:1.0.0 |
| Spring Boot Hello World API | stacksimplify/kube-helloworld:1.0.0 |
| Simple Nginx V2 | stacksimplify/kubenginx:2.0.0 |
| Simple Nginx V3 | stacksimplify/kubenginx:3.0.0 |
| Simple Nginx V4 | stacksimplify/kubenginx:4.0.0 |
| Backend Application | stacksimplify/kube-helloworld:1.0.0 |
| Frontend Application | stacksimplify/kube-frontend-nginx:1.0.0 |

## Kubernetes Fundamentals - 단계별 로드맵

### Kubernetes Architecture
- **Step-01:** Kubernetes 아키텍처 개요
- **Step-02:** Kubernetes vs AWS EKS 아키텍처 비교
- **Step-03:** Kubernetes Fundamentals 소개

### Kubernetes - kubectl로 Pod 다루기
- **Step-01:** Pod 소개 (단일/멀티 컨테이너)
- **Step-02:** Pod 생성/조회/삭제 데모
- **Step-03:** NodePort Service 개념 소개
- **Step-04:** NodePort Service로 외부 접근 데모
- **Step-05:** Pod 내부 접근 및 컨테이너 로그 확인
- **Step-06:** Pod 삭제

### Kubernetes - kubectl로 ReplicaSet 다루기
- **Step-01:** ReplicaSet 개념 소개
- **Step-02:** ReplicaSet 생성 및 스케일링
- **Step-03:** 서비스 노출 및 고가용성 테스트 후 삭제

### Kubernetes - kubectl로 Deployment 다루기
- **Step-02:** Deployment 생성 데모
- **Step-03:** set image로 이미지 업데이트
- **Step-04:** kubectl edit로 설정 수정
- **Step-05:** 롤백(Undo)로 이전 버전 복구
- **Step-06:** Deployment 일시 정지/재개

### Kubernetes - kubectl로 Service 다루기
- **Step-01:** Service 개요
- **Step-02:** Service 생성/검증 데모

### YAML Basics
- **Step-01:** 선언형 접근 소개
- **Step-02:** YAML 기본 문법 요약

### Kubernetes - YAML로 Pod 다루기
- **Step-01:** Pod 매니페스트 작성 및 적용
- **Step-02:** NodePort Service 생성 및 테스트

### Kubernetes - YAML로 ReplicaSet 다루기
- **Step-01:** ReplicaSet 매니페스트 작성 및 적용
- **Step-02:** NodePort Service 생성 및 테스트

### Kubernetes - YAML로 Deployment 다루기
- **Step-01:** Deployment 매니페스트 작성/배포/테스트

### Kubernetes - YAML로 Service 다루기
- **Step-01:** Backend 애플리케이션용 Deployment + ClusterIP Service 생성
- **Step-02:** Frontend 애플리케이션용 Deployment + NodePort Service 생성
- **Step-03:** Frontend/Backend 통합 테스트

## 이 과정에서 배우는 것
- kubectl로 Pods, ReplicaSets, Deployments, Services를 생성/관리하는 방법
- YAML로 Kubernetes 매니페스트를 작성하는 방법
- Kubernetes의 명령형/선언형 접근 방식 비교
- eksctl을 사용해 AWS EKS 클러스터를 생성하는 방법
- 실습을 통해 kubectl 주요 명령을 익히는 방법

## 사전 요구사항
- 실습을 위해 AWS 계정이 필요합니다.
- Kubernetes를 처음 접하더라도 따라올 수 있도록 기초부터 설명합니다.

## 대상 학습자
- AWS EKS로 Kubernetes를 처음 접하는 입문자
- EKS 기반 운영을 준비하는 AWS 아키텍트/시스템 관리자/개발자
