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
- [1️⃣ Goals](#1%EF%B8%8F⃣-goals)
- [2️⃣ Architecture](#2%EF%B8%8F⃣-architecture)
- [3️⃣ Skills](#3%EF%B8%8F⃣-skills)
- [4️⃣ Project File Structure Example](#4%EF%B8%8F⃣-project-file-structure-example)
- [5️⃣ Main Flow](#5%EF%B8%8F⃣-main-flow)
- [6️⃣ Trouble Shooting](#6%EF%B8%8F⃣-trouble-shooting)

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

## 🐳 1. Docker 이미지 빌드 및 Docker Hub 업로드

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

# ❗ EXTERNAL-IP가 <pending>이면?
minikube tunnel

# 터널링 실행 후 다시 확인
kubectl get service
```

### ✅ 접속
브라우저에서 접속: `http://<EXTERNAL-IP>:8081`

### ✅ 결과 요약
항목	결과
Docker 이미지 빌드	✅ 완료
Docker Hub 업로드	✅ 완료
Kubernetes 배포 (NodePort)	✅ 2개 Pod 실행 및 접속 성공
Kubernetes 배포 (LoadBalancer)	✅ 외부 접속 가능하게 구성

