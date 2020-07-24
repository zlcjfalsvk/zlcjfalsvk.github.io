---
title: "Angular - Data Chart"
categories: 
  - Angular
tags:
  - Angular
last_modified_at: 2018-08-20T11:30:00+09:00
author_profile: true
---
Angular를 사용하다가 Data Chart를 표현해야할 때가 있었다. <br />
Google에서 Angular Data Chrart를 찾다가 좋은것을 찾아 공유한다.

모듈명은 [ngx-charts](https://swimlane.github.io/ngx-charts/#/ngx-charts/area-chart-stacked)이다.

![1](/assets/img/posts/angular/dataChart/1.png)

여러가지 차트 및 스킨, 옵션등을 제공한다.

![2](/assets/img/posts/angular/dataChart/2.png)
![3](/assets/img/posts/angular/dataChart/3.png)

기본적인 사용법은 NPM에서 모듈 추가 후에 임포트 해서 사용하는 것이다.

    npm install @swimlane/ngx-charts --save

### 사용법

    {% highlight typescript %}
    // moudle.ts에 추가를 해준다.

    import { NgxChartsModule } from '@swimlane/ngx-charts';

    @NgModule({
        imports: [
            RouterModule.forChild(routes),
            FormsModule,
            CommonModule,
            NgxChartsModule
        ],
        exports: [
            
        ],
        declarations: [
            DashboardComponent
        ],
        providers: [
            
        ]
    })
    export class DashboardModule {}
    {% endhighlight %}

그다음에 data.ts 파일을 만들어준다.<br />
필자는 경우에는 밑의 사진처럼 만들었기 때문에 다음과 같은 데이터 형식으로 만들었다.

![4](/assets/img/posts/angular/dataChart/4.png)

    {% highlight typescript %}

    // name과 series를 만들어주고 그 안에 name과 value를 만들면 각각 가로축과 세로축이 된다.

    export var day = [
        {
        "name": "매출",
        "series": [
            {
            "name": "06-01",
            "value": 730000
            },
        ]
        },
    ];

    export var month = [
    {
        "name": "매출",
        "series": [
        {
            "name": "2017-10",
            "value": 7300000
        },
        ]
    },
    ];
    ------------------------------------------
    import { day, month } from "./data";

    // 컴포넌트에 사용할 변수 선언
    day: any;
    month: any;

    // 객체 복사
    try {
        Object.assign(this, { day, month });
    } catch(e) {}
    {% endhighlight %}

Template 에는 다음과 같이 추가해준다.

    {% highlight html %}
    <!- ngx-charts-lie-chart를 타고 들어가면 옵션및 데이타 타입들을 볼 수 있다.
        그리고 사용할 옵션들을 컴포넌트에 선언하여 값들을 설정하면 된다. -->

    <ngx-charts-line-chart 
        [view]="viewRight" 
        [scheme]="colorScheme" 
        [results]="month" 
        [gradient]="gradient" 
        [xAxis]="showXAxis" 
        [yAxis]="showYAxis"
        [legend]="showLegend" 
        [legendTitle]="legendTitle"
        [showXAxisLabel]="showXAxisLabel" 
        [showYAxisLabel]="showYAxisLabel" 
        [xAxisLabel]="xAxisLabel"
        [yAxisLabel]="yAxisLabel" 
        [autoScale]="autoScale" 
        (select)="onSelect($event)">
    {% endhighlight %}


---
#### 참고 및 출처

Angular Data Chart

- <https://github.com/swimlane/ngx-charts>
- <https://swimlane.github.io/ngx-charts/#/ngx-charts/polar-chart>
