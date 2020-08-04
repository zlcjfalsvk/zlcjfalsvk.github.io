---
title: "prometheus - alertmanager"
categories: 
  - prometheus
tags:
  - prometheus
last_modified_at: 2020-08-04T09:00:00+09:00
author_profile: true
sidebar:
  - nav: prometheus
---
[alertmanager](https://prometheus.io/docs/alerting/latest/configuration/) **prometheus server**에서 **metrice**을 수집했을 때 [alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)에 의해 경고를 받은 경우 엔지니어 등에 알리기 위해 **push alerts**를 가능하게 해주는 도구이다.

## 설치

[prometheus](https://prometheus.io/download/) 에서 알맞은 os 선택을 하면 다운로드 list를 확인할 수 있다.

    {% highlight console %}
    cd ~
    mkdir prometheus
    wget https://github.com/prometheus/alertmanager/releases/download/v0.21.0/alertmanager-0.21.0.linux-amd64.tar.gz
    tar -zxvf alertmanager-0.21.0.linux-amd64.tar.gz
    cd alertmanager-0.21.0.linux-amd64
    {% endhighlight %}

![1](/assets/img/posts/prometheus/alertmanager/1.png)

## 준비
**alertmanager**는 `alertmanager.yml`을 이용하여 **config** 설정을 한다. [config]](https://prometheus.io/docs/alerting/latest/configuration/#slack_config)서 규칙을 확인할 수 있다.


### email 설정
**email**에 **alert**보내고 싶을 경우 다음과 같이 **stmp**주소를 알아야 한다.

##### 예) office 365
![2](/assets/img/posts/prometheus/alertmanager/2.png)

### slack 설정
**slack**에 **alert**보내고 싶을 경우 다음과 같이 [webhook](https://api.slack.com/messaging/webhooks)을 설정해야 한다.

1. [webhook](https://api.slack.com/messaging/webhooks) 접속 후 `create your app `클릭

    ![21](/assets/img/posts/prometheus/alertmanager/21.png)

2. **app name**설정 후 **slack workspacke** 연결

    ![22](/assets/img/posts/prometheus/alertmanager/22.png)
        
3. **web hook** 추가 버튼 클릭 및 **webhoob 채널** 선택 

    ![23](/assets/img/posts/prometheus/alertmanager/23.png)
    ![24](/assets/img/posts/prometheus/alertmanager/24.png)

4. **URL**복사

    ![25](/assets/img/posts/prometheus/alertmanager/25.png)

## 설정 `alertmanager.yml`

    {% highlight yml %}
    global:
    resolve_timeout: 5s

    # email smtp global config
    #-----------------------
    smtp_smarthost: <STMP_URL: PORT>
    smtp_auth_username: <STMP_AUTH>
    smtp_auth_password: <STMP_PASSWORD>
    # -----------------------

    # slack global config
    slack_api_url: <SLTACK_WEBHOOK_URL

    route:
    group_by: [ alertname ]
    group_wait: 10s
    group_interval: 10s
    repeat_interval: 1h
    receiver: 'hook'
    routes:
        - receiver: 'hook'
        match:
            severity: warning

    receivers:
    - name: 'hook'
    email_configs:
    - send_resolved: true
        to: <RECEIVER_EMAIL>
        from: <SENDER_EMAIL>
        # Add whatever other settings you need to talk to your SMTP server
        headers:
        subject: "You have \{\{ .Alerts.Firing | len \}\} firing alerts"
        html: |
        <p>
        You have the following firing alerts:
        <ul>
            \{\{ range .Alerts \}\}
            <li>\{\{ .Labels.alertname \}\} on \{\{ .Labels.instance \}\}</li>
            \{\{ end \}\}
        </ul>
        </p>
    
    slack_configs:
        - channel: <SLACK_CHANNERL_NAME>
        send_resolved: true
        text: "\{\{ range .Alerts \}\} \{\{ .Labels.alertname\}\} on \{\{ .Labels.instance \}\} \n\{\{ end \}\}"    
    {% endhighlight %}    

## prometheus에 추가
`prometheus.yml`의 `alerting`에 추가해준다.

    {% highlight yml %}
    # my global config
    global:
      scrape_interval:     5s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
      evaluation_interval: 5s # Evaluate rules every 15 seconds. The default is every 1 minute.
      # scrape_timeout is set to the global default (10s).

    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          - alertmanager:9093

    rule_files:
      - "./rules/alert_rules.yml"

    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
        - targets: ['localhost:9090']
               
      - job_name: 'node-exporter-local'
        static_configs: 
        - targets: ['localhost:9100']    
    {% endhighlight %}  

## 결과

![31](/assets/img/posts/prometheus/alertmanager/31.png)

---
#### 참고 및 출처

promethesu concept
- <https://prometheus.io/docs/introduction/overview/>

alertmanager
- <https://prometheus.io/docs/alerting/latest/overview/>

email template example
- <https://www.robustperception.io/using-labels-to-direct-email-notifications>

slack template example
- <https://prometheus.io/docs/alerting/latest/notification_examples/>