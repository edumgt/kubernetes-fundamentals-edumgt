# Kubernetes - Service 다루기

## Step-01: Service 소개
- **Service 유형**
  1. ClusterIP
  2. NodePort
  3. LoadBalancer
  4. ExternalName
- 이 섹션에서는 ClusterIP와 NodePort를 자세히 다룹니다.
- LoadBalancer는 클라우드 제공자마다 동작이 달라, 해당 환경별 섹션에서 다룹니다.
- ExternalName은 명령형 생성이 없어 YAML로 정의해야 하며, 필요 시 추가 설명합니다.

## Step-02: ClusterIP Service - Backend 애플리케이션 구성
- Backend(Spring Boot REST) Deployment 생성
- Backend를 위한 ClusterIP Service 생성
```
# Backend REST 앱 Deployment 생성
kubectl create deployment my-backend-rest-app --image=stacksimplify/kube-helloworld:1.0.0
kubectl get deploy

# Backend REST 앱 ClusterIP Service 생성
kubectl expose deployment my-backend-rest-app --port=8080 --target-port=8080 --name=my-backend-service
kubectl get svc
Observation: 기본값이 ClusterIP이므로 --type=ClusterIp 옵션을 생략해도 됩니다.
```
- **중요:** 컨테이너 포트(8080)와 서비스 포트(8080)가 동일하면 `--target-port`는 생략 가능합니다.

- **Backend HelloWorld 소스** [kube-helloworld](../00-Docker-Images/02-kube-backend-helloworld-springboot/kube-helloworld)

## Step-03: NodePort Service - Frontend 애플리케이션 구성
- 기존에도 NodePort를 사용해봤지만, ClusterIP와 함께 전체 아키텍처 관점에서 다시 구성합니다.
- Frontend(Nginx Reverse Proxy) Deployment 생성
- Frontend를 위한 NodePort Service 생성
- **중요:** Nginx reverse proxy 설정에서 backend 서비스 이름 `my-backend-service`가 정확해야 합니다.

### Nginx 설정 예시
```conf
server {
    listen       80;
    server_name  localhost;
    location / {
    # Backend ClusterIP Service 이름과 포트 지정
    # proxy_pass http://<Backend-ClusterIp-Service-Name>:<Port>;
    proxy_pass http://my-backend-service:8080;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```
- **Docker 이미지 위치:** https://hub.docker.com/repository/docker/stacksimplify/kube-frontend-nginx
- **Frontend 소스** [kube-frontend-nginx](../00-Docker-Images/03-kube-frontend-nginx)
```
# Frontend Nginx Proxy Deployment 생성
kubectl create deployment my-frontend-nginx-app --image=stacksimplify/kube-frontend-nginx:1.0.0
kubectl get deploy

# Frontend Nginx Proxy NodePort Service 생성
kubectl expose deployment my-frontend-nginx-app --type=NodePort --port=80 --target-port=80 --name=my-frontend-service
kubectl get svc

# 접근 정보 확인
kubectl get svc
kubectl get nodes -o wide
http://<node1-public-ip>:<Node-Port>/hello

# Backend를 10개로 스케일링
kubectl scale --replicas=10 deployment/my-backend-rest-app

# 다시 접근해 로드밸런싱 확인
http://<node1-public-ip>:<Node-Port>/hello
```

## 추가 설명
- ClusterIP는 내부 통신의 기본이며, 마이크로서비스 간 안정적인 서비스 디스커버리를 제공합니다.
- NodePort는 고정 포트를 열기 때문에 보안 그룹/방화벽 규칙도 함께 확인해야 합니다.

# Kubernetes Service: LoadBalancer / ExternalName 상세 정리

아래 내용은 기존 Service 유형 소개(ClusterIP/NodePort/LoadBalancer/ExternalName) 중, **LoadBalancer**와 **ExternalName**을 실무 관점에서 조금 더 자세히 정리한 문서입니다.

---

## 1) LoadBalancer Service (외부 트래픽을 “클라우드 LB”로 받아서 Service로 전달)

### 핵심 개념
- `type: LoadBalancer` 는 **클러스터 밖에서 들어오는 트래픽**을 “클라우드 제공자의 로드밸런서(ELB/NLB/ALB, Azure LB, GCP LB 등)”가 받아서
- 그 트래픽을 **Kubernetes Service(대개 NodePort 경유)** 로 전달하는 방식입니다.
- `kubectl get svc` 에서 **EXTERNAL-IP(또는 hostname)** 가 생기며, 이것이 외부 진입점이 됩니다.

### 내부 동작 흐름(일반적인 형태)
1. Service 생성(`type: LoadBalancer`)
2. 클라우드 컨트롤러(또는 로드밸런서 컨트롤러)가 감지
3. 클라우드에 LB 생성 + 헬스체크/리스너/타겟그룹(환경에 따라) 구성
4. LB → (노드로) 전달 → kube-proxy가 Pod로 라우팅

> 구현은 클라우드별로 다릅니다. 같은 “LoadBalancer”라도 **AWS/GCP/Azure/온프렘(MetalLB)/k3s**에서 “생성되는 LB의 종류, 헬스체크, 소스IP 보존, 비용/제약”이 달라질 수 있습니다.

### 언제 쓰나?
- Ingress 없이도 **단일 서비스를 바로 외부에 노출**해야 할 때 (예: 단독 API, 테스트용)
- L4(TCP/UDP) 레벨로 외부 노출이 필요한 경우
- 운영에서는 보통 **Ingress(HTTP/HTTPS) + LoadBalancer(ingress-controller 앞단)** 조합이 흔합니다.  
  (즉 “앱마다 LoadBalancer”보다 “Ingress Controller 하나만 LoadBalancer”로 빼는 형태)

