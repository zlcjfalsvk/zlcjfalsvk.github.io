---
title: "node.js - certificate has expired (1)"
categories: 
  - node.js
tags:
  - node.js
last_modified_at: 2020-05-31T13:00:00+09:00
author_profile: true
---
Node.js로 구현된 API Server를 운영하는 도중 갑자기 이런 에러를 발견하였다.

### certificate has expired

이러한 에러를 처음 접하면서 실서버에서 발견된 에러로 인해 빨리 해결해야 겠다는 생각에 디버깅도 해보고 로그를 찍어보며 API가 동작하는지 확인도 해보며 여러가지 시도를 하였는데 **API Server** 문제는 아니었다.

![1](/assets/img/posts/nodejs/certificate/1.png)

되게 위험한 방법이지만 다음과 같은 환경변수를 설정하여 해결하였다.

    process.env.NODE_TLS_REJECT_UNAUTHORIZED = '0';

node.js 환경에서 HTTPS / SSL / TLS 검사를 비활성화한다는 옵션인데 점심까지만 해도 잘 작동되던 서버가 갑자기 이러한 에러가 발생한이유에 대해서 생각을 하다가 다음과 같은 생각이 들었다.

"AWS EC2 콘솔에 접속하기 위한 pem를 변경한 적이 있는데 이게 원인중 하나가 아닐까?"

아직 확실한 대답을 할 순 없지만 지금까지 잘 동작 하다가 갑자기 HTTPS / SSL / TLS 검사를 한다는 것도 의문이 들고 일단 계속 알아보면서 찾으면 글을 업데이트 해야겠다.


---
#### 참고 및 출처

- <https://whitelife.tistory.com/303>
- <https://stackoverflow.com/questions/20433287/node-js-request-cert-has-expired﻿>
