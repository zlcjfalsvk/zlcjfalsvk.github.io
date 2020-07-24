---
title: "node.js - certificate has expired (2)"
categories: 
  - node.js
tags:
  - node.js
last_modified_at: 2020-05-03T21:38:00+09:00
author_profile: true
---

Server to Server 통신을 하다가 발견하였다.

    { 
        Error: certificate has expired
        at TLSSocket.<anonymous> (_tls_wrap.js:1116:38)
        at emitNone (events.js:106:13)
        at TLSSocket.emit (events.js:208:7)
        at TLSSocket._finishInit (_tls_wrap.js:643:8)
        at TLSWrap.ssl.onhandshakedone (_tls_wrap.js:473:38) code: 'CERT_HAS_EXPIRED' 
    }

저번과 다르게 이번에는 해당 서버의 운영체제에서 curl로 API 호출을 하면 정상작동하는데, Node 서버끼리 통신이 문제였다. 혹시 몰라 다음 사이트를 통해 AWS EC2 콘솔에 CA 인증서를 업데이트도 해봤다. [이동](https://stackoverflow.com/questions/62107431/curl-error-60-ssl-certificate-problem-certificate-has-expired/62108103#62108103)


구글에 TLSWrap.ssl.onhandshakedone 키워드로 검색을 하다가 어제 (2020-06-02) 올라온 질문을 발견하였다. [이동](https://stackoverflow.com/questions/62128611/nodejs-certificate-has-expired)

여기서 흥미로운 답변 중 노드 버전 이슈에 대한 답변을 봤다.

테스트 코드를 만들고 Node 버전 별로 테스트를 했더니 
- lts / carbon-> v8.17.0 (인증서 만료)
- lts / dubnium-> v10.20.1 (확인)
- lts / erbium-> v12.17.0 (확인)

라는 결과를 얻었다고 한다.



이게 내가 겪은 문제라고 생각하는게 현재 운영 중인 서버의 Node 버전이 8.16.0이다. (넘 구식....)

그래서 현재는 Node.js 버전 문제일 가능성이 가장 컸다.

하지만 Node.js 버전 문제와 동시에 SSL 인증서 문제도 있지 않을까?라는 의심이 생겼다.

네이버나 구글같은 기업의 인증서는 문제가 없다고 나오는데, 문제가 있는 사이트는 오른쪽 사진과 같이 나왔기 때문이다.

![2](/assets/img/posts/nodejs/certificate/2.png)

인증서는 2021-06-04 까지이지만 루트 또는 중간 **CA?가 만료**됐다고 한다.

이게 무슨말인지 아직까지 이해하지는 못했지만 내가 사용 중인 인증서에 문제가 있을 것이라는 확신이 점점 들기 시작했다.

---
#### 참고 및 출처

- <https://www.sslshopper.com/ssl-checker.html>
- <https://stackoverflow.com/questions/62128611/nodejs-certificate-has-expired>
- <https://stackoverflow.com/questions/62107431/curl-error-60-ssl-certificate-problem-certificate-has-expired/62108103#62108103﻿>