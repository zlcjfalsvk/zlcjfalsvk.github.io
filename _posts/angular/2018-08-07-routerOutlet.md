---
title: "Angular - Router Outlet"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-08-07T11:30:00+09:00
author_profile: true
---
웹페이지를 만든다면 모달과 팝업을 만드는 일은 흔한 일인것 같다.
**Angular**에서 적용하는 다음과 같다.

1. **bootstrap.js** 또는 **bootstrap.min.js** 를 **import**
2. 하위 컴포넌트 지정
3. 상위 컴포넌트의 이벤트 버튼에다가 **data-toggle** 및 **data-target** 지정

이런식으로 사용해도 되지만 나는 좀 더 **Angular** 고유의 **Popup**을 띄우고 싶어 여러가지 방법을 찾아가 **Angular Router Popup** 을 찾았다.

Angular에서는 [Router Outlet](https://angular.io/guide/router#nesting-routes)을 다음과 같이 소개한다.

    The RouterOutlet is a directive from the router library that marks the spot in the template where the router should display the views for that outlet.

\<**router-outlet**\>이라는 **Directive**를 사용한다.

기존의 Router가 컴포넌트의 전환이라면,<br />
**Router Oultet**은 **컴포넌트의 전환**이 아닌 **상위 컴포넌트** 위에 **하위 컴포넌트**를 띄운다는 개념이다.

![1](/assets/img/posts/angular/outlet/1.png)


### 사용방법

    // router 설정
    // Routing Module

    [
            {
                path: 'notice',
                component: NoticeComponent,
                children: [
                    {
                        path: 'add',
                        component: NoticeAddComponent,
                        outlet: 'sub' // outlet 명칭을 sub이라고 지정    나중에 path는 /notice/(sub:add) 이런식으로 나오게 된다.
                    }, {
                        path: 'view/:sq',
                        component: NoticeViewComponent,
                        outlet: 'sub'        //  /notice/(sub:view/sq)
                    },
    ]

    // HTML

    <router-outlet name="sub"></router-outlet>

    // navigate 
    // 방법 1 - Template 사용
    <a [routerLink]= "(['/board/notice',{ outlets: { sub: 'add' }}])" class="btn btn_b">+ 공지추가</a>

    // 방법 2 - Component.ts 에서 사용
    this.router.navigate([{ outlets: { sub: [ 'add' , param?] }}]);  

### View

상위 컴포넌트(/notice)가 다음과 같이 이루어져 있고 수정버튼을 클릭 시

![2](/assets/img/posts/angular/outlet/2.png)

Router Outlet을 이용해 하위 컴포넌트를 띄운다. <br />
이 때 **상위 컴포넌트**에서 **하위 컴포넌트**의 전환이 아닌 상위 컴포넌트 위에 하위 컴포넌트를 뛰우는것 이므로 **URl**이 다음과 같다.

    /notice/(sub:edit/param)

![3](/assets/img/posts/angular/outlet/3.png)

### 또 다른 방법
[Angular Resource](https://angular.io/resources)에 기재되어 있는 [Angular Bootstrap](https://ng-bootstrap.github.io/#/home), [Angular Material](https://material.angular.io/)를 사용해 modal 및 popup을 생성하는 방법도 있다.

---
#### 참고 및 출처

Angular Router Outlet
- <https://angular.io/guide/router#nesting-routes>