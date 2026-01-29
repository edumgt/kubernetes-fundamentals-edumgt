# kubectl apply vs replace 차이 + Pod 삭제 이력/삭제된 Pod 추적 방법

작성 환경 가정: k8s/k3s 공통(차이점은 별도 표시)

---

## 1) `kubectl apply` 와 `kubectl replace` 핵심 차이

### 한 줄 요약
- **apply = 선언적(merge/patch) 업데이트**: “변경분만 반영”
- **replace = 통째 교체(overwrite)**: “파일 내용으로 전체를 갈아끼움”

---

## 2) `kubectl apply -f ...` 동작 방식

### 기본 개념
- 리소스가 없으면 생성(create)
- 리소스가 있으면 **필요한 부분만 patch/merge**로 업데이트

### 특징
- 일반적으로 리소스에 `kubectl.kubernetes.io/last-applied-configuration` 어노테이션을 남겨
  이후 apply에서 “이전 적용 상태 대비 변경점”을 기반으로 업데이트합니다.
- 운영/GitOps/반복 배포에 가장 많이 쓰는 방식

### 장점
- 변경 필요한 필드만 바뀌어 상대적으로 안전
- 사람이 수동으로 추가한 일부 설정과 충돌이 덜함
- 반복 적용해도 안정적(멱등성에 가까움)

### 주의(오해 포인트)
- YAML에서 “삭제한 필드”가 항상 100% 깔끔하게 제거되는 건 아닐 수 있음(필드 성격/필드 매니저 충돌 등)
- 과거에 `kubectl create`로 만든 리소스를 `apply`하면
  “last-applied 어노테이션이 없다”는 경고가 뜰 수 있으나 보통 자동으로 패치됨

---

## 3) `kubectl replace -f ...` 동작 방식

### 기본 개념
- 리소스가 있으면 **파일 내용으로 리소스를 통째로 교체**(overwrite 성격)
- 리소스가 없으면 기본적으로 실패(에러)

### 장점
- “정확히 파일에 있는 상태로 맞추겠다”가 목적일 때 직관적
- apply로 정리되지 않는 일부 케이스에서 강제로 맞추는 데 유용

### 주의(매우 중요)
- YAML에 없는 필드가 사라질 수 있음  
  예: 다른 사람이 `kubectl annotate`/`kubectl label`로 임시로 넣은 값, 특정 컨트롤러가 추가한 부가 설정 등
- `kubectl replace --force`는 사실상 **delete 후 create**와 유사  
  - 오브젝트 UID가 바뀜
  - 짧은 순간이라도 리소스가 “없던 상태”가 생길 수 있음(특히 Service/Ingress 등 영향)
  - 연결이 잠깐 끊길 수 있음

---

## 4) 실무 추천 기준

### 대부분의 운영/배포 = `apply`
- Deployment/Service/Ingress/HPA/ConfigMap/Secret 등 대다수 리소스
- Git에 있는 YAML을 “원하는 상태”로 계속 유지하는 방식

예:
```bash
kubectl apply -f .
```

### 제한적으로 `replace`를 고려
- “파일에 없는 설정이 남아서 정리가 안 된다”거나 “정확히 파일대로만 맞춰야 한다”는 특수 목적
- 특히 `--force`는 운영 환경에서 주의(삭제→재생성급 영향)

---

## 5) Pod 삭제 이력 조회 / RS 삭제로 사라진 Pod를 다시 조회 가능한가?

### 한 줄 결론
- **삭제된 Pod 오브젝트를 API에서 ‘다시 조회’하는 것은 기본적으로 불가능**합니다.
- 대신 **Events / Audit Log / 컨트롤플레인 로그 / 외부 로깅/모니터링**으로 “흔적(이력)”을 추적합니다.

---

## 6) Pod 삭제 이력을 확인하는 방법(가능한 것부터)

### 6.1 Events로 추적 (가장 쉬움, 단 보관 기간 짧음)
> 이벤트는 시간이 지나면 사라질 수 있습니다.

