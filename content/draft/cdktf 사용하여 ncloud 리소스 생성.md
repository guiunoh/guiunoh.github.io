---
title: "cdktf 사용하여 ncloud 리소스 생성"
categories:
  - "development"
tags:
  - "terraform"
  - "cdjtf"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---


### 설명
* terraform 은 선언적으로 리소스를 정의 하고 관리 하는 반면 cdktf는 리소스를 코드로서 정의 하고 관리를 한다


### 구성
* vpc
* subnet
* network acl


### 사전 준비
* cdktf
  ```bash
  brew install cdktf
  ```
* node
  ```bash
  brew install nvm
  nvm install --lts
  ```
* ncloud 인증키 - ncloud.com 에서 계정 관리의 인증키 관리에서 API 인증키 발급 [link](https://www.ncloud.com/mypage/manage/authkey)
￼

### 실행 방법
* 소스 받기
  ```bash
  git clone git@github.com:guiunoh/cdktf-ncp-example.git
  ```
* terraform.tfvars 생성
  ```bash
  # variable에 정의된 변수에 주입
  echo 'access_key = "[발급 받은 Access Key ID]"
  secret_key = "[발급 받은 Secret Key]"' > terraform.tfvars
  ```
* 초기화
  ```bash
  cdktf get
  npm install
  ```
* 리소스 생성
  ```bash
  cdktf deploy
  ```
* 리소스 삭제
  ```bash
  cdktf destroy
  ```


### 리소스 생성 가이드
* 프로젝트 생성
  ```bash
  mkdir cdktf-ncp-example
  cd cdktf-ncp-example
  cdktf init --template=typescript --providers=navercloudplatform/ncloud --local
  ```
* main.ts
  ```typescript
  import { Construct } from "constructs";
  import { App, TerraformStack } from "cdktf";
  import { provider } from './.gen/providers/ncloud';

  class MyStack extends TerraformStack {
    constructor(scope: Construct, name: string) {
      super(scope, name);

      // define resources here
      const accessKey = new TerraformVariable(this, "access_key", {
        type: "string",
        nullable: false
      });

      const secretKey = new TerraformVariable(this, "secret_key", {
        type: "string",
        nullable: false
      });

      const name = "cdktf-demo";
      const region = process.env.NCLOUD_REGION || 'KR';


      // define resources here
      new provider.NcloudProvider(this, 'ncloud', {
        accessKey: accessKey.toString(),
        secretKey: secretKey.toString(),
        region: region,
        supportVpc: true,
      });
    }
  }

  const app = new App();
  new MyStack(app, "cdktf-example-ncp");
  app.synth();
  ```
* TerraformVariable을 사용하기 위해 terraform.tfvars 파일 생성이 필요
* 리소스 생성
  ```bash
  cdktf synth
  cdktf diff # 리코트 서버와 로컬의 차이를 확인
  cdktf deploy
  ```
* 리소스 삭제
  ```bash
  cdktf destroy
  ```


### 기타
* terraform.[스택 이름].tfstate 파일이 생성되면 반드시 보관되어야 한다 - 리모트에 생성된 리소스 모양과 일치


### 참고
* [CDK for Terraform](https://developer.hashicorp.com/terraform/cdktf)

