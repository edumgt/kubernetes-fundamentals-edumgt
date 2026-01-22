# YAML로 Pod 만들기

## Step-01: Kubernetes YAML 최상위 객체
- Kubernetes 매니페스트의 기본 구조를 확인합니다.
- **01-kube-base-definition.yml**
```yml
apiVersion:
kind:
metadata:

spec:
```
- **Pod API 오브젝트 레퍼런스:** https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#pod-v1-core

## Step-02: 간단한 Pod 정의 작성
- 기본적인 Pod 매니페스트를 작성합니다.
- **02-pod-definition.yml**
```yml
apiVersion: v1 # String
kind: Pod  # String
metadata: # Dictionary
  name: myapp-pod
  labels: # Dictionary
    app: myapp
spec:
  containers: # List
    - name: myapp
      image: stacksimplify/kubenginx:1.0.0
      ports:
        - containerPort: 80
```
- **Pod 생성**
```
# Pod 생성
kubectl create -f 02-pod-definition.yml
[or]
kubectl apply -f 02-pod-definition.yml

# Pod 목록 확인
kubectl get pods
```

## Step-03: NodePort Service 생성
- **03-pod-nodeport-service.yml**
```yml
apiVersion: v1
kind: Service
metadata:
  name: myapp-pod-nodeport-service  # Service 이름
spec:
  type: NodePort
  selector:
  # 이 라벨과 매칭되는 Pod로 트래픽 분산
    app: myapp
  # Service가 수신할 포트
  ports:
    - name: http
      port: 80       # Service 포트
      targetPort: 80 # 컨테이너 포트
      nodePort: 31231 # NodePort
```
- **NodePort Service 생성**
```
# Service 생성
kubectl apply -f 03-pod-nodeport-service.yml

# Service 목록 확인
kubectl get svc

# Public IP 확인
kubectl get nodes -o wide

# 애플리케이션 접근
http://<WorkerNode-Public-IP>:<NodePort>
http://<WorkerNode-Public-IP>:31231
```

## API 오브젝트 레퍼런스
- **Pod:** https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#pod-v1-core
- **Service:** https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/#service-v1-core

## 최신 API 오브젝트 레퍼런스
- **Pod:** https://kubernetes.io/docs/reference/kubernetes-api/workload-resources/pod-v1/
- **Service:** https://kubernetes.io/docs/reference/kubernetes-api/service-resources/service-v1/
- **Kubernetes API Reference:** https://kubernetes.io/docs/reference/kubernetes-api/

## 추가 설명
- 매니페스트의 `metadata.labels`와 Service의 `spec.selector`가 일치해야 트래픽이 정상 분배됩니다.
- NodePort는 학습/테스트용으로 유용하지만, 운영에서는 Ingress/LoadBalancer가 일반적입니다.

## 트러블슈팅 - 라벨/셀렉터 불일치
- Service는 생성됐지만 트래픽이 전달되지 않는다면 라벨 매칭을 확인합니다.
```
# Pod 라벨 확인
kubectl get pod myapp-pod -o yaml

# Service 셀렉터 확인
kubectl get service myapp-pod-nodeport-service -o yaml
```
