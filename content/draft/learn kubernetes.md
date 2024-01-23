---
title: "learn kubernetes"
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


---
# Terraform을 활용한 NKS 배포


[Terraform을 활용한 네이버 클라우드 플랫폼 IaC\(Infrastructure as Code\) 적용하기](https://d2.naver.com/helloworld/3612055)
[Terraform Registry](https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs/resources/nks_cluster)




---


docker in docker

 [https://applatix.com/case-docker-docker-kubernetes-part-2/](https://applatix.com/case-docker-docker-kubernetes-part-2/) 
 [https://shisho.dev/blog/posts/docker-in-docker/](https://shisho.dev/blog/posts/docker-in-docker/) 
 [https://hub.docker.com/_/docker](https://hub.docker.com/_/docker) 
 [https://pyrasis.com/jHLsAlwaysUpToDateDocker/Unit06/07](https://pyrasis.com/jHLsAlwaysUpToDateDocker/Unit06/07) 
 [https://m.blog.naver.com/isc0304/222274955992](https://m.blog.naver.com/isc0304/222274955992) 

```bash
# 사전 준비
docker network create dind-network
docker volume create docker-certs-ca
docker volume create docker-certs-client

# start a daemon instance
docker run --privileged --rm \
	--name dind-docker -d \
	--network dind-network \
	--network-alias docker \
	-e DOCKER_TLS_CERTDIR=/certs \
	-v docker-certs-ca:/certs/ca \
	-v docker-certs-client:/certs/client \
	-p 8080:80 \
	docker:dind

# connect to it from a second container
docker run --rm --network dind-network \
	-e DOCKER_TLS_CERTDIR=/certs \
	-v docker-certs-client:/certs/client:ro \
	docker:latest version

docker run -it --rm --network dind-network \
	-e DOCKER_TLS_CERTDIR=/certs \
	-v docker-certs-client:/certs/client:ro \
	docker:latest sh
```


### Docker-outside-of-Docker
```bash
kubectl apply -f - <<EOF
apiVersion: v1 
kind: Pod 
metadata: 
    name: dood 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker:1.12.6 
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        volumeMounts: 
          - mountPath: /var/run 
            name: docker-sock 
    volumes: 
      - name: docker-sock 
        hostPath: 
            path: /var/run 
EOF
```

### Docker-in-Docker
```bash
kubectl apply -f - <<EOF
apiVersion: v1 
kind: Pod 
metadata: 
    name: dind 
spec: 
    containers: 
      - name: docker-cmds 
        image: docker
        command: ['docker', 'run', '-p', '80:80', 'httpd:latest'] 
        resources: 
            requests: 
                cpu: 10m 
                memory: 256Mi 
        env: 
          - name: DOCKER_HOST 
            value: tcp://localhost:2375
      - name: dind-daemon 
        image: docker:dind
        resources: 
            requests: 
                cpu: 20m 
                memory: 512Mi
        command: ["dockerd", "--host", "tcp://127.0.0.1:2375"]
        securityContext: 
            privileged: true 
        volumeMounts: 
          - name: docker-graph-storage 
            mountPath: /var/lib/docker 
    volumes: 
      - name: docker-graph-storage 
        emptyDir: {}
EOF
```


* k8s에서 실행되는 httpd curl 확인
  ```bash
  kubectl run curl-pod --restart=Never --image=hiromasaono/curl -- curl 10.1.0.17
  kubectl logs curl-pod  
  ```





--- 

# kubernetes cli

### context
```bash
kubectl config get-contexts
kubectl config current-context
kubectl config use-context CONTEXT_NAME
```

### helm install tiller
* Delete the pre-installed helm

```bash
kubectl delete svc tiller-deploy -n kube-system
kubectl delete deploy tiller-deploy -n kube-system
```
* Install helm and tiller using this commands

```bash
helm init —client-only
helm plugin install https://github.com/rimusz/helm-tiller
helm tiller install
helm tiller start kube-system
```

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Namespace
metadata:
  name: app-one
EOF
```

```bash
kubectl config set-context for-dev@cd1 --cluster=cd1 --namespace=for-dev 
```





---
# k8s docker-desktop dashboard

## create secret manually
```bash
# secret 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: default-secret
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
EOF

kubectl edit serviceaccounts default
# 다음과 같이 secrets 추가
apiVersion: v1
kind: ServiceAccount
metadata:
  creationTimestamp: "XXXX-XX-XXTXX:XX:XXZ"
  name: default
  namespace: default
  resourceVersion: "XXXX"
  uid: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
secrets:
- name: default-secret
```

## 클러스터 어드민 롤 생성
```bash
kubectl create clusterrolebinding serviceaccounts-cluster-admin --clusterrole=cluster-admin --group=system:serviceaccounts
```

## dashboard 설치
```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

## proxy 실행
```bash
kubectl proxy
```

## default secret token 확인
```bash
kubectl get secret $(kubectl get secrets | grep default | cut -f1 -d ' ') -o jsonpath='{$.data.token}' | base64 --decode | pbcopy
```

## dashboard 접속 url
* http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/


## sample nginx 설치
### deployment 생성
```bash
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
EOF

kubectl describe deployment nginx-deployment
```

### service 생성
```bash
kubectl expose deployment/nginx-deployment --type=LoadBalancer --name=nginx-service

kubectl get service nginx-service
kubectl describe service nginx-service 
kubectl get endpointslices -l kubernetes.io/service-name=nginx-service

```

### 서비스에 접근
```bash
kubectl exec nginx-deployment-cd55c47f5-d5gsw -- printenv | grep SERVICE
kubectl scale deployment nginx-deployment --replicas=0; kubectl scale deployment nginx-deployment --replicas=2;
kubectl get pods -l app=nginx -o wide
```

### DNS
```bash
kubectl get services kube-dns --namespace=kube-system
kubectl run curl --image=radial/busyboxplus:curl -i --tty
```

## 삭제
```bash
kubectl delete services nginx-service
kubectl delete deployment nginx-deployment
```








```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```






---

 [쿠버네티스 기초 학습](https://kubernetes.io/ko/docs/tutorials/kubernetes-basics/) 

## cluster
## deployment
* application을 어떻게 생성하고 업데이트 해야 하는지를 지시
* deployment controller가 지속적으로 모니터링
* self-healing mechanism

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1

kubectl proxy
curl http://localhost:8001/version

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .itms}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME

curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/
```

## pod & node
### pod
* k8s의 최소 단위
* node에 배치

### node
* worker machine
* controller plain 에 의 해 관리

## service
* 서비스가 대상으로 하는 파드 셋은 보통 LabelSelector에 의해 결정 


deployment를 생성하고 
service 를 통해서 외부에 노출

deployment 의 replicaset 을 변경하여 스케일링 수행
desired state
