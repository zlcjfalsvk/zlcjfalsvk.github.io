---
title: "Docker Compose File To K8S Resources "
categories: 
  - Kubernetes
tags:
  - Kubernetes
  - Docker-compose
last_modified_at: 2020-05-13T13:00:00+09:00
author_profile: false
sidebar:
  - nav: kubernetes-nav
---
## docker compose 파일을 k8s Resource로 바꿔보자 [문서](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/)

### Kompose 설치

    # Linux
    curl -L https://github.com/kubernetes/kompose/releases/download/v1.21.0/kompose-linux-amd64 -o kompose
    # macOS
    curl -L https://github.com/kubernetes/kompose/releases/download/v1.21.0/kompose-darwin-amd64 -o kompose


    # Windows
    curl -L https://github.com/kubernetes/kompose/releases/download/v1.21.0/kompose-windows-amd64.exe -o kompose.exe


    chmod +x kompose
    sudo mv ./kompose /usr/local/bin/kompose

![install](../../assets/img/_posts/kubernetes/composeTokube/install.png)


### Kompose 사용

1. docker-compose.yml 파일이 있는 곳으로 이동

2. kompose convert 입력 (Deployment 파일 생성) 
   -  바로 생성하고 싶다면 kompose up 실행
   
![convert](../../assets/img/_posts/kubernetes/composeTokube/convert.png)


