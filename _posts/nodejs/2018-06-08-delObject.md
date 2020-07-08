---
title: "Object 중복객체 제거"
categories: 
  - node.js
tags:
  - node.js
last_modified_at: 2018-06-08T13:00:00+09:00
author_profile: true
---
Json 형태Object가 있을 때 중복된 key를 해야할 때가 있다.

그럴때 나는 밑의 방법처럼 사용한다

1. filter를 이용한 방법

        // GoodsCategory라는 Data가 주어줬을 때
        category:GoodsCategory= new GoodsCategory();

        /**
          * Data에 따라 Object(this.category) || Array(this.category)를 사용할 수 있다.
          */

        get this.mainCategory = Object(this.category).filter((array, index, self) => self.findIndex((t) => {
            // mainCategoryName, mainCategoryId이 같은 data 제거
            return t.mainCategoryName === array.mainCategoryName &&
             t.mainCategoryId ===  array.mainCategoryId; 
            }) === index 
        );

2. Set을 이용한 방법

    Data의 중복을 허용하지 않는 Set 자료구조를 이용한 방법이다.

        this.mainCategory = Array.from(new Set(Object(this.category).map(name => name.mainCategoryName)));

        // 또는
        Array.from(new Set(Object(this.category).map(JSON.stringify))).map(JSON.parse);

    

---
#### 참고 및 출처

- https://code.i-harness.com/ko/q/21dbf7

Array.from
- https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/from

