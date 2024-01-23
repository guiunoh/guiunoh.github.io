---
title: "terraform 사용하여 ncloud 리소스 생성"
categories:
  - "development"
tags:
  - "terraform"
  - "ncloud"
date: "2024-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---


### 구성
* vpc
* subnet
* network acl
* nat gateway
* public ip
* route table
* public server
* private server
* pem file

### 사전 준비
* terraform cli
```bash
brew install terraform
```
* ncloud 인증키 - ncloud.com 에서 계정 관리의 인증키 관리에서 API 인증키 발급 [link](https://www.ncloud.com/mypage/manage/authkey)
￼
### 실행 방법
* 소스 받기
```bash
git clone git@github.com:guiunoh/tf-ncp-example.git
```
* .auto.tfvars 생성
```bash
# variable에 정의된 변수에 주입
echo 'access_key = "[발급 받은 Access Key ID]"
secret_key = "[발급 받은 Secret Key]"' > .auto.tfvars
```
* 초기화
```bash
terraform init
```
* 리소스 생성
```bash
terraform apply
```
* 리소스 삭제
```bash
terraform destroy
```

### 리소스 생성 가이드
* 작업 폴더 생성
```bash
mkdir tf-ncp-example
```
* 파일 생성 - 다음과 같이 3개의 파일을 준비한다 - 파일 구조는 특별한 것은 없고 필요에 따라서 나누거나 합쳐서 사용하면 된다.
```bash
├── main.tf       # 리소스 생성에 대한 정의
├── providers.tf  # 어떤 리소스 제공자를 사용하는지 정의
├── variables.tf  # 필요한 변수 정의
```
* providers.tf
```terraform
terraform {
    required_providers {
        ncloud = {
            source = "navercloudplatform/ncloud" # navercloud
        }
    }
    required_version = ">= 0.13"
}

provider "ncloud" {
    support_vpc = true
    region      = "KR"
    access_key  = var.access_key # variables.tf 에 정의 된 access_key 사용
    secret_key  = var.secret_key # variables.tf 에 정의 된 secret_key 사용
}
```
* variables.tf
```terraform
variable name {
    default = "demo" # 리소스 생성시 prefix 사용
}

variable "access_key" {
    type     = string
    nullable = false
}

variable "secret_key" {
    type     = string
    nullable = false
}

variable front_count {
    default = 1
}

variable backend_count {
    default = 1
}
```
* 초기화 - 위에 까지 준비가 된 상태에서 초기화를 진행
```bash
terraform init
```
* 리소스 생성 - vpc
```terraform
resource "ncloud_vpc" "vpc" {
    name            = "${var.name}-vpc"
    ipv4_cidr_block = "10.0.0.0/16"
}
```
* 리소스 생성 - subnet
```terraform
resource "ncloud_subnet" "subnet_public" {
    name   = "${var.name}-subnet-public"
    vpc_no = ncloud_vpc.vpc.id
    subnet         = cidrsubnet(ncloud_vpc.vpc.ipv4_cidr_block, 8, 0)
    zone           = "KR-2"
    network_acl_no = ncloud_network_acl.nacl_public.id
    subnet_type    = "PUBLIC"
}

resource "ncloud_subnet" "subnet_private" {
    name   = "${var.name}-subnet-private"
    vpc_no = ncloud_vpc.vpc.id
    subnet         = cidrsubnet(ncloud_vpc.vpc.ipv4_cidr_block, 8, 1)
    zone           = "KR-2"
    network_acl_no = ncloud_network_acl.nacl_private.id
    subnet_type    = "PRIVATE"
}
```
* 리소스 생성 명령 구조
```terraform
resource "생성할 리소스" "변수명" {
    리소스의 속성
    ....
}
```
* 정의된 코드 확인
```terraform
terraform plan
```
* 추가된 리소스가 있다면 위에 명령어는 실패 - 다음 명령어를 사용하여 추가 리소스 다운로드
```terraform
terraform init -upgrade
```
* 리소스 생성
```terraform
terraform apply
```
* 리소스 삭제
```terraform
terraform destroy
```

### 기타
* terraform apply 이후 생성 되는 "terraform.tfstate" 파일은 반드시 보관되어야 한다 - 리모트에 생성된 리소스 모양과 일치
* terraform 라이센스 변경으로 대체 가능한 오픈소스 [opentofu](https://opentofu.org)

### 참고
* [Ncloud Provider](https://registry.terraform.io/providers/NaverCloudPlatform/ncloud/latest/docs)
* [네이버 클라우드 플랫폼 Terraform Provider 개발기](https://tv.naver.com/v/23651001)
* [간단하게 베어메탈 서버 구성하기](https://blog.naver.com/n_cloudplatform/222272175921)
