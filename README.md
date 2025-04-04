<img src="https://capsule-render.vercel.app/api?type=waving&color=64b5f6&height=300&section=header&text=SpringBoot-on-Kubernetes&fontSize=50&fontColor=FFFFFF&animation=fadeIn&width=1200" width="1200" />

# 🚩 프로젝트 개요
Spring Boot 애플리케이션을 Docker로 이미지화하고,  

Docker Hub에 푸시한 뒤 Kubernetes에 NodePort와 LoadBalancer 방식으로 배포하는 프로젝트입니다.


<br>

|<img src="https://github.com/DoomchitYJ.png" width="220" />|<img src="https://github.com/imhaeunim.png" width="220" />|<img src="https://github.com/jinhyunpark929.png" width="220" />|<img src="https://github.com/letmeloveyou82.png" width="220" />|
|:-:|:-:|:-:|:-:|
|박영진<br/>[@DoomchitYJ](https://github.com/DoomchitYJ)|임하은<br/>[@imhaeunim](https://github.com/imhaeunim)|박진현<br/>[@jinhyunpark929](https://github.com/jinhyunpark929)|최윤정<br/>[@letmeloveyou82](https://github.com/letmeloveyou82)|

<br>

## 📍 Contents
- [🔧 기술 스택](#-기술-스택)
- [📁 프로젝트 구성](#-프로젝트-구성)
- [🐳 1. Docker Hub 업로드](#-1-docker-hub-업로드)
- [🚀 2. NodePort 방식으로 Kubernetes 배포](#-2-nodeport-방식으로-kubernetes-배포)
- [☁️ 3. LoadBalancer 방식으로 Kubernetes 배포](#%EF%B8%8F-3-loadbalancer-방식으로-kubernetes-배포)

<br>


## 🔧 기술 스택
<div>
  <img src="https://img.shields.io/badge/ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white">
  <img src="https://img.shields.io/badge/springboot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white">
  <img src="https://img.shields.io/badge/gradle-02303A?style=for-the-badge&logo=gradle&logoColor=white">
  
  <img src="https://img.shields.io/badge/docker-2496ED?style=for-the-badge&logo=docker&logoColor=white">
  <img src="https://img.shields.io/badge/kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white">
  <img src="https://img.shields.io/badge/yaml-CB171E?style=for-the-badge&logo=yaml&logoColor=white">
</div>

<br>

## 📁 프로젝트 구성
```bash
├── Dockerfile                 # Spring Boot 애플리케이션을 위한 Docker 이미지 설정
├── step07_cicd-0.0.1-SNAPSHOT.jar  # 빌드된 Spring Boot 애플리케이션 JAR 파일
├── myjar-nodeport.yaml        # Kubernetes 배포(NodePort 방식) 설정 파일
└── myjar-loadbalancer.yaml    # Kubernetes 배포(LoadBalancer 방식) 설정 파일
```

<br>

## 🐳 1. Docker Hub 업로드

### ✅ Dockerfile 예시

```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY step07_cicd-0.0.1-SNAPSHOT.jar app.jar
RUN mkdir -p /app/logs
CMD ["sh", "-c", "java -jar app.jar > /app/logs/app.log 2>&1"]
```

### ✅ 이미지 빌드 & 업로드
```bash
# 이미지 빌드
docker build -t <docker-hub-사용자이름>/myjar:1.0 .

# Docker Hub 로그인
docker login

#이미지 푸시
docker push <docker-hub-사용자이름>/myjar:1.0
```
<br>

## 🚀 2. NodePort 방식으로 Kubernetes 배포
### ✅ myjar-nodeport.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myjar-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myjar
  template:
    metadata:
      labels:
        app: myjar
    spec:
      containers:
      - name: myjar
        image: <docker-hub-사용자이름>/myjar:1.0
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: myjar-service
spec:
  selector:
    app: myjar
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
      nodePort: 30080
  type: NodePort

```

### ✅ 배포 및 확인
```bash
kubectl apply -f myjar-nodeport.yaml

kubectl get all

minikube ip
```

### ✅ 브라우저에서 접속
브라우저에서 접속: `http://<minikube-ip>:30080`

<br>

## ☁️ 3. LoadBalancer 방식으로 Kubernetes 배포
### ✅ myjar-loadbalancer.yaml
```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: myjar02-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myjar02
  template:
    metadata:
      labels:
        app: myjar02
    spec:
      containers:
      - name: myjar02
        image: <docker-hub-사용자이름>/myjar:1.0
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: myjar02-service
spec:
  type: LoadBalancer
  selector:
    app: myjar02
  ports:
    - protocol: TCP
      port: 8081
      targetPort: 80
```

### ✅ 배포 및 확인
```bash
kubectl apply -f myjar-loadbalancer.yaml

kubectl get all

```


**🚫 EXTERNAL-IP가 <pending>이면?** 
<br>
minikube의 경우 기본적으로 LoadBalancer를 사용할 수 없음
- LoadBalancer 타입의 서비스는 클라우드 환경(예: AWS, GCP, Azure)에서
클라우드 Load Balancer 리소스를 자동으로 생성 및 연결해줍니다.
- Minikube는 로컬 환경(Docker/VM 기반) 으로 클러스터를 구성하므로,
외부 IP를 가진 LoadBalancer를 프로비저닝할 수 있는 인프라가 없음 -> EXTERNAL-IP는 <pending> 상태로 남음

<br>

**👉 minikube tunnel 명령어를 사용하면 Minikube가 가상의 LoadBalancer를 터널링 방식으로 흉내내어 EXTERNAL-IP가 할당**

```bash
minikube tunnel

# 터널링 실행 후 다시 확인
kubectl get service
```



### ✅ 접속
브라우저에서 접속: `http://<EXTERNAL-IP>:8081`

