---
title: "k8s Private Docker Image"
categories: 
  - kubernetes
tags:
  - kubernetes
  - Docker
last_modified_at: 2020-05-13T13:00:00+09:00
author_profile: false
sidebar:
  - nav: kubernetes-nav
---

K8S(kubernetes)에서 Deployment, Pod 등 도커의 이미지를 활용해 컨테이너를 만드는데,
DockerHub(cloud)에 image를 올려 가져와야할 때가 있고 그 중에서 private한 image를 가져와야 할 때가 있다.

### 준비 사항
-   DockerHub에 image 올리기
    
    1. [DockerFile](https://docs.docker.com/engine/reference/builder/#usage)을 참조해서 DockerFile을 만들고 build 후 image를 만든 다음에 DockerHub private Repository에 올린다.
    ![dockerHub](/assets/img/posts/kubernetes/privateDocker/dockerhub.png)

    2. 터미널에 'docker login'을 입력 후 로그인을 한다.    
    ![login](/assets/img/posts/kubernetes/privateDocker/login.png)

    3. DockerFile이 있는 위치에서 build를 한다.
    ![build](/assets/img/posts/kubernetes/privateDocker/build.png)

    4. Push
    ![Push](/assets/img/posts/kubernetes/privateDocker/push.png)


### 세팅
- Master Noed접속 후 Docker Login

    1. dockerconfig secret 만들기 [(이동)](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/)<br/>
        \- Docker login 성공 후 기본적으로 다음과 같은 명령어를 통해 secret를 만들 수 있다.<br/>

        \- 더 많은 제어를 하고 싶으면 링크 클릭 후 yaml 참조

            kubectl create secret generic <secret name> \
            --from-file=.dockerconfigjson=<path/to/.docker/config.json> \
            --type=kubernetes.io/dockerconfigjson

    2. 배포하려는 image를 내 private repository로 변경
    3. spec부분에 imagePullSEcrets.name 기재

            apiVersion: apps/v1
            kind: Deployment
            metadata:
            name: privatePod
            labels:
                io.kompose.service: privatePod
            spec:
            replicas: 1
            selector:
                matchLabels:
                io.kompose.service: privatePod
            template:
                metadata:
                labels:
                    io.kompose.service: privatePod
                spec:
                containers:
                  image: pabii/bidder:latest # 내 Private Image 결로
                  name: bidder                
                imagePullSecrets:
                - name: regcred # 아까 설정한 secret이름
                nodeSelector:
                    kubernetes.io/hostname: worker1
                restartPolicy: Always
    4. 배포
    
        ![deploy](/assets/img/posts/kubernetes/privateDocker/deploy.png)

---
#### 참고 및 출처

Pull an Image from a Private Image
- https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
