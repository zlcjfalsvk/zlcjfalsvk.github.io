---
title: "Angular - Angular Security Sanitization"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-07-08T13:00:00+09:00
author_profile: true
---

Angular를 사용하다 string안에 **html tag**가있고 그것을 화면에 적용 하고 싶으면 **[innerHTML]** 속성을사용하면된다.

    {% highlight HTML %}
    <td [innerHtml]="content?.reply">
    {% endhighlight %}

Angular에서 innerHTML 속성을 이용해 출력한 결과는 불완전하다.
왜냐하면 Angular가 이들 요소를 잠재적인 위험 요소로 판단해 [Sanitization](https://angular.io/guide/security#sanitization-and-security-contexts)을 적용했기 때문이다.

[Sanitization](https://angular.io/guide/security#sanitization-and-security-contexts)은 데이터의 잠재된 위험 요소를제거하는기능이다.

Angular에서 팝업 및 데이터가 수시로 바뀌는 **innerHTML**을 사용하는 경우에 **id** 및 **name**에 대한 **security**에 대한 경고가 콘솔에 나온다.
그럴경우 [Sanitization](https://angular.io/guide/security#sanitization-and-security-contexts)을 예외적으로 허용 해줘야하는데 그 메서드가 **bypassSecurityTrustHtml(v)** 이다.


    {% highlight typescript %}
    
    // Component.ts
    import { Pipe, PipeTransform } from "@angular/core";
    import { DomSanitizer, SafeHtml } from '@angular/platform-browser';
    
    // 파이프를만들어준후사용해야한다.
    @Pipe({
    name: 'sanitizeHtml'
    })
    export class SanitizeHtmlPipe implements PipeTransform {
        constructor(private _sanitizer:DomSanitizer) {
        }
        transform(v:string):SafeHtml {
            return this._sanitizer.bypassSecurityTrustHtml(v);
        }
    }

    // Template in use...
     <td [innerHtml]="content?.reply | sanitizeHtml">

    {% endhighlight %}


**Sanitization**의 사용 목적은 [XSS(Cross-Site-Scripting)](https://namu.wiki/w/XSS)공격을 예방하기 위함이다.<br />
Angular에서는 Sanitization And Security Contexts의 다음과 같이 정의한다.

    HTML is used when interpreting a value as HTML, for example, when binding to innerHtml.
    Style is used when binding CSS into the style property.
    URL is used for URL properties, such as <a href>.
    Resource URL is a URL that will be loaded and executed as code, for example, in <script src>.

그리고 **Sanitization**을 사용하기 위해 위에서와 같이 **DomSanitizer**를 불러와 사용하는데 다음과 같은 함메서드들이 있다.

- bypassSecurityTrustHtml
- bypassSecurityTrustScript
- bypassSecurityTrustStyle
- bypassSecurityTrustUrl
- bypassSecurityTrustResourceUrl

---
#### 참고 및 출처

Angular Sanitization

- https://angular.io/guide/security#sanitization-and-security-contexts
