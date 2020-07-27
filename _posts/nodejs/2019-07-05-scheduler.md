---
title: "node.js - scheduler"
categories: 
  - node.js
tags:
  - node.js
last_modified_at: 2019-07-05T13:00:00+09:00
author_profile: true
---
이번에는 node js 환경에서 스케줄러를 이용하는 방법을 작성하려 합니다.

그전에 스케줄러라는 개념에 대해 알고 있으신가요?

[위키백과](https://ko.wikipedia.org/wiki/%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81)에는 스케줄링을 다음과 같이 정의하고 있습니다.

`스케줄링(scheduling)은 다중 프로그래밍을 가능하게 하는 운영 체제의 동작 기법이다. 운영 체제는 프로세스들에게 CPU 등의 자원 배정을 적절히 함으로써 시스템의 성능을 개선할 수 있다.`

운영 체제의 동작 기법은 위처럼 정의를 하는데 프로그래밍에서 스케줄링은 일정 주기에 맞춰 로직을 실행시키는 방법?이라고 생각을 합니다. 

보통 사람들이 많이 사용하는 시간대에 일괄 데이터 처리를 하게 되면 cpu 사용량이 증가하여 서버가 죽거나 할 수 있어 트래픽이 적은 시간대에 일괄처리를 하기 위해 스케줄러를 사용합니다.

**Linux**를 많이 이용하시는 분은 **crontab**이라고 들어보셨죠? cron이 unix os 계열의 스케줄러입니다.<br />
그리고 대부분의 프로그래밍 언어들 또한 스케줄러를 제공을 하고 있으며, node js 또한 스케줄러 모듈이 있습니다.

[node-schedule](https://www.npmjs.com/package/node-schedule)이라는 모듈인데 많이들 사용하고 있더라고요.

### 규칙

    *    *    *    *    *    *
    ┬    ┬    ┬    ┬    ┬    ┬
    │    │    │    │    │    │
    │    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
    │    │    │    │    └───── month (1 - 12)
    │    │    │    └────────── day of month (1 - 31)
    │    │    └─────────────── hour (0 - 23)
    │    └──────────────────── minute (0 - 59)
    └───────────────────────── second (0 - 59, OPTIONAL)

실행 주기에 대한 설정은 crontab과 같습니다.

왼쪽부터 **초(생략 가능) 분 시간 일 달 주** 라는 규칙이며

예를 들어

23 30 \* \* \* \* 는 매 시간 30분 23초에 실행시키라는 규칙이며

\* \* 12 1 \* 는 매년 1월 12일에 실행시키라는 규칙이 적용됩니다.

## 사용

### pakage.json

    {% highlight json %}
    "dependencies": {
        "node-schedule": "^1.3.2"
    },
    "devDependencies": {
        "@types/node-schedule": "^1.2.3"
    }    
    {% endhighlight %}

### contTab.ts
    {% highlight typescript %}
    import schedule from "node-schedule";
    import l from "../configure/logger";

    /** cron-style Scheduling
    *    *    *    *    *    *
    ┬    ┬    ┬    ┬    ┬    ┬
    │    │    │    │    │    │
    │    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
    │    │    │    │    └───── month (1 - 12)
    │    │    │    └────────── day of month (1 - 31)
    │    │    └─────────────── hour (0 - 23)
    │    └──────────────────── minute (0 - 59)
    └───────────────────────── second (0 - 59, OPTIONAL)
    */
    export const scheduled = schedule.scheduleJob("0 0 * * *", async (cb) => {
    // export const scheduled = async () => {
    l.debug("Start Scheduling at " + new Date());
    console.log("자정마다 호출될거임!");
    l.debug("End Scheduling at " + new Date());
    });
    {% endhighlight %}

이렇게 만든 후 

여러분이 사용하는 index.ts나 function을 실행시키는 부분에 등록을 하면 끝입니다.

node js 자체가 모듈들이 잘 되어있다 보니 스케줄러도 또한 손쉽게 적용을 할 수 있었습니다.    


## 참고

**node.js Scheduler**를 이용하면 쉽게 스케줄러를 만들 수 있지만,

클러스터 모드를 이용할 때 주의해야 할 점이 있다면, [pm2](https://www.npmjs.com/package/pm2) 나 [forever](https://www.npmjs.com/package/forever) 같은 프로세서 매니저를 이용할 경우, `processer.isMaster`를 이용할 수 없어 클러스터마다 스케줄러가 돈다.

그 이유는 pm2나 forever 모듈이 master가 되어 클러스터로 도는 놈들은 worker 프로세스가 된다.

그래서 클러스터로 돌릴 경우 그냥 linux cron을 이용하자.


---
#### 참고 및 출처

스케줄링
- <https://ko.wikipedia.org/wiki/%EC%8A%A4%EC%BC%80%EC%A4%84%EB%A7%81>
  
node-schedule
- <https://www.npmjs.com/package/node-schedule>


