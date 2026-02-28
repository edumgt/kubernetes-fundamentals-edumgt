# Kubernetes-Fndamentals 리포지토리 정리

## 1) 기술 스택 요약
- **컨테이너/오케스트레이션**: Kubernetes, kubectl, k3s(containerd 런타임 설명 포함)
- **인프라 리소스 학습 대상**: Pod, ReplicaSet, Deployment, Service(ClusterIP/NodePort), Label/Selector
- **컨테이너 이미지/웹 계층**: Docker, Nginx(정적 페이지 및 reverse proxy 구성)
- **백엔드 애플리케이션**: Java + Spring Boot + Maven 기반 HelloWorld REST API
- **매니페스트 형식/자동화**: YAML 기반 선언형 구성(kubectl apply/create/delete)
- **문서/학습 자료 형식**: Markdown 튜토리얼, 다이어그램(png/mmd), 발표자료(pptx)

근거 파일(예시): 루트 README의 전체 커리큘럼/리소스 구성, Docker 이미지 실습 문서, Spring Boot 프로젝트(`pom.xml`), YAML 실습 디렉토리들.

---

## 2) 폴더별 Markdown 내용 요약

### `/` (루트)
- `README.md`
  - 전체 학습 로드맵을 제시하며, Kubernetes 핵심 개념(Node, Namespace, Pod, Service)과 주제별 실습 순서를 정리.
  - 선언형(Declarative) vs 명령형(Imperative) 접근을 비교하고, Docker 이미지 목록 및 단계별 실습 흐름을 안내.

### `00-Docker-Images/03-kube-frontend-nginx`
- `README.md`
  - 프런트엔드 Nginx 이미지를 Dockerfile로 빌드하고 Docker Hub에 푸시하는 절차 정리.
  - 백엔드 ClusterIP Service를 향하는 `proxy_pass` 설정 포인트 포함.

### `01-Kubernetes-Architecture`
- `README.md`
  - Kubernetes 아키텍처 개요 및 구성요소 역할을 설명.
  - k3s/containerd/kubelet 관련 동작 원리와 확인 명령(예: `crictl`, `ctr`, `systemctl`, `journalctl`)까지 운영 관점에서 상세 설명.

### `02-PODs-with-kubectl`
- `README.md`
  - `kubectl run/get/describe/delete` 중심으로 Pod 생성/조회/삭제 실습.
  - 스케줄링 위치 확인, 노드별 확인, Pod 미노출 시 점검 포인트 등 운영 트러블슈팅 내용 포함.

### `03-ReplicaSets-with-kubectl`
- `README.md`
  - ReplicaSet 개념(고가용성/복제본 유지) 소개, 생성/확인/삭제 실습, 스케일 조정 흐름 정리.
  - Deployment와의 역할 분리(ReplicaSet은 복제본 유지, 배포전략은 Deployment)도 설명.
- `kubectl_apply_vs_replace_and_pod_delete_history.md`
  - `kubectl apply` vs `kubectl replace` 차이를 실무 관점으로 비교.
  - Pod 삭제 이력 및 리소스 변경 이력 추적 방법 정리.

### `04-Deployments-with-kubectl`
- `README.md`
  - Deployment 학습 전체 범위(생성/스케일/업데이트/롤백/pause-resume) 개요.
- `04-01-CreateDeployment-Scaling-and-Expose-as-Service/README.md`
  - Deployment 생성 후 스케일링하고 Service로 노출하는 기본 실습 시나리오.
- `04-02-Update-Deployment/README.md`
  - `set image` 기반 버전 업데이트와 rollout 상태 확인 절차.
- `04-03-Rollback-Deployment/README.md`
  - revision 히스토리 조회와 이전 버전 롤백 절차.
- `04-04-Pause-and-Resume-Deployment/README.md`
  - Deployment pause/resume를 통한 변경 일시 보류 및 재개 흐름.

### `05-Services-with-kubectl`
- `README.md`
  - Backend Deployment + ClusterIP Service, Frontend Deployment + NodePort Service 조합으로 2티어 연결 실습.
  - Nginx reverse proxy 설정과 외부 접근 확인까지 포함.

### `06-YAML-Basics`
- `README.md`
  - YAML 기본 문법(키-값, 맵/리스트, 중첩 구조)과 Kubernetes 매니페스트 작성 기초를 설명.
  - Labels/Selectors의 의미와 활용 포인트를 상세 보충.

### `07-PODs-with-YAML`
- `README.md`
  - YAML 파일로 Pod를 정의/생성하고 NodePort Service로 노출하는 흐름.

### `08-ReplicaSets-with-YAML`
- `README.md`
  - YAML로 ReplicaSet 정의/생성, Pod 재생성 확인, NodePort Service 연결 실습.

### `09-Deployments-with-YAML`
- `README.md`
  - ReplicaSet 템플릿을 바탕으로 Deployment 매니페스트를 구성해 배포/서비스 노출하는 방법.

### `10-Services-with-YAML`
- `README.md`
  - Backend/Frontend Service를 YAML로 구성하고 `kubectl apply`/`delete`를 파일/폴더 단위로 다루는 운영 패턴 정리.
  - 라벨 셀렉터 활용 참고 내용 포함.

---

## 3) 한 줄 총평
이 리포지토리는 **Docker 이미지 제작 → kubectl 명령형 실습 → YAML 선언형 실습 → Deployment/Service 운영 시나리오(업데이트·롤백·중단/재개)**로 이어지는, Kubernetes 입문~기초운영 학습용 커리큘럼 구조다.
