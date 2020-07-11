---
title: "AWS - 해킹"
categories: 
  - else
tags:
  - AWS
last_modified_at: 2020-06-08T13:00:00+09:00
author_profile: true
---
누가 **AWS AccessKey**를 바꿔놨다.

[CloudTrail Management](https://docs.aws.amazon.com/ko_kr/awscloudtrail/latest/userguide/cloudtrail-concepts.html)을 이용하여 로그인 기록을 확인했는데 로그인 기록은 없고 **AccessKey**변경 기록만 남아있어 [Cli](https://aws.amazon.com/ko/cli/)를 통한 **AccessKey** 변경이 의심된다.

![1](/assets/img/posts/else/awsHacking/1.png)

이벤트가 발생한 IP를 확인해보니 중국 쪽이라 나오지만 정확한 데이터는 아니지만 그래도 의심하고 상황에 대처해야 생각을 했다.

![2](/assets/img/posts/else/awsHacking/2.png)

완벽한 해결책은 없지만 우선 가장 먼저 해야할일은 **비밀번호 변경**, **AccessKey** 변경, **보안 그룹** - 인바운드 규칙에 Service 포트 외에는 다 막아주는 것이다.

여기서 추가적으로 **pem Key**까지 바꿔주면 금상첨화라 생각한다.

서비스를 운영 하면서 이러한 해킹 공격은 자주온다 생각했지만 직접 겪어 보니 당황했지만 다음에도 이런일이 발생하지 않게 항상 보안을 생각해야겠다.

---
#### 참고 및 출처

AWS Cli
- https://aws.amazon.com/ko/cli/
