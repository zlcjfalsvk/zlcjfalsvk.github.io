---
title: "Angular - Component Communication"
categories: 
  - angular
tags:
  - angular
last_modified_at: 2018-05-30T13:00:00+09:00
author_profile: true
---
angular는 컴포넌트 형식이라 모든 부분이 컴포넌트로 이루어져있다.

그런 컴포넌트 끼리 데이터를 주고받을 때 사용하는것이 **Component Communication**이다.

### 1. @ViewChild와 @ViewChildren

컴포넌트 템플릿에 배치된 자식요소(자식 컴포넌트, 디렉티브, 네이티브 DOM 요소)를 [**ViewChild**](https://angular.io/api/core/ViewChild)라고 한다. 이름에서 알 수 있듯이 [**@ViewChild**](https://angular.io/api/core/ViewChild)는 탐색 조건에 부합하는 1개의 요소를 취득할 수 있고, [**@ViewChildren**](https://angular.io/api/core/ViewChildren)는 탐색 조건에 부합하는 여러개의 요소를 한꺼번에 취득할 수 있다.

**@ViewChild**(탐색대상 클래스명) 프로퍼티명: 탐색대상 클래스명;

**@ViewChildren**(탐색대상 클래스명) 프로퍼티명: QueryList<탐색대상 클래스명>;

### 2. 변수값전달

![1](/assets/img/posts/angular/componentCommu/1.png)

왼쪽부터
1. 부모에서 자식
2. 자식에서 부모
3. 원거리 컴포넌트 절달

##### 1. 부모에서 자식

    {% highlight typescript %}    
    부모 Component
    //[]사이의 명은 은 자식과 일치
    <app-child [childMessage]="parentMessage"></app-child> 

    자식 Component
    @Input() childMessage: string; //받을 값
    {% endhighlight %}

##### 2. 자식에서 부모

자식 컴포넌트는 [**@Output**](https://angular.io/api/core/Output) 데코레이터와 함께 선언된 컴포넌트 프로퍼티(출력 프로퍼티)를 EventEmitter 객체로 초기화한다. 

그리고 부모 컴포넌트로 상태를 전달하기 위해 emit() 메소드를 사용하여 이벤트를 발생시키면서 상태를 전달한다. 

부모 컴포넌트는 자식 컴포넌트가 전달한 상태를 이벤트 바인딩을 통해 접수한다.

![2](/assets/img/posts/angular/componentCommu/2.png)


##### 3. 원거리 컴포넌트가 데이터 전달

 - 서비스중재자패턴(Service Mediator pattern) 을이용
    => 서비스객체를이용 set get 이용

    구글로 키워드를 검색할 때 Component Communication으로 검색하면 많이 나온다

---
#### 참고 및 출처
- https://www.slideshare.net/AmritaChopra1/angular-2-component-communication-talk-by-rob-mcdiarmid