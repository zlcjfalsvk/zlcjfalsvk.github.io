---
title: "k8s dashboard"
categories: 
  - Kubernetes
tags:
  - Kubernetes
last_modified_at: 2020-05-13T13:00:00+09:00
author_profile: true
sidebar:
  - nav: kubernetes-nav
---

**minikube**에서는 **minikube dashboard**를 입력하면 됬지만, <br/>
**kubeadm**에서는 직접 **dashboard**를 구축해야 한다.
이 방법을 [**리소스 메트릭 파이프라인**](https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-metrics-pipeline/)이라고 한다.

구축은 쉬우나 
접근 방법은 3가지가 있다. 이 글은 3번 API Server를 이용한 구축 및 접근을 작성한다.

1. proxy 이용 [(가이드)](https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/)

    \- master node에서만 접근 가능, 보안에 취약

2. nodePort 이용 [(가이드)](https://github.com/kubernetes/dashboard/blob/master/docs/user/accessing-dashboard/README.md)

    \-  싱글 클러스터 환경에서만 유용, 보안에 취약

3. API Server

    \- 인증서를 이용한 접근

### 1. 설치

1. master node에서 dashboard ui 설치

        kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

        kubecet get pods -A # 설치후 확인 가능


### 2. 인증서 
실제 운영 환경에서는 외부에서 접속을 해야 하며, 접속을 하기 위해 접속하려는 호스트에 인증서를 등록해줘야 한다.

1. 인증서 생성

-   master node 생성 후 ~/.kube/config를 만들었다면 바로 만들 수 있다. [(없다면 이동)](./2020-05-25-kubeadm.md#생성)
-   내 인증서를 만들자

        $ grep 'client-certificate-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.crt

        $ grep 'client-key-data' ~/.kube/config | head -n 1 | awk '{print $2}' | base64 -d >> kubecfg.key

        # 입력 후 패스워드를 입력해야하는데 기억하자
        $ openssl pkcs12 -export -clcerts -inkey kubecfg.key -in kubecfg.crt -out kubecfg.p12 -name "kubernetes-admin"

2. CA 기관 인증서 
-   /etc/kubernetes/pki를 가보면 **ca.crt**파일이 있다.
    ![cacrt](/assets/img/posts/kubernetes/dashboard/cacrt.png)


3. **1 & 2** 에서 만든 kubecfg.p12, ca.crt 파일을 접근할 호스트에 다운을 받거나 복사를 한다.
    ![copy](/assets/img/posts/kubernetes/dashboard/copy.png)

### 3. 접속

이 부분은 os마다 다르지만 여기서는 리눅스를 담당한다. 다른 od는 여기를 참고하자 [(이동)](https://crystalcube.co.kr/199)

1. 호스트에서 다음과 같이 설정하자
   
        # 크롬 및 다른 브라우저 설정
        certutil -A \
        -n "kubernetes" \
        -t "TC,," \
        -d sql:$HOME/.pki/nssdb \
        -i $HOME/ca.crt


        pk12util -i $HOME/kubecfg.p12 \
        -d sql:$HOME/.pki/nssdb \
        -W pabii1234

        ----------------------------------------------------------------
        #firefox 인증서 설정할 위치
        find ~/.mozilla/firefox -name "cert8.db"|xargs dirname

        # firefox 에 설정
        certutil -A \
        -n "kubernetes" \
        -t "TC,," \
        -d sql:/home/cheolmin/.mozilla/firefox/y8qw3klj.default \
        -i $HOME/ca.crt


        pk12util -i $HOME/kubecfg.p12 \
        -d sql:/home/cheolmin/.mozilla/firefox/y8qw3klj.default \
        -W pabii1234


2. 이제 브라우저에서 다음의 url에 접근하면 인증서 페이지가 나온다.
            
        # master에서 'kubectl cluster-info' 확인가능
        https://<MasterIP>:<MasterPort>/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

|chrome|firefox|
|:---:|:---:|
|![chrome](/assets/img/posts/kubernetes/dashboard/chrome.png)|![firefox](/assets/img/posts/kubernetes/dashboard/firefox.png)|

3. 만약에 안된다면 다음과 같은 방법을 이용해보자 [별첨](#별첨)

4. 로그인 화면이 나온다면 성공!
   ![login](/assets/img/posts/kubernetes/dashboard/login.png)


### 로그인

이제 Token을 만들어서 접속을 해보자 (참고로 master에서 만들어야한다.)

1. service account 생성

        $ cat <<EOF | kubectl create -f -
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: admin-user
          namespace: kube-system
        EOF

2. clusterRoleBinding

        $ cat <<EOF | kubectl create -f -
        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: admin-user
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: cluster-admin
        subjects:
        - kind: ServiceAccount
          name: admin-user
          namespace: kube-system
        EOF

3. 추가한 계정의 Token 생성

        kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

    ![token](/assets/img/posts/kubernetes/dashboard/token.png)

4. 이 Token을 가지로 로그인!

    ![login](/assets/img/posts/kubernetes/dashboard/logined.png)


5. **Metric**을 이용해보자
    
    Dashboard 및 **kubectl top <node|pod>**명령어를 사용하고 싶다면 [metric-server](https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-metrics-pipeline/)를 설치해야한다.

        kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/download/
        v0.3.6/components.yaml

    설치가 완료되면 노드및 파드의 metric수진 후 상단에 리소스 사용량에 대한 그래프를 확인할 수 있다.
    
    <table style="text-align:center;">
        <tr>
            <td>
                <img src="/assets/img/posts/kubernetes/dashboard/1.png"/>        
            </td>
            <td>
                <img src="/assets/img/posts/kubernetes/dashboard/2.png"/>
            </td>
        </tr>  
    </table>


### 후기
현재는 토큰을 가지고 로그인을 하는 방법을 만들었지만 나중에는 멀티 클러스터 환경에서의 접근 설정을 위한 kubeconfig 접근 방법을 해봐야겠다.


### 별첨

-   브라우저마다 다른데 chrome에서 인증서 설정은 다음과 같다.

    -   설정 - 인증서 검색 - 더보기 -인증서 관리
        ![chromeconfig](/assets/img/posts/kubernetes/dashboard/chromeconfig.png)

    -   내 인증서 부분에서는 kubecfg.p12 선택
    -   인증 기관 에서는 ca.crt 선택
        - 인증서가 이미 있다고 할 시 org-kubenetes 삭제 후 다시 해본다.
    

- prometheus를 이용한 완전 메트릭 파이프라인 [이동](https://gruuuuu.github.io/cloud/monitoring-02/#)



---
#### 참고 및 출처

access
- <https://crystalcube.co.kr/199>

dashboard
- <https://kubernetes.io/ko/docs/tasks/access-application-cluster/web-ui-dashboard/>
- <https://github.com/kubernetes/dashboard/tree/master/docs>

token
- <https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md>

Resource Metric Pipeline
- <https://kubernetes.io/ko/docs/tasks/debug-application-cluster/resource-usage-monitoring/>

Metric Server
- <https://github.com/kubernetes-sigs/metrics-server>