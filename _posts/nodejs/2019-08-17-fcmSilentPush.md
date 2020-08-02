---
title: "nodejs - FCM Silent Push"
categories: 
  - node.js
tags:
  - node.js
  - fcm
last_modified_at: 2019-08-17T16:23:00+09:00
author_profile: true
sidebar:
  - nav: fcm
---
Push Server를 개발하면서 단순 Push 용도로만 사용하는 줄 알았지만,

Push 기능이 제법 다양하게 사용하더라고요.

**예를 들면** Background Service를 개발을 할 때 Client가 매번 API를 호출하여 새로운 데이터를 받아오는 작업 대신에 

Server가 Push 기능을 이용하여 Client에 데이터를 보내줄 수 있습니다. 

이러한 기능을 위해 Push를 보내는 작업을 **Silent Push**라고 하는 거 같습니다.

저번 글에 FCM 구현한 거를 이용하여 Silent Push를 사용할 시 data 형태만 바꿔서 보내주면 됩니다.

필자는는 Android와 IOS 공용으로 사용하기 위해서 FCM을 사용했으며 Push 전송 요청 형식은 [여기](https://firebase.google.com/docs/cloud-messaging/send-message)를 클릭해 주세요.

    {% highlight typescript %}
    const message = {
      data: {
        title: title,
        contents: mes,
        id: type, // AOS param
        aos_silent: silent, // AOS param,
      },
      tokens: tokenA,
      android: {
        ttl: 3600 * 1000
      },
      apns: {
        // IOS 설정
        payload: {
          aps: {}
        }
      }
    };

    if (silent === "Y") {
      message.apns.payload.aps["contentAvailable"] = true;
    } else {
      message.apns.payload.aps["contentAvailable"] = true;
      message.apns.payload.aps["sound"] = "default";
      message.apns.payload.aps["alert"] = {};
      message.apns.payload.aps["alert"]["title"] = title;
      message.apns.payload.aps["alert"]["body"] = mes;
    }    
    {% endhighlight %}

**data** 형태는 다음과 같습니다.

**Android Push** 같은 경우에는 `data key` 안에 넣으면 되지만, **IOS** 같은 경우에는 `data key`가 아닌 `apns.payload.aps` 안에 넣어줘야 합니다.

그리고 **contentAvailable**을 **true**로 해줘야 Silent Push로 보내지더라고요.

그 외에 alert나 sound 같은 "key"를 넣어서 보내줄 수도 있고, 그 외에 custom data도 넣어줄 수 있습니다.

이러한 정보는 문서에서도 볼 수 있고 `node_modules/firebase-admin` 모듈에 들어가서 어떠한 `key`들이 있는지 확인하실 수 있습니다.
