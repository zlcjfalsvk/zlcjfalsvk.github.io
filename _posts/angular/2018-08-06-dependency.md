---
title: "Angular - WARNING in Circular dependency detected"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-08-06T13:00:00+09:00
author_profile: true
---
Angular를 사용하다가 갑자기 다음과 같은 **Circular Dependency Detected** 경고문이 나왔다.

**Dependency import**와 **export**를 잘못 사용해서 나오는 경고문이다. 

제일 좋은 방법은 필요없는 import 및 export를 지우는 것인데 너무 많아서 해결하기 힘들면 다음과 같이 사용해도 된다.

![1](/assets/img/posts/angular/dependency/1.png)

    WARNING in Circular dependency detected:
    src/app/_core/index.ts -> src/app/_shared/shared.module.ts -> src/app/_shared/component/goodssearch/goodssearch.component.ts -> src/app/_core/index.ts

    WARNING in Circular dependency detected:
    src/app/_shared/component/goodssearch/goodssearch.component.ts -> src/app/_core/index.ts -> src/app/_shared/shared.module.ts -> src/app/_shared/component/goodssearch/goodssearch.component.ts

    WARNING in Circular dependency detected:
    src/app/_shared/shared.module.ts -> src/app/_shared/component/goodssearch/goodssearch.component.ts -> src/app/_core/index.ts -> src/app/_shared/shared.module.ts

이럴 때 는 **angular-cli.json** 파일에서 다음과 같이 설정을 해주면 된다.

    # angular-cli.json
    "defaults": {
        ....
        "build": {
        "showCircularDependencies": false
        }
    }

---
#### 참고 및 출처