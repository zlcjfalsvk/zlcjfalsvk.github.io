---
title: "Angular - Shadow Dom"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-06-18T13:00:00+09:00
author_profile: true

---
angular에서 **bind**로 인해 맨 처음 **dom**이 초기화 되고 그 다음에 **component**에 적용한 **css**가 먹혀 가끔씩 안 먹히는 경우가있다.

예를들어 다음과 같이 작성 했을 때

![1](/assets/img/posts/angular/shadowdom/1.png)

실제로는 **jquery**로 인해 밑처럼 확장이된다.

![2](/assets/img/posts/angular/shadowdom/2.png)

보면 _ngcontent-c1 이게 그 경우인데, 이런 경우 **styleUrls** 이나 **style**로 **css**를 적용 하고 싶은 경우에는 **/deep/** 이나 **>>>** 를 사용하면 적용이된다.

### Shadow Dom [이동](https://angular.io/guide/component-styles)

|이름|사용법|의미|
|:---|:---|:---|
|host|:host|현재 컴포넌트에 대한 shadow dom 선택|
|host-context|:host-context|템플릿 외부 element의 클래스 조건에 따라 현재 컴포넌트의 element를선택|
|deep|/deep/ or ::ng-deep|자식 컴포넌트에 속한 element를선택|



### 2018/8/23추가
**/deep/ ::ng-deep** (shadow-piercing descendant combinator 피어싱 자손 결합자) 는 주요 브라우저에서 지원이 제거되며, **Angular**또한 지원을 중단할 계획이다.


---
#### 참고 및 출처

Special selectors
- <https://angular.io/guide/component-styles>
- <https://stackoverflow.com/questions/37689673/angular2-styling-issues-caused-by-dom-attributes-ngcontent-vs-nghost?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa>