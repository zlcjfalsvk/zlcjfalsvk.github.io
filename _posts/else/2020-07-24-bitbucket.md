---
title: "Bitbucket - SSH Key를 이용한 git clone"
categories: 
  - else
tags:
  - git
  - bitbucket
last_modified_at: 2020-07-24T13:00:00+09:00
author_profile: true
---

### 1. `ssk-keygen`을 이용한 키 만들기

![1](/assets/img/posts/else/bitbucket/1.png) 

Key가 만들어졌다.

![1_1](/assets/img/posts/else/bitbucket/1_1.png)


### 2. [Bitbucket](https://bitbucket.org/) 접속

**Profile** - **Personal setting** - **SSH 키** 이동

![2](/assets/img/posts/else/bitbucket/2.png)

![3](/assets/img/posts/else/bitbucket/3.png)

### 3. Key 추가

`ssk-keygen`에서 만들었던 Keys에서 ***.pub**의 내용을 옮긴 후 추가

![4](/assets/img/posts/else/bitbucket/4.png)

### 4. git clone

![5](/assets/img/posts/else/bitbucket/5.png)

Key를 만들 때 입력했던 pwd를 입력 해야한다. 

![6](/assets/img/posts/else/bitbucket/6.png)

---
#### 참고 및 출처

공개키 만들기
- <https://git-scm.com/book/ko/v2/Git-%EC%84%9C%EB%B2%84-SSH-%EA%B3%B5%EA%B0%9C%ED%82%A4-%EB%A7%8C%EB%93%A4%EA%B8%B0>
