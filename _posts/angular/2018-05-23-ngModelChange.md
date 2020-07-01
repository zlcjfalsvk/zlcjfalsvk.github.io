---
title: "Angular - ngModelChange"
categories: 
  - angular
tags:
  - angular
last_modified_at: 2018-05-23T13:00:00+09:00
author_profile: true
---

Angular 를 사용하다 보면 양방향 바인딩을 사용하는 **ngModel**을 자주 사용할 것이다.

사용법은 \[\(ngModel\)\]="data" 이런식으로 데이터를 제어할 때 사용하는데 <br/>**양방향 바인딩**을 사용하기 때문에 html의 \<input\> 이나 \<select\> 그외 버튼 이런거를 사용할 때마다 값을 변경할 때 유용하다.


하지만 data 뿐만아니라 함수 호출을 사용하고싶은 때 에는 어떻게 사용해야하나?<br />
그럴 때 **ngModelChange** 를 사용하면 된다.

사용법은

    <select id="num1" #inputnum1="" [(ngmodel)]="displayValue.num1" 
        (ngmodelchange)="changeValueNum1($event)">
    </select>

이런식으로 사용한다.

위의 예제로는 select를 선택할 때마다 **(ngModelChange)="changeValueNum1($event)"** 이 호출된다.

---
#### 참고 및 출처

