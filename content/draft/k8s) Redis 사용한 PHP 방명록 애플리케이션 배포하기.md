---
title: "k8s) Redis 사용한 PHP 방명록 애플리케이션 배포하기"
categories:
  - "development"
tags:
  - "kubernetes"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---

* [예시: Redis를 사용한 PHP 방명록 애플리케이션 배포하기 | Kubernetes](https://kubernetes.io/ko/docs/tutorials/stateless-application/guestbook/)
* [Redis 및 PHP로 방명록 만들기  |  Kubernetes Engine  |  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/tutorials/guestbook)
  ![](/images/Pasted%20image%2020240122194312.png)

## redis 데이타베이스를 실행
### redis deployment 생성
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: leader
        tier: backend
    spec:
      containers:
      - name: leader
        image: "docker.io/redis:6.2.7"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
EOF

kubectl get deployments
```

## redis leader 서비스 생성하기
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: redis-leader
  labels:
    app: redis
    role: leader
    tier: backend
spec:
  ports:
  - port: 6379
    targetPort: 6379
  selector:
    app: redis
    role: leader
    tier: backend
EOF

kubectl get service
```

### redis follower 설정
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
        role: follower
        tier: backend
    spec:
      containers:
      - name: follower
        image: gcr.io/google_samples/gb-redis-follower:v2
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
EOF

kubectl get service
```

### redis follower 서비스 생성
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: redis-follower
  labels:
    app: redis
    role: follower
    tier: backend
spec:
  ports:
    # the port that this service should serve on
  - port: 6379
  selector:
    app: redis
    role: follower
    tier: backend
EOF

kubectl get service
```


## 방명록 프론트엔드를 설정하고 노출하기
### 방명록 프론트엔드 디플로이먼트 생성
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
        app: guestbook
        tier: frontend
  template:
    metadata:
      labels:
        app: guestbook
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v5
        env:
        - name: GET_HOSTS_FROM
          value: "dns"
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 80
EOF
```

### 방명록 프론트엔드 서비스 생성
```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # if your cluster supports it, uncomment the following to automatically create
  # an external load-balanced IP for the frontend service.
  # type: LoadBalancer
  type: LoadBalancer
  ports:
    # the port that this service should serve on
  - port: 80
  selector:
    app: guestbook
    tier: frontend
EOF
```

### 프론트엔드 서비스 확인하기
```bash
kubectl port-forward svc/frontend 8080:80
```

## 정리하기
```bash
kubectl delete deployment -l app=redis
kubectl delete service -l app=redis
kubectl delete deployment frontend
kubectl delete service frontend
```


























