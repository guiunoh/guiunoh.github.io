---
title: "AWS Apigateway Private Endpoint"
categories:
  - "development"
tags:
  - "aws"
  - "apigateway"
date: "2021-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---

### 구성
![](/images/Pasted%20image%2020240122210010.png)


### 같은 리전에서 private link
* API Gateway execute-api용 인터페이스 VPC endpoint 생성
  * https://console.aws.amazon.com/vpc
  * Endpoint 생성
    ![](/images/Pasted%20image%2020240122210026.png)
* VPC DNS 호스트 이름이 체크 되어야 한다.
  ![](/images/Pasted%20image%2020240122210053.png)
* apigateway private endpoint 배포 sam template 정의
  * sam template 정의
    ![](/images/Pasted%20image%2020240122210112.png)
  * console 에서 endpoint 구성 확인
    ![](/images/Pasted%20image%2020240122210127.png)
  * console 에서 리소스 정책 확인
    ![](/images/Pasted%20image%2020240122210140.png)
* 확인 방법
  * ec2 인스턴스를 하나 생성하여 확인
  * nslookup 확인 -정상 확인
    ![](/images/Pasted%20image%2020240122210156.png)
  * curl를 사용하여 테스트 api 호출를 하면 호출이 실패 된다
  * 해당 ec2 및 vpc endpoint 의 보안 그룹의 인바운드 규칙에 443 포트를 추가 하여 해결
    ![](/images/Pasted%20image%2020240122210211.png)


### 다른 리전에서 private link
* 사전 준비
  * 겹치지 않는 CIDR 범위를 가지고 있는 서로 다른 리전에 존재하는 VPC
* VPC Peering 구성
  * 서울 리전에 VPC와 오레곤 VPC를 연결
  * 서울 리전에서 다음과 같이 피어링을 구성한다.
    ![](/images/Pasted%20image%2020240122210226.png)
* VPC ID(Accepter)는 직접 입력을 한다.
  ![](/images/Pasted%20image%2020240122210326.png)
* 위의 구성이 완료가 되면 오레콘 리전의 VPC Peering connection 에서 수락을 하면 일단 구성 완료
  ![](/images/Pasted%20image%2020240122210400.png)
  ![](/images/Pasted%20image%2020240122210430.png)
* VPC DNS resolution, hostnames 옵션 체크

  ![](/images/Pasted%20image%2020240122210505.png)

  ![](/images/Pasted%20image%2020240122210544.png)

* EC2 인스턴스 인바운드 규칙에 https를 허용해야 한다.

  ![](/images/Pasted%20image%2020240122210627.png)

* VPC Peering Connection DNS resolution support 활성화
* VPC Route Tables 에 상대 VPC CIDR 등록하고 target은 vpc peering id 이다.
* 테스트 방법
* 서울 리전 및 오레곤 리전에 위에 설정에 사용한 VPC에 EC2 인스턴스를 각각 생성 서울 리전
* VPC Endpoint를 위에 설명대로 설정을 하였다면 다음과 같이 private 으로 설정된 apigateway 호출이 정상 동작 할것이다.
* 오레곤 리전
* execute-api 호출 url이 서울 리전과 다르다 https://{rest-api-id}-{vpce-id}.execute-api.{region}.amazonaws.com/{stage}

![](/images/Pasted%20image%2020240122210757.png)
![](/images/Pasted%20image%2020240122210806.png)
![](/images/Pasted%20image%2020240122210815.png)
![](/images/Pasted%20image%2020240122210824.png)



### 같은 리전에서 private link
* API Gateway execute-api용 인터페이스 VPC endpoint 생성
* https://console.aws.amazon.com/vpc
* Endpoint 생성
  ![](/images/Pasted%20image%2020240122210855.png)
* VPC DNS 호스트 이름이 체크 되어야 한다.
  ![](/images/Pasted%20image%2020240122210907.png)
* console 에서 endpoint 구성 확인
  ![](/images/Pasted%20image%2020240122210917.png)
* console 에서 리소스 정책 확인
  ![](/images/Pasted%20image%2020240122210928.png)

### 확이 방법
* ec2 인스턴스를 하나 생성 하여 확인
* nslookup 확인 - 정상 확인
* curl를 사용하여 테스트 api 호출를 하면 호출이 실패 된다해당 ec2 및 vpc endpoint 의 보안 그룹의 인바운드 규칙에 443 포트를 추가 하여 해결


### 참고
* [Amazon API Gateway에서 프라이빗 API 생성](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-private-apis.html)
* [문제 해결 가이드 참고](https://aws.amazon.com/ko/premiumsupport/knowledge-center/api-gateway-private-endpoint-connection)
* [다른 계정에서 접근 참고](https://aws.amazon.com/ko/premiumsupport/knowledge-center/api-gateway-private-cross-account-vpce)
* [피어링 DNS](https://docs.aws.amazon.com/ko_kr/vpc/latest/peering/modify-peering-connections.html)
