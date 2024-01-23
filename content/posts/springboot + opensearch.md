---
title: "springboot + opensearch"
categories:
  - "development"
tags:
  - "springboot"
  - "opensearch"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---


### 요약
* spring-data-opensearch
  * JDK 17 이상 지원
  * ReactiveClient는 준비중
  * ElasticsearchRepository 사용 가능
* spring-data-opensearch-starter
  * springboot3 이상 필요
* spring-data-elasticsearch
  * RestHighLevelClient 연결 실패
  * 기본 제공되는 reactive client는 연결 성공
  * ReactiveElasticsearchRepository 설정만으로 연결 가능


### 테스트 버전
* elasticsearch : 7.10.1
* opensearch : 1.2.4


### spring-data-opensearch
* 특징
  * JDK 17 이상 필요
  * spring-data-opensearch-starter을 사용하기 위해서는 springboot 3 이상 사용
  * reactive 미지원 (현재 이슈 등록 - [FEATURE Reactive Client for OpenSearch · Issue #108](https://github.com/opensearch-project/spring-data-opensearch/issues/108))
* 예제
  * JDK 17 사용
  * build.gradle
    ```groovy
    sourceCompatibility = '17'

    dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
      implementation 'org.opensearch.client:spring-data-opensearch:1.0.1'
    }
    ```
  * application.yml
    ```yaml
    opensearch:
      host: xxxxxxxxxxxxxxx
      port: 9200
      scheme: http
      username: xxxxx
      password: xxxxx
    ```
  * config - AbstractOpenSearchConfiguration
    ```java
    @Override
    public RestHighLevelClient opensearchClient() {
      final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
      credentialsProvider.setCredentials(AuthScope.ANY, new UsernamePasswordCredentials(username, password));

      var endpoint = new HttpHost(host, port, scheme);
      var builder = RestClient.builder(endpoint)
        .setHttpClientConfigCallback(httpClientBuilder -> httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider))
        .setRequestConfigCallback(requestConfigBuilder -> requestConfigBuilder.setConnectTimeout(connectTimeout)
          .setSocketTimeout(socketTimeout)
        );
      return new RestHighLevelClient(builder);
    }
    ```
  * repository
    ```java
    @Repository
    public interface ArticleRepository extends ElasticsearchRepository<Article, String> {
    }
    ```
  * controller
    ```java
    @PostMapping(value = "/", consumes = "application/json")
    public Article create(@RequestBody Article article) {
      return articleRepository.save(article);
    }

    @GetMapping(value = "/{id}", produces = "application/json")
    public Article retrieve(@PathVariable String id) {
      log.info("id: {}", id);
      return articleRepository.findById(id).orElseThrow(() -> new ResourceNotFoundException("ID: " + id));
    }
    ```
  * test cli
    ```bash
    http post http://localhost:8080/api/v1/articles/ title=example \
    | jq -r '.id' \
    | xargs -I {} sh -c 'http GET http://localhost:8080/api/v1/articles/{}'
    ```

### spring-data-elasticsearch
* 특징
  * ElasticsearchRepository
    * RestHighLevelClient 을 사용
    * RestHighLevelClient 는 opensearch 연결시 version 정보 관련 에러로 연결 실패
      ![](/images/Pasted%20image%2020240122200231.png)
* RestHighLevelClient 는 현재 Deprecated 되어 있음
* ReactiveElasticsearchRepository
* ReactiveElasticsearchClient 을 사용
* opensearch 정상 연결
* 예제
  * build.gradle
    ```groovy
    dependencies {
      implementation 'org.springframework.boot:spring-boot-starter-data-elasticsearch'
    }
    ```
  * application.yml
    ```yaml
    spring:
      elasticsearch:
        uris: http://xxxxxxxxx:9200
        username: xxxxx
        password: xxxxx
    ```
  * repository
    ```java
    @Repository
    public interface ReactiveArticleRepository extends ReactiveElasticsearchRepository<Article, String> {
    }
    ```
  * controller
    ```java
    @PostMapping(value = "/", consumes = "application/json")
    public Mono<Article> create(@RequestBody Article article) {
      return articleRepository.save(article);
    }

    @GetMapping(value = "/{id}", produces = "application/json")
    public Mono<Article> retrieve(@PathVariable String id) {
      log.info("id: {}", id);
      return articleRepository.findById(id)
        .switchIfEmpty(Mono.error(new ResponseStatusException(HttpStatus.NOT_FOUND, "Article not found")));
    }
    ```
  * test cli
    ```bash
    http post http://localhost:8080/api/v2/articles/ title=example \
    | jq -r '.id' \
    | xargs -I {} sh -c 'http GET http://localhost:8080/api/v2/articles/{}'
    ```
  * [GitHub](https://github.com/guiunoh/springboot-opensearch/tree/reactive)


### migration
* 클러스터를 중지하지 않고 업그레이드를 진행하려면 롤링 업그레이드를 선택
* [Migrating from Elasticsearch OSS to OpenSearch - OpenSearch documentation](https://opensearch.org/docs/latest/upgrade-to/upgrade-to/)
  ![](/images/Pasted%20image%2020240122200250.png)


### 참고
* [Spring Data ElasticSearch · Spring WebFlux By Example](https://hantsy.github.io/spring-reactive-sample/data/data-elasticsearch.html)
* [GitHub - opensearch-project/spring-data-opensearch](https://github.com/opensearch-project/spring-data-opensearch)
