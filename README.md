# Kubernetes Fundamentals

## 목차

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
