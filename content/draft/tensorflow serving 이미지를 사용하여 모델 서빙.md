---
title: "tensorflow serving 이미지를 사용하여 모델 서빙"
categories:
  - "development"
tags:
  - "tensorflow"
  - "container"
date: "2024-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---

```javascript
// 여기에 Gist에 있는 코드를 붙여넣으세요
// Gist URL: https://gist.github.com/guiunoh/1e3413d168038bf9f9e55a29b0ac4b40
```


### 사전 설치
```bash
pip3 install tensorflow 
pip3 install numpy
brew install httpie
```


### model 생성
* 다음 모델 생성 코드를 model_create.py 으로 저장 하여 실행한다
  [tensorflow/serving - model create](https://gist.github.com/guiunoh/1e3413d168038bf9f9e55a29b0ac4b40)
* 다음과 같이 실행하여 모델을 생성한다.
  ```bash
  python3 ./model_create.py
  ```


### 모델 생성 파일 확인
* saved_model 폴더 확인


### tensorflow/serving 을 사용하여 학습 모델 서빙
* 공식 제공하는 컨테이너 이미지는 tensorflow/serving 이다.
* mac arm에서 공식 이미지는 지원하지 않는 것 같다 - 실행시 cpu 정보 관련 에러 발생
* emacski/tensorflow-serving 을 사용하면 정상 실행 확인
* 8501 REST 포트, 8500 GRPC 포트
* GRPC 테스트는 하지 못함 - grpcurl 사용시 reflection 관련 에러 발생
  ```bash
  docker run -d -it --rm --publish 8501:8501 --publish 8500:8500 \
      --mount type=bind,source=./saved_model/,target=/models/test_model \
      --env MODEL_NAME=test_model \
      emacski/tensorflow-serving
      #tensorflow/serving
  ```


### 테스트 요청 전송 및 확인
* 다음 내용을 request_test.py 로 저장
  [tensorflow/serving - request prediction](https://gist.github.com/guiunoh/2c6987f7e06a5ec98732f4a6b892c6d4)
* 다음과 같이 실행하여 모델 요청
  ```bash
  python3 ./request_test.py
  # http POST http://localhost:8501/v1/models/test_model:predict < data.json
  ```


### 참고
    * [Docker와 함께 제공되는 TensorFlow](https://www.tensorflow.org/tfx/serving/docker?hl=ko)
