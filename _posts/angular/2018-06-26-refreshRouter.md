---
title: "Angular - Router Refresh"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-06-26T13:00:00+09:00
author_profile: true
---

**SPA(Single Page Application)** 인 **Angular**는 자신의 라우터링크를 누르면 
아무 반응이 없는데 이 대신 새로고침으로 하게 하는 편법이 있다.

    {% highlight typescript %}
    import { Router, NavigationEnd } from '@angular/router';

    // class 생략...
    ngOnInit(){
        this.router.routeReuseStrategy.shouldReuseRoute = function(){
            return false;
        };
        
        this.router.events.subscribe((event) => {
            if (event instanceof NavigationEnd) {
                this.router.navigated = false;
                window.scrollTo(0, 0);
            }
        });
    }
    {% endhighlight %}

---
#### 참고 및 출처
- <https://github.com/angular/angular/issues/13831>
