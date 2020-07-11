---
title: "Angular - Router Guard"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-07-13T13:00:00+09:00
author_profile: true
---
Angular의 **View**의 전환으로 사용하는 [router](https://angular.io/guide/router)가 있다.

그리고 이 **View**의 접근제어를 할 수 있는 기능이 있는데 [라우터 가드(Router Guard)](https://angular.io/guide/router#preventing-unauthorized-access)가 있다.

라우터 guard에는 다음과 같은 인터페이스 들이 있다.

- [CanActivate](https://angular.io/api/router/CanActivate)
- [CanActivateChild](https://angular.io/api/router/CanActivateChild)
- [CanDeactivate](https://angular.io/api/router/CanDeactivate) # 라우터의 접근권한을 검사
- [Resolve](https://angular.io/api/router/Resolve) # 라우터 변경시 호출되는 라우트
- [CanLoad](https://angular.io/api/router/CanLoad) #라우트 데이터를 가져와 컴포넌트에 제공

로그인 구현이나 접근 제어를 하기위해 **CanActivate**를 이용해서 구현을 해보았다.

    {% highlight json %}
    # app.module.ts
    
    { 
        path: 'mypage', 
        loadChildren: './mypage/mypage.module#MypageModule', 
        canActivate: [ AuthLoginGuard ], 
        data: {
            roles: ['마이페이지'] 
        }
    }
    {% endhighlight %}

라우트를 정의할 때 **CanActivate**를 이용해서 권한이 필요하다고 정의하고 **data**를 이용해 파라미터를 설정한다.


    {% highlight typescript %}
    # app.auth-login.guard.ts

    import { Injectable } from '@angular/core';
    import { CanActivate, ActivatedRouteSnapshot, RouterStateSnapshot } from '@angular/router';
    import { Observable } from 'rxjs/Observable';
    import { Router } from '@angular/router';
    import { Globals } from '../app.globals';

    @Injectable()
    export class AuthLoginGuard implements CanActivate {

        constructor(
            private router: Router,
            private globals:  Globals
        ) { }
        
        canActivate(
            route: ActivatedRouteSnapshot,
            state: RouterStateSnapshot
        ): Observable | Promise | boolean {
            let roles = route.data["roles"] as string;   // module에서 선언한 data: 밑의 roles 값을 받아온다.
                    
            if ( localStorage.getItem('login.token')) {
                return true;
            } else {
                alert(roles +'는 로그인이 필요합니다.' + '\n로그인 창으로 이동합니다.');
                // this.router.navigate(['/member/login']);
                this.globals.link('/member/login', state.url);
                return false;
            }
        }
    }
    {% endhighlight %}

이런식으로 선언해서 특정 **Url**로 **View**를 접근할 때 **CanActivate**로 인해서 이동하기전에 **Guard**로 이동해 로직을 점검하고 이동할지 말지를 정한다.

이런식으로 비정상적인 루트를 통해 **View**에 접근하려는 것도 **Guard**를 통해 제어할 수 있다.

---
#### 참고 및 출처

Angular Router Guard

- https://angular.io/guide/router#preventing-unauthorized-access
