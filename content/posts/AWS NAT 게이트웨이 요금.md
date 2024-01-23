---
title: "AWS NAT 게이트웨이 요금"
categories:
  - "development"
tags:
  - "aws"
date: "2021-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---

### 요금 (서울 기준)
* 시간당 요금: 0.059USD
* 데이터 처리 요금: 1GB 당 0.059USD
* 데이터 전송 요금:
	* AWS 에서 인터넷으로 나가는 또는 서로 다른 가용영역으로 나가는 요금
	* EC2와 같은 요금
	* [참고](https://aws.amazon.com/ko/ec2/pricing/on-demand/#Data_Transfer)


### 예시 - 한달 동안 10TB
* 시간당 요금: 0.059 * 720 = 42.48USD
* 데이터 처리 요금: 0.059 * 10TB = 590USD
* 데이터 전송 요금: 0.126 * 10TB = 1260USD
* 합계: 1892.48USD(2,270,569.12원)