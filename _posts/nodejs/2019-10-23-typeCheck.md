---
title: "node.js - Typescript type check"
categories: 
  - node.js
tags:
  - node.js
  - typescript
last_modified_at: 2019-10-23T10:39:00+09:00
author_profile: true
---
안녕하십니까  저는 지금까지 typescript를 사용하면서 typesciprt의 장점을 잘 활용한 적이 없었는데,

오늘은 그 중하나인 **type**의 기능을 사용해 보려고 합니다.


이 글을 작성을 하면서 그전에 type과 interface의 차이에 대해 문뜩 궁금해졌습니다.

stack overflow에서는 2018년 TS 3.2 버전에는 다음과 같은 차이가 있다고 하는데, 2019년 버전 업 하면서 거의 차이가 없다고 하는 거 같습니다.

그래서 결국 저는 둘 다 사용해도 된다고 생각을 했고, 이번에는 **type**을 이용한 **type check**를 해보려고 합니다.

![1](/assets/img/posts/nodejs/typeCheck/1.png)

저는 express 모듈을 이용한 rest api를 위한 백엔드 서버를 위한 controller를 만들어 봤습니다.

## 기존 코드

    {% highlight typescript %}
    somePush(req: express.Request, res: express.Response): any {
        FirebaseService.sendPush(req.body.ids,
                                req.body.title,
                                req.body.message,
                                req.body.type,
                                req.body.silent,
                                req.body.roomId,
                                req.body.mesType)  
        .then(r => {
        res.status(200).send(r);
        })
        .catch(err => {
        res.status(200).send(err);
        });
    }


    async sendPush(ids: Array<{ id: string }>, 
                title: string,
                mes: string,
                type: string,
                silent: "Y" | "N",
                roomId?: string,
                mesType?: string): Promise<any> {
                
            //  코드 생략              
    }    
    {% endhighlight %}

저기서 파라미터를 더 추가한다던가, 공통 모듈로 사용하고 싶을 때 따로 function을 더 만들어야 할까요? 

저는 이걸 어떻게 해야 공통 모듈로 사용할 수 있을까 생각하다가 type check를 이용해 봤습니다.

type을 이용하면 다음과 같은 코드로 작성할 수 있습니다.

## type check 이용

    {% highlight typescript %}
    somePush(req: express.Request, res: express.Response): any {
        FirebaseService.sendPush(req.body)
        .then(r => {
            res.status(200).send(r);
        })
        .catch(err => {
            res.status(200).send(err);
        });
    }      
    {% endhighlight %}
    
### type

    {% highlight typescript %}
    declare namespace PushType {
        type somePush = {
                        ids: Array<{ id: string }>, 
                        title: string,
                        message: string,
                        type: string,
                        silent: "Y" | "N",
                        roomId?: string,
                        mesType?: string
        }

        type somePush2 = {
                            id: string, 
                            title: string,
                            message: string,
                            type: string,
                            textType: string,
                            silent: "Y" | "N",
                            mesType?: string
        }
    }

    async sendPush(param: PushType.somePush | PushType.somePush2): Promise<any> {
        // 코드 생략
    }
    {% endhighlight %}

파라미터 받는 부분이 깔끔해진거 같습니다. **type**을 이용하니 받는 파라미터에 따라 **type**을 만들어 공통 모듈로 사용할 수 있을 거 같습니다.    

## 추가 보안
여기서 더 보안을 하면 type guard라는 기능을 이용할 수 있습니다.

[type guard](https://www.typescriptlang.org/docs/handbook/advanced-types.html?fbclid=IwAR1bjoH6EOdz2KjadoUI-2aoxxnIpY2097xj4nla8q645Ef9MxPiGnMEV3c#user-defined-type-guards﻿)를 하는 방법은 다양합니다. 

- typeof

- instanceof

- is

- === 



이러한 예약어를 이용한 방법이 있지만, 저는 `===` 키워드를 이용하여 **사용자 정의 타입 가드**를 만들어봤습니다.

    {% highlight typescript %}
    pushTypeCheck(param: PushType.somePush | PushType.somePush2): param is PushType.somePush2 {

        // if((param as PushType.somePush2).textType) {
        //   return true;
        // }
        // return false;
        return (param as PushType.somePush2).textType !== undefined;
    }    
    {% endhighlight %}

## 마무리

대단한 코트는 아니지만, 이러한 **type guard**를 만들어서 type에 따른 분기 처리를 있고,

아니면 **handler**에 **guard**를 넣어서 내가 원하는 type 형태를 보내지 않으면 error를 보내게 할 수도 있고

그 외에도 다양하게 이용할 수 있다고 생각합니다.



---
#### 참고 및 출처
type과 interface 차이
- <https://medium.com/@alexsung/typescript-type%EA%B3%BC-interface-%EC%B0%A8%EC%9D%B4-86666e3e90c>
- <https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types>



Basic Types
- <https://www.typescriptlang.org/docs/handbook/basic-types.html>

type check
- <https://stackoverflow.com/questions/14425568/interface-type-check-with-typescript>



`is` keyword
- <https://www.typescriptlang.org/docs/handbook/advanced-types.html?fbclid=IwAR2GutUK0bEVle5tFRTKuFd-XrICVUbgRtO1vzazg9DuTtkXdDlj1fQIyWI#user-defined-type-guards>


type guard

- <https://dev.to/krumpet/generic-type-guard-in-typescript-258l>
- <https://basarat.gitbooks.io/typescript/docs/types/typeGuard.html>
- <https://www.typescriptlang.org/docs/handbook/advanced-types.html?fbclid=IwAR1bjoH6EOdz2KjadoUI-2aoxxnIpY2097xj4nla8q645Ef9MxPiGnMEV3c#user-defined-type-guards﻿>
