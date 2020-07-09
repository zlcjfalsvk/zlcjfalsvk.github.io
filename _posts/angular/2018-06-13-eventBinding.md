---
title: "Angular - Event Binding"
categories: 
  - angular
tags:
  - angular
last_modified_at: 2018-06-13T13:00:00+09:00
author_profile: true
---
Angular의 대표적인 [이벤트 바인딩](https://angular.io/guide/template-syntax#event-binding)의 종류들

**myMethod()** 는 내가 호출하고 싶은 **function**

(focus)="myMethod()"  // An element has received focus<br />
(blur)="myMethod()"   // An element has lost focus<br />
(submit)="myMethod()"  // A submit button has been pressed<br />
(scroll)="myMethod()"

(cut)="myMethod()"<br />
(copy)="myMethod()"<br />
(paste)="myMethod()"<br />

(keydown)="myMethod()"<br />
(keypress)="myMethod()"<br />
(keyup)="myMethod()"

(mouseenter)="myMethod()"<br />
(mousedown)="myMethod()"<br />
(mouseup)="myMethod()"<br />

(click)="myMethod()"<br />
(dblclick)="myMethod()"

(drag)="myMethod()"<br />
(dragover)="myMethod()"<br />
(drop)="myMethod()"

(change) = "changemonths($event)"


---
#### 참고 및 출처

Angular Event Binding
- https://angular.io/guide/template-syntax#event-binding

Mozilla Event Bindind
- https://developer.mozilla.org/en-US/docs/Web/Events