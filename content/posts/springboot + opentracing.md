---
title: "springboot + opentracing"
categories:
  - "development"
tags:
  - "springboot"
  - "opentracing"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---

### 설명
* 분산된 시스템에서의 마이크로서비스 간 상호작용을 추적하고 모니터링하기 위한 벤더 중립적인 API와 도구 세트


### 데이타 모델
* Trace : 분산 시스템에서 하나의 요청 또는 작업을 의미. Span의 집합
* Span : Trace 내에서의 하나의 작은 작업 의미
* Span Context : Span이 분산 시스템에서 고유하게 식별될 수 있도록 하는 정보. 다른 서비스와 상호 작용할 때도 사용


### SpringBoot에서 데이타 모델 관찰
* Trace : 하나의 요청의 시작과 끝. Request -> Response 하나의 요청 처리 단위
* Span : 보통 Thread 가 달라 지는 경우 새로운 Span 이 생성
* Span Context : 다른 서비스에 요청시 사용
  ![](/images/Pasted%20image%2020240122195842.png)


### SpringBoot 사용 참고 내용
* BeanPostProcessor을 사용하여 Span을 생성
* 내부에서 생성된 오브젝트를 사용하는 경우 Span생성이 안됨
* Sleuth를 사용해야 자동으로 로그에 Trace ID 및 Span ID 가 삽입
* opentracing-spring-cloud-starter 사용시에 Log에 Trace ID 및 Span ID를 삽입하기 위해서는 Trace Log 사용


### 대표적인 구현체
* Jaeger
* Zipkin


### Jaeger와 Zipkin 비교
![](/images/Pasted%20image%2020240122195858.png)


### 예제
* 로컬에서 Jaeger 실행
  ```bash
  docker run -d --rm \
    -e COLLECTOR_ZIPKIN_HOST_PORT=9411 \
    -p 5775:5775/udp \
    -p 6831:6831/udp \
    -p 6832:6832/udp \
    -p 5778:5778 \
    -p 16686:16686 \
    -p 14268:14268 \
    -p 14250:14250 \
    -p 9411:9411 \
    jaegertracing/all-in-one:latest
  ```
* SprintBoot
  * build.gradle
    ```gradle
    implementation 'org.springframework.cloud:spring-cloud-starter-sleuth'
    implementation 'org.springframework.cloud:spring-cloud-sleuth-zipkin'
    ```
  * application.properties
    ```properties
    spring.sleuth.sampler.probability=1.0
    spring.zipkin.base-url=http://localhost:9411

    # default가 true
    spring.sleuth.opentracing.enabled=true
    ```
  * code
    ```java
    public String ping() throws ExecutionException, InterruptedException {
      log.info("Call FirstService ping");
      var response = client.getForObject("http://localhost:9090/ping", String.class);
      log.info("Receive Second Service:", response);

      CompletableFuture.runAsync(() -> {
        log.info("start long job");
        sleep(ThreadLocalRandom.current().nextInt(1, 1000));
        log.info("end long job");
      }, executor).get();
      return response;
    }
    ```


##### 테스트 실행
* 단일 요청
  ```bash
  curl http://localhost:8080/ping
  ```
* Apache Benchmark 를 실행하여 테스트
  ```bash
  ab -n 10 -c 10 http://localhost:8080/ping
  ```


##### 로그
* service1
  ![](/images/Pasted%20image%2020240122195926.png)
* service2
  ![](/images/Pasted%20image%2020240122195952.png)
* 처음 생성된 TraceID는 굵은 글씨
  ![](/images/Pasted%20image%2020240122200013.png)


##### Jaeger 에서 확인
![](/images/Pasted%20image%2020240122200030.png)


### 참고
* [The OpenTracing project](https://opentracing.io/)
* [LINE 광고 플랫폼의 MSA 환경에서 Zipkin을 활용해 로그 트레이싱하기](https://engineering.linecorp.com/ko/blog/line-ads-msa-opentracing-zipkin/)