```bash
# 네임스페이스 내 최근 이벤트 시간순
kubectl -n <ns> get events --sort-by=.lastTimestamp | tail -n 50

# 전체 네임스페이스 최근 이벤트
kubectl get events -A --sort-by=.lastTimestamp | tail -n 200
```

Pod 이름을 알고 있다면:
```bash
kubectl -n <ns> get events   --field-selector involvedObject.name=<pod-name>   --sort-by=.lastTimestamp
```

---

### 6.2 RS/Deployment 관점으로 “무슨 일이 있었는지” 추적
RS가 삭제되면 그 하위 Pod도 같이 삭제될 수 있습니다.

```bash
kubectl -n <ns> get deploy,rs -o wide
kubectl -n <ns> describe rs <rs-name>
kubectl -n <ns> get events --sort-by=.lastTimestamp | tail -n 100
```

---

### 6.3 (가장 확실) Audit Log가 켜져 있으면 “누가/언제 delete 했는지” 확인 가능
- 클러스터에 Audit 로그가 활성화되어 있어야만 가능합니다.
- 관리형(EKS/GKE/AKS) 또는 보안 강화된 온프렘에서 종종 사용

> Audit이 꺼져 있었다면 과거 이력은 복구 불가에 가깝습니다.

---

### 6.4 k3s 환경에서 로그로 단서 찾기 (kubelet.service가 없음)
k3s는 kubelet이 별도 systemd 서비스가 아니라 `k3s`/`k3s-agent`에 통합됩니다.

```bash
# control-plane 노드
sudo journalctl -u k3s -n 300 --no-pager

# worker 노드
sudo journalctl -u k3s-agent -n 300 --no-pager
```

키워드로 좁히기:
```bash
sudo journalctl -u k3s --no-pager | grep -iE "delete|deleted|replicaset|pod/" | tail -n 50
```

> 보통 “삭제 사실/에러 단서”는 얻을 수 있어도, Pod의 전체 스펙을 완전히 복원할 정도의 정보는 남지 않는 경우가 많습니다.

---

### 6.5 외부 로깅/모니터링이 있으면 “Pod는 없어도 데이터는 남음”
- Loki/ELK/OpenSearch 등에 앱 로그가 남아 있으면 “해당 Pod가 존재했던 시점의 로그”는 조회 가능
- Prometheus가 있으면 “해당 시간대의 CPU/메모리 사용량” 등은 확인 가능

즉, **Pod 리소스 오브젝트 복구는 불가**해도,
**운영 데이터로 흔적 복구**는 가능합니다.

---

## 7) “RS 삭제로 날아간 Pod”를 다시 조회 대신, 다시 만들기(재생성)
조회는 못 하지만, 같은 리소스를 다시 적용하면 새 Pod가 생성됩니다.

- Deployment가 있었다면: Deployment YAML을 다시 `apply`
- RS만 있었다면: RS YAML을 다시 `apply`

예:
```bash
kubectl apply -f deployment.yaml
# 또는
kubectl apply -f replicaset.yaml
```

---

## 8) 빠른 체크리스트(현장용)

### “왜 Pod가 갑자기 없어졌지?” 3단계
1) 해당 NS 이벤트:
```bash
kubectl -n <ns> get events --sort-by=.lastTimestamp | tail -n 200
```

2) 컨트롤러 상태(Deployment/RS):
```bash
kubectl -n <ns> get deploy,rs -o wide
```

3) (k3s) 노드/컨트롤플레인 로그:
```bash
sudo journalctl -u k3s -n 300 --no-pager
```

---

## 9) 요약
- **apply**: 변경분만 반영(대부분 추천)
- **replace**: 파일 내용으로 통째 교체(주의, `--force`는 삭제→재생성급)
- **삭제된 Pod 재조회**: 기본적으로 불가
- **삭제 이력 추적**: Events(짧음) → Audit Log(있으면 확실) → k3s/journalctl → 외부 로그/메트릭

---
