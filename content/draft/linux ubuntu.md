---
title: "linux ubuntu"
categories:
  - "environment"
tags:
  - "linux"
date: "2023-01-01T01:01:00+09:00"
comments: true
toc: true
sidebar: false
draft: true
---

#development/environment/linux

* change hostname
  ```bash
  hostnamectl set-hostname pluto
  ```

* install zsh shell
  ```bash
  apt-get update 
  apt-get install zsh
  ```

* add user
  ```bash
  useradd -m -s /bin/zsh guiunoh
  echo 'guiunoh ALL=NOPASSWD: ALL' >> /etc/sudoers
  ```