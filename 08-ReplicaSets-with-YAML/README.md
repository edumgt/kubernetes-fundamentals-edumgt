# YAML로 ReplicaSet 만들기

## Step-01: ReplicaSet 정의 작성
- **replicaset-definition.yml**
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp2-rs
spec:
  replicas: 3 # 항상 3개의 Pod 유지
  selector:  # ReplicaSet이 관리할 Pod 라벨 선택
    matchLabels:
      app: myapp2
  template:
    metadata:
      name: myapp2-pod
      labels:
        app: myapp2 # selector와 매칭되는 라벨 필요
    spec:
      containers:
      - name: myapp2
        image: stacksimplify/kubenginx:2.0.0
        ports:
          - containerPort: 80
```

## Step-02: ReplicaSet 생성
- 3개의 복제본을 유지합니다.
```
# ReplicaSet 생성
kubectl apply -f 02-replicaset-definition.yml

# ReplicaSet 목록 확인
kubectl get rs
```
- Pod를 삭제하면 ReplicaSet이 자동으로 복구합니다.
```
# Pod 목록 확인
kubectl get pods

# Pod 삭제
kubectl delete pod <Pod-Name>
```

## Step-03: ReplicaSet용 NodePort Service 생성
```yml
apiVersion: v1
kind: Service
metadata:
  name: replicaset-nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp2
  ports:
    - name: http
      port: 80
      targetPort: 80
      nodePort: 31232
```
- NodePort Service 생성 및 테스트
```
# NodePort Service 생성
kubectl apply -f 03-replicaset-nodeport-servie.yml

# NodePort Service 목록 확인
kubectl get svc

# Public IP 확인
kubectl get nodes -o wide

# 애플리케이션 접근
http://<Worker-Node-Public-IP>:<NodePort>
http://<Worker-Node-Public-IP>:31232
```

## API 레퍼런스
- **ReplicaSet:** https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#replicaset-v1-apps

## 추가 설명
- ReplicaSet은 Pod 수를 유지하지만, 롤링 업데이트/롤백은 Deployment가 담당합니다.

## 트러블슈팅 - ReplicaSet 셀렉터 불일치
- ReplicaSet이 Pod를 관리하지 못하면 `selector.matchLabels`와 `template.metadata.labels` 일치를 확인합니다.
```
# ReplicaSet 셀렉터 확인
kubectl get rs myapp2-rs -o yaml

# Pod 소유자 확인
kubectl get pods -o yaml
```
