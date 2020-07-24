---
title: "Angular - Http Interceptor"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-08-22T13:00:00+09:00
author_profile: true
---
Angular 4.3부터 추가된 [HttpClientModule](https://angular.io/api/common/http/HttpClientModule)에는 http 요청에 인터셉터를 적용할 수 있다.

인터셉터를 간단히 설명하면 api나 특정 data를 요청 할 때 그 요청을 가로채는 역할을 한다. 인터셉터를 사용하기 위해서는 [HttpInterceptor](https://angular.io/api/common/http/HttpInterceptor)를 구현해야 한다.

    {% highlight typescript %}
    interface HttpInterceptor {
        // intercept의 첫번째 인자는 처리할 요청을 받고, 두번째는 다음 인터셉터를 가리키는 HTTP 핸들러다.
        intercept(req: HttpRequest, next: HttpHandler): Observable<httpevent<any>>
    }
    {% endhighlight %}

### 예제

    {% highlight typescript %}
    export class HeaderInterceptor implements HttpInterceptor {

        intercept(req: HttpRequest<any>, next: HttpHandler): Observable<httpevent<any>> {

            const token = localStorage.getItem('token');
            let newHeader: HttpHeaders = req.headers;
            
                    if (token) {
                newHeader = newHeader.set('X-Auth-Token', token);
            }
                    // intercept의 req는 이뮤터블(인스턴스의 내용이 변하지 않는다 <-> 뮤터블)이다.
                    // 헤더를 추가하거나 변경을 하려면 clone 함수를 통해 새로운 httpRequest객체를 생성해야 한다.
            const clonedRequest = req.clone({headers: newHeader});

                    // next.handle 로부터 인터셉터를 체이닝(엮다, 묶다)한다.
            return next.handle(clonedRequest); 
        }
    }

    // 모듈 추가
    @NgModule({
        imports: [
            HttpClientModule,
        ],
        providers: [
            LoaderService,
            {
                provide: HTTP_INTERCEPTORS, // InjectionToken
                useClass: HeaderInterceptor,

                            // HTTP_INTERCEPTORS가 단일 값이 아닌 배열을 주입한는 multiprovider에 대한 토큰임을 설정한다(영알못 해석)
                multi: true,
            }
        ],
    })
    export class CoreModule {
    }

    {% endhighlight %}

이런식으로 헤더에 대한 정보를 얻기위해 인터셉터를 사용할 수도 있고, SPA에서 API를 통해 데이터를 받을 때 로딩표시를 표현하기 위해 사용할 수도 있고 다양하게 사용할 수 있다.


---
#### 참고 및 출처

Angular Interceptor

- <http://han41858.tistory.com/39>
- <https://angular.io/guide/http#intercepting-requests-and-responses>
