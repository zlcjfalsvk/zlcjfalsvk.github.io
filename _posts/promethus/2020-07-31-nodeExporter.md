---
title: "promethus - node exporter"
categories: 
  - promethus
tags:
  - promethus
last_modified_at: 2020-07-31T15:00:00+09:00
author_profile: true
---
[promethus](https://prometheus.io/)가 **metric**를 수집하기 위해선 **exporter**라는 것이 필요하며, **exporter**는 data를 수집하고 있다가  `promethus -> metric`로 요청을 해 data를 가져온다.

## 설치

[promethus](https://prometheus.io/download/) 에서 알맞은 os 선택을 하면 다운로드 list를 확인할 수 있다.

    {% highlight console %}
    cd ~
    mkdir promethus
    wget https://github.com/prometheus/node_exporter/releases/download/v1.0.1/node_exporter-1.0.1.linux-amd64.tar.gz
    tar -zxvf node_exporter-1.0.1.linux-amd64.tar.gz 
    cd node_exporter-1.0.1.linux-amd64/
    {% endhighlight %}

![1](/assets/img/posts/promethus/nodeexpoter/1.png)

## 실행

    {% highlight console %}
    # 도움말    
    ./node_exporter --help

    # 실행 & 를 붙여 백그라운드 실행
    ./node_exporter &
    {% endhighlight %}

실행 후 브라우저에서 접속해봤는데 **metric**을 잘 수집하는거 같다.
![2](/assets/img/posts/promethus/nodeexpoter/2.png)

## 참고

[guide](https://prometheus.io/docs/instrumenting/exporters/)에 보면 **node** 뿐 아니라 **haproxy**, **mysql** 등 다양한 **exporter** 들을 지원한다.

---
#### 참고 및 출처

promethus
- <https://prometheus.io/>
- <https://prometheus.io/docs/instrumenting/exporters/>