### 실전에서 자주 보는 설정 포인트

#### (1) `externalTrafficPolicy`
- `Cluster`(기본): LB가 어떤 노드로 보내도 kube-proxy가 다른 노드의 Pod로 넘길 수 있음  
  - 장점: 분산이 쉬움  
  - 단점: **클라이언트 원본 IP가 보존되지 않을 수 있음(SNAT)**
- `Local`: 트래픽을 받은 노드에 “로컬 Pod”이 있을 때만 전달  
  - 장점: **원본 IP 보존 가능**, 불필요 홉 감소  
  - 단점: 노드에 Pod이 없으면 드롭될 수 있어 **배포/스케일 전략**을 같이 봐야 함

#### (2) 클라우드별 “LB 옵션”은 대부분 Annotation으로 제어
- AWS면 NLB/ALB 타입, 내부용(internal), cross-zone, proxy-protocol 등
- GCP/Azure도 내부/외부, 정적 IP, 헬스체크 방식 등  
→ 따라서 LoadBalancer는 “Service YAML 하나”로 끝나지 않고 **환경별 annotation/컨트롤러**가 같이 따라옵니다.

#### (3) 방화벽/보안그룹
- NodePort와 달리 “외부로부터 들어오는 포트”는 LB 리스너 기준이지만,
- 헬스체크/노드 타겟 포트 등으로 인해 **노드 쪽 보안그룹도 같이 열어야 하는 케이스**가 생깁니다(환경에 따라).

### 예시 YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-frontend-lb
spec:
  type: LoadBalancer
  selector:
    app: my-frontend-nginx-app
  ports:
    - name: http
      port: 80        # 외부에서 접속할 포트(서비스 포트)
      targetPort: 80  # Pod 컨테이너 포트
  externalTrafficPolicy: Cluster
```

> 온프렘/로컬(k3s, kind, bare metal)에서는 “클라우드 LB가 없으니” `EXTERNAL-IP`가 안 뜨는 게 정상입니다.  
> 이때는 **MetalLB**(bare metal)나 **k3s의 servicelb(klipper-lb)** 같은 구성요소가 있어야 “LoadBalancer가 동작”합니다.

---

## 2) ExternalName Service (K8s Service를 “외부 DNS 이름(CNAME)”으로 매핑)

### 핵심 개념
- `type: ExternalName` 은 **프록시/로드밸런싱을 하지 않습니다.**
- 대신, 클러스터 DNS(CoreDNS)가  
  `my-external-svc.default.svc.cluster.local` 같은 이름을 조회하면
  **외부 도메인으로 CNAME 응답**을 돌려주는 “DNS 별칭 서비스”입니다.

즉,
- ClusterIP/NodePort/LoadBalancer처럼 **selector로 Pod을 고르거나**
- kube-proxy가 트래픽을 **Pod로 라우팅**하지 않습니다.

### 언제 쓰나?
- 마이크로서비스 코드/설정에서 “접속 대상”을 바꾸기 어렵거나,
- 내부 서비스처럼 보이게 하되 실제로는 외부 시스템(관리형 DB, 외부 API, 레거시 서버 등)으로 보내고 싶을 때
- 환경별(DEV/PROD)로 외부 endpoint만 바꿔야 할 때  
  → 앱 설정은 `http://some-service`로 고정하고, ExternalName의 `externalName:`만 환경별로 바꾸는 식

### 중요한 제약/주의사항
1. **포트/프로토콜 라우팅 기능 없음**
   - DNS만 바꿔줄 뿐이라 “80으로 들어오면 8080으로 보내줘” 같은 건 못합니다.
   - 앱이 `externalName`으로 받은 호스트에 대해 **직접 포트까지 포함해서** 호출해야 합니다.

2. **IP를 직접 넣을 수 없음**
   - `externalName`에는 보통 **DNS 이름**이 들어갑니다(CNAME).
   - “외부 IP로 보내고 싶다”면 ExternalName이 아니라 다른 방법(EndpointSlice/Headless 등)을 고려합니다.

3. **kube-proxy/LB/세션/헬스체크 같은 기능 없음**
   - “쿠버네티스가 외부 엔드포인트를 로드밸런싱”해주지 않습니다.
   - 그냥 DNS alias이기 때문에, 로드밸런싱이 필요하면 **외부에서 LB를 제공**해야 합니다.

### 예시 YAML
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-external-api
spec:
  type: ExternalName
  externalName: api.example.com
```

Pod 내부에서 아래처럼 호출하면:
- `http://my-external-api` → DNS가 `api.example.com`으로 CNAME 해석 → 외부로 나감

> (실무 팁) **TLS 인증서 SNI/Host 헤더**가 중요한 외부 API라면,  
> ExternalName으로 별칭을 주는 것이 오히려 문제를 만들 수도 있습니다(요청 Host가 무엇인지, 인증서 도메인이 무엇인지).

---

## 정리: 언제 무엇을 쓰나?
- **ClusterIP**: 클러스터 내부 통신 기본 (서비스 디스커버리 + 로드밸런싱)
- **NodePort**: 노드 포트를 고정 오픈해서 외부 진입(간단하지만 보안/운영 부담)
- **LoadBalancer**: “외부 LB를 공식 진입점”으로(클라우드/온프렘 구성에 따라 구현 상이)
- **ExternalName**: “K8s 서비스 이름 → 외부 DNS” 별칭(DNS만, 프록시/라우팅 없음)
