# Kubernetes - Deployment 다루기

## 다루는 주제
1. Deployment 생성
2. Deployment 스케일링
3. Deployment를 Service로 노출
4. Deployment 업데이트
5. Deployment 롤백
6. 롤링 재시작(Rolling Restart)
7. Deployment 일시정지/재개
8. 카나리 배포(Declarative 섹션에서 다룸)

## Deployment 설명
- **정의**
  - Deployment는 ReplicaSet과 Pod를 선언적으로 관리하는 상위 리소스입니다.
  - 원하는 상태(desired state)를 선언하면 Kubernetes가 실제 상태를 지속적으로 맞춥니다.
- **핵심 역할**
  - 롤링 업데이트와 롤백을 통한 무중단 배포 관리
  - 스케일링(복제 수 조절) 자동화
  - 장애 발생 시 원하는 Pod 수를 유지하도록 자동 복구
- **주요 구성 요소**
  - `spec.replicas`: 원하는 Pod 개수
  - `spec.selector`: Deployment가 관리할 Pod를 식별하는 라벨 셀렉터
  - `spec.template`: Pod 템플릿(컨테이너 이미지/포트/환경변수 등)
  - `spec.strategy`: 업데이트 전략(RollingUpdate/Recreate)
- **운영 포인트**
  - 라벨/셀렉터가 일치해야 Deployment가 Pod를 정확히 관리합니다.
  - 이미지 태그 변경은 롤링 업데이트 트리거의 핵심입니다.

## 추가 설명
- **Deployment란?**
  - ReplicaSet을 관리하는 상위 리소스입니다.
  - 선언형 업데이트(롤링 업데이트), 롤백, 스케일링을 지원합니다.
- **왜 Deployment를 쓰는가?**
  - ReplicaSet을 직접 관리하지 않아도 되어 운영이 단순해집니다.
  - 배포 실패 시 빠른 롤백이 가능해 안정성이 높습니다.
