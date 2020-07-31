---
title: "promethus - promethus"
categories: 
  - promethus
tags:
  - promethus
last_modified_at: 2020-07-31T18:00:00+09:00
author_profile: true
---
## 설치

[promethus](https://prometheus.io/download/) 에서 알맞은 os 선택을 하면 다운로드 list를 확인할 수 있다.

    {% highlight console %}
    cd ~
    mkdir promethus
    wget https://github.com/prometheus/prometheus/releases/download/v2.20.0/prometheus-2.20.0.linux-amd64.tar.gz
    tar -zxvf prometheus-2.20.0.linux-amd64.tar.gz 
    cd prometheus-2.20.0.linux-amd64/
    {% endhighlight %}

![1](/assets/img/posts/promethus/promethus/1.png)

## Config

**promethus** 자체는 수집할 **node**들의 [exporter](https://prometheus.io/docs/instrumenting/exporters/)들에 **metrics**을 요청하고 수집하는 서버이다.

그리고 그 설정은 `promethus.yml`에서 설정할 수 있다. [문서](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#static_config)

### default 

    {% highlight yaml %}
    # my global config
    global:
      scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
      # scrape_timeout is set to the global default (10s).

    # Alertmanager configuration
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          # - alertmanager:9093

    # Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
    rule_files:
      # - "first_rules.yml"
      # - "second_rules.yml"

    # A scrape configuration containing exactly one endpoint to scrape:
    # Here it's Prometheus itself.
    scrape_configs:
      # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
      - job_name: 'prometheus'

        # metrics_path defaults to '/metrics'
        # scheme defaults to 'http'.

        static_configs:
        - targets: ['localhost:9090']
    {% endhighlight %}

### custom
[node-exporter](./2020-07-31-nodeExporter.md) 실행 후 지표를 수집해보자.

    {% highlight yaml %}
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
        - targets: ['localhost:9090']
      
      - job_name: 'node-exporter-local'
        static_configs: 
        - targets: ['localhost:9100']          
    {% endhighlight %}

## 실행

`./prometheus`를 실행 후 브라우저에서 `http://localhost:9090`을 입력하면 다음과 같이 화면이 나온다.

![2](/assets/img/posts/promethus/promethus/2.png)

현재 **promethus** 수집하고 있는 **metrics**를 보려면 다음과 같이 확인할 수 있다.
![3](/assets/img/posts/promethus/promethus/3.png)

---
#### 참고 및 출처

promethus
- <https://prometheus.io/>
- <https://github.com/prometheus/prometheus>