---
title: "AWS Route53 + ApiGateway 설정 과정"
categories:
  - "development"
tags:
  - "aws"
  - "apigateway"
  - "route53"
date: "2021-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---


### 도메인 등록
![](/images/Pasted%20image%2020240122205518.png)

### 인증서 발급
![](/images/Pasted%20image%2020240122205535.png)

### apigateway 사용자 도메인 설정 및 api 매핑
![](/images/Pasted%20image%2020240122205549.png)

### Route53 A 레코드 이름 등록
![](/images/Pasted%20image%2020240122205606.png)


### 연결까지 성공 그러나 request context 정보의 endpoint도 지정되 도메인으로 변경
* ~~api management를 위한 endpoint를 별도 지정 하거나 context에서 정보가 있는지 확인 필요~~
* api management endpoint 생성 방법을 appid, region, stage 정보를 가지고 생성


### Route53 레코드 생성
* 라우팅 정책을 지리적 위치로 설정
* 라우팅 대상을 위헤 apigateway에서 사용자 지정 도메인 이름을 등록 하면 라투팅 대상에서 검색 가능
  ![](/images/Pasted%20image%2020240122205635.png)


### Route53 설정 후 테스트
* 현재 PC에서 연결 -> 서울 리전 접속 확인
* 오레콘 리전에 ec2를 생성하여 연결 -> 버지니아 접속 확인
* 확인은 Dynamodb에 연결 정보를 기준으로 확인
* 버지니아 리전, 오레곤 리전, 도쿄 리전, 작업 laptop 에서 접속
  ![](/images/Pasted%20image%2020240122205828.png)
* 동일 pubsub.[domain] 도메인으로 접속 하여 각각 가까운 region에 접속한 것을 아래 테이블에서 확인


### Route53 최종 설정 정보
![](/images/Pasted%20image%2020240122205852.png)


### 주의 사항
* apigateway에 사용자 지정 도메인 API 매핑은 websocket, http를 같이 사용 할 수 없다.



