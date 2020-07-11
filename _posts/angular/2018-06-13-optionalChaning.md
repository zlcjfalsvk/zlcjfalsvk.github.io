---
title: "Angular - Optional Chaning"
categories: 
  - Angular
tags:
  - Angular
  - Node.js
last_modified_at: 2018-06-28T13:00:00+09:00
author_profile: true
---
Angular 바인딩을 사용하다가 template에 NULL인 데이터가 나올수 있을 때 표현법
뒤에 **?** 를 붙인다.

**?**의 이름은  safe navigation operator 라고 한다.

    // content._data가 없으면 null || undefined 표시
    {{content?._data}}

Angular가 typescript를 채택해서 그런지 [**Optional Channing**](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining)과 같은 원리 같다.

---
#### 참고 및 출처

safe navigation operator
- https://angular.io/guide/template-syntax#safe-navigation-operator
- https://stackoverflow.com/questions/42364184/why-we-use-operator-in-template-binding-in-angular-2

Typescript Optional Channing
- https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining