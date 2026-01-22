# YAML로 Service 만들기

## Step-01: Service 소개
- 프론트엔드/백엔드 예제로 아래 두 가지 Service를 다룹니다.
  - NodePort Service
  - ClusterIP Service

## Step-02: Backend Deployment & ClusterIP Service 생성
- Backend REST 애플리케이션 Deployment 템플릿 작성
- Backend REST 애플리케이션 ClusterIP Service 템플릿 작성
- **중요 메모:**
  - ClusterIP Service 이름은 `name: my-backend-service`여야 합니다.
  - Frontend Nginx reverse proxy의 `default.conf`에서 같은 이름을 사용합니다.
  - 다른 이름으로 바꿔보며 어떤 문제가 발생하는지 확인해 보세요.
  - 관련 설명은 [05-Services-with-kubectl](/05-Services-with-kubectl/README.md)에서도 다룹니다.
```
cd <Course-Repo>\kubernetes-fundamentals\10-Services-with-YAML\kube-manifests
kubectl get all
kubectl apply -f 01-backend-deployment.yml -f 02-backend-clusterip-service.yml
kubectl get all
```

## Step-03: Frontend Deployment & NodePort Service 생성
- Frontend Nginx 애플리케이션 Deployment 템플릿 작성
- Frontend Nginx 애플리케이션 NodePort Service 템플릿 작성
```
cd <Course-Repo>\kubernetes-fundamentals\10-Services-with-YAML\kube-manifests
kubectl get all
kubectl apply -f 03-frontend-deployment.yml -f 04-frontend-nodeport-service.yml
kubectl get all
```
- **REST 애플리케이션 접근**
```
# 노드 외부 IP 확인
kubectl get nodes -o wide

# REST 애플리케이션 접근 (Frontend Service에 31234 고정 포트 사용)
http://<node1-public-ip>:31234/hello
```

## Step-04: kubectl apply로 삭제/재생성

### 파일 단위 삭제
```
kubectl delete -f 01-backend-deployment.yml -f 02-backend-clusterip-service.yml -f 03-frontend-deployment.yml -f 04-frontend-nodeport-service.yml
kubectl get all
```

### 폴더 단위 생성
```
cd <Course-Repo>\kubernetes-fundamentals\10-Services-with-YAML
kubectl apply -f kube-manifests/
kubectl get all
```

### 폴더 단위 삭제
```
cd <Course-Repo>\kubernetes-fundamentals\10-Services-with-YAML
kubectl delete -f kube-manifests/
kubectl get all
```

## 추가 참고 자료 - 라벨 셀렉터 활용
- https://kubernetes.io/docs/concepts/cluster-administration/manage-deployment/#using-labels-effectively
- https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors

## 추가 설명
- Frontend/Backend 통신은 Service 이름 기반의 DNS로 연결됩니다.
- 잘못된 Service 이름은 502/504 에러의 주요 원인이 됩니다.

## 트러블슈팅 - Service 이름 확인
- Frontend에서 502/504가 발생하면 Backend Service 이름과 프록시 설정을 확인합니다.
```
kubectl get svc
kubectl get service my-backend-service -o yaml
```
