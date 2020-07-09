---
title: "Angular - vsCode로 디버깅하기"
categories: 
  - angular
tags:
  - angular
  - vscode
last_modified_at: 2018-06-14T09:00:00+09:00
author_profile: true
---

1. 확장에서 'Debugger for Chrome' 검색하여 설치 한다.

   ![1](/assets/img/posts/angular/debug/1.png)   
2. 디버그 화면으로 이동하여 구성추가를 누르면 launch.json 파일이 생성된다.
3. Launch.json 파일에 아래 설정을 추가 한다.

        {% highlight json %}
        {
            "version": "0.2.0",
            "configurations": [
                {
                    "name": "Launch localhost with sourcemaps",
                    "type": "chrome",
                    "request": "launch",
                    "url": "http://localhost:4200/home",
                    "sourceMaps": true,
                    "webRoot": "${workspaceRoot}"
                }
            ]
        }
        {% endhighlight %}

4. **tsconfig.json** 파일에서 **"sourceMap": true** 설정을 추가하거나 수정한다.
5. 프로젝트를 실행 한다.
6. 디버깅 시작 버튼을 누른다. 그럼 크롬 브라우저가 실행 된다.
7. 원하는 **Typescript** 코드 라인에 **Break Point** 를 설정 한다.
8. **Break Point** 설정한 로직이 타도록 크롬 브라우저에서 페이지 이동을 한다. 

---
#### 참고 및 출처

vsCode
- https://code.visualstudio.com/docs/nodejs/angular-tutorial