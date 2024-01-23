---
title: "graphql 간단 소개 및 실습"
categories:
  - "development"
tags:
  - "graphql"
date: "2022-10-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---

### 특징
* 장점
  * over-fetching
  * under-fetching
  * schema base
  * playground
  * without versioning
  * single endpoint

* 단점
  * learning curve
  * cache policy

#### 실습
* 공개된 graphql 사용해 보기
  * [SWAPI GraphQL API](https://graphql.org/swapi-graphql)
  * [Overview - GitHub Docs](https://docs.github.com/en/graphql/overview)
* hasura cloud graphql  실습
  * [Hasura GraphQL Tutorial](https://hasura.io/learn/graphql/hasura/introduction/)
* aws console 를 사용하여 appsync api 생성
  * [Tutorial: DynamoDB Resolvers - AWS AppSync](https://docs.aws.amazon.com/appsync/latest/devguide/tutorial-dynamodb-resolvers.html)
* front tutorial
  * [FullStack Tutorial with GraphQL, React & Apollo](https://www.howtographql.com/react-apollo/0-introduction/)
* backend tutorial
  * [Building a GraphQL Server with TypeScript & Apollo](https://www.howtographql.com/typescript-apollo/0-introduction/)
* go graphql library - gqlgen
  * [Building GraphQL servers in golang — gqlgen](https://gqlgen.com/getting-started/)

### 참고
* [GraphQL Code Libraries, Tools and Services](https://graphql.org/code/)

### by chatgpt
>* 정리
>   * GraphQL은 데이터를 효과적으로 관리하고 API를 구축하는 데 도움이되는 새로운 데이터 질의 언어입니다.
>   * 이 언어는 RESTful API와는 다른 방식으로 작동합니다.
>   * GraphQL은 클라이언트가 필요한 데이터를 정확하게 요청할 수 있도록하는 유연한 데이터 모델을 제공합니다.
>* 장점:
>   * 유연한 데이터 모델: GraphQL은 클라이언트가 필요한 데이터를 정확하게 요청할 수 있도록하는 유연 한 데이터 모델을 제공합니다. 이 모델은 RESTful API보다 더 효율적이며 불필요한 데이터를 전송하 지 않습니다.
>   * 단일 요청: GraphQL은 클라이언트가 여러 RESTful API 호출을 대신하는 단일 요청을 보낼 수 있습 니다. 이것은 대역폭을 줄이고 네트워크 지연 시간을 감소시킵니다.
>   * 자체 설명: GraphQL은 스키마를 사용하여 API의 구조와 데이터 형식을 자동으로 설명합니다. 이것은 클라이언트가 API를 사용하는 방법을 더 쉽게 배울 수 있게합니다.
>   * 동적 쿼리: GraphQL은 동적 쿼리를 지원합니다. 이것은 클라이언트가 필요한 데이터를 조건에 따라 선택할 수 있도록합니다.
>* 단점:
>   * 학습곡선: GraphQL은 RESTful API보다 더 복잡합니다. 클라이언트와 서버 모두에 대한 새로운 학습 곡선이 필요합니다.
>   * 캐싱: GraphQL은 RESTful API보다 캐싱이 더 어렵습니다. 이것은 대역폭과 네트워크 지연 시간을 증가시킬 수 있습니다.
>   * 보안: GraphQL은 RESTful API보다 더 많은 보안 문제가 있을 수 있습니다. 이것은 잘못된 구성 및 권한 부여 문제로 인해 발생할 수 있습니다.
