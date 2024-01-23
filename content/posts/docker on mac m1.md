---
title: "docker on mac m1"
categories:
  - "environment"
tags:
  - "container"
  - "mac"
date: "2022-09-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: false
---


### 사전 준비
* rosetta 설치
  ```bash
  softwareupdate —install-rosetta
  ```
* [option]환경변수에 DOCKER_DEFAULT_PLATFORM 설정
  ```bash
  export DOCKER_DEFAULT_PLATFORM=linux/amd64
  ```

### docker 실행
* docker의 명령어에 platform 옵션을 설정을 할수 있다.
  ```bash
  --platform linux/amd64
  ```
* docker run, build 의 명령어에 platform 옵션을 다음과 주면 해당 architecture 이미지를 사용하게 된다
  ```bash
  docker run --platform linux/amd64 nginx
  docker build --platform linux/amd64 --tag example .
  ```
* DOCKER_DEFAULT_PLATFORM설정 되어 있다면 —platform 옵션을 제거해도 된다.


### 확인
* container image architecture
  ```bash
  docker image inspect --format='{{json .Architecture}}' b692a91e4e15
  ```


### multi architecture build
```bash
docker buildx create --name builder
docker buildx use builder
docker buildx inspect --bootstrap

docker buildx build \
	--platform linux/amd64,linux/arm64,linux/arm/v7 \
	--tag [container image name]:latest \
	--push .

docker run --rm -p 8080:8080 [container image name]

docker image inspect --format='{{json.Architecture}}' [container image name]
```


### 참고
* [Docker Desktop for Apple silicon | Docker Documentation](https://docs.docker.com/desktop/mac/apple-silicon/)
* [Working with Buildx | Docker Documentation](https://docs.docker.com/build/buildx/)
