---
title: "Spring boot - JPA DTO Mapping ( Native Query to DTO )"
categories: 
  - java
tags:
  - java
  - Spring Boot
last_modified_at: 2020-01-05T13:00:00+09:00
author_profile: true
---
JPA에 대해 간단히 소개해 드리자면, Java에서 DB를 객체로 관리하기 위해 사용되는 **ORM**입니다.

객체를 이용하여 Data를 관리하기 때문에 프로그래밍 단에서 제어가 가능해지고 개발자는 비즈니스 로직에 집중할 수 있으며, 배우기만 하면 빠른 개발이 가능하다고 합니다. 

그리고 **Query Method**라고 특정 규칙을 이용한 **Method Name**을 설정함으로써, 쿼리를 작성 안 할 수 있다는 큰 장점이 있지만, 복잡한 쿼리 및 Join 등을 이용할 시에는 **@Query**라는 어노테이션을 사용하거나, **QueryDsl**을 이용해야 하는데, 이거에 대한 학습이 필요합니다.

그리고 이 글을 쓰는 이유가 바로 JPA를 이용하면 [영속성(Spring 영속성)](https://openjpa.apache.org/builds/1.2.3/apache-openjpa/docs/jpa_overview_emfactory_perscontext.html)을 이용하여 Data 관리를 해야 하는데,<br />
Entity가 아닌 내가 원하는 **DTO**로 Data를 받고 싶을 때, 어떻게 해야 하는지 찾아보다가 발견하여 정리하기 위해 글을 작성하는 겁니다.

우선 방법은 여러 가지가 있습니다. 제가 아는 방법들은 

### 1.  @SqlResultSetMapping를 이용하여 DTO로 받기

### 2.  [QLRM 라이브러리](https://github.com/72services/qlrm) 이용하기

이 정도입니다. 이 두 가지 방법의 **공통점**은 **EntityManager**를 이용한다는 점이며, 다만 **Result**를 개인이 만든 DTO로 받을 때 어떻게 받느냐의 차이입니다.


## SqlResultSetMapping을 이용한 방법

1. 내가 이용하고 싶은 DTO를 만든다. ( 생성자도 만들어 줘야 함 )

        {% highlight java %}
        @Data
        public class RecommandPostList {
            
            private Long postId;
            private Long userId;
            private Integer phoneNumber;
            private Date registerTime;
            private Geometry location;
            
            public RecommandPostList(
                    Long postId,
                    Long userId,
                    Integer phoneNumber,
                    Date registerTime,
                    Geometry location
                    ) {
                this.postId = postId;
                this.userId = userId;
                this.phoneNumber = phoneNumber;
                this.registerTime = registerTime;
                this.location = location;
            }

        }        
        {% endhighlight %}

2. @SqlResultSetMapping을 이용하여 Mapping? ( Mabatis의 resultMap과 비슷)
- 다만 @Entity가 선언된 Class에서 선언을 해줘야 합니다.

        {% highlight java %}
        @Data
        @Entity(name="t_posts")

        // 내용을 보시면 Mapping Name을 RecommandPostListMapping로 선언했으며, 해당 Query Result 들을 Matching 시켜주는 그런 역할입니다.
        @SqlResultSetMapping(
                name="RecommandPostListMapping",
                classes = @ConstructorResult(
                        targetClass = RecommandPostList.class,
                        columns = {
                                @ColumnResult(name="postId", type = Long.class),
                                @ColumnResult(name="userId", type = Long.class),
                                @ColumnResult(name="phoneNumber", type = Integer.class),
                                @ColumnResult(name="registerTime", type = Date.class),
                                @ColumnResult(name="location", type = Geometry.class),
                        })
        )


        public class Post {
        // 생략
        }        
        {% endhighlight %}

3. DTO로 Mapping 하기

        {% highlight java %}
        @Service
        public class DtoSampleService{

            @PersistenceContext
            EntityManager em;
            
            public List<RecommandPostList> getList() {
                
                String query = "생략";
                // 아까 선언해둔거 이용
                Query query = em.createNativeQuery(q, "RecommandPostListMapping");
                List<RecommandPostList> list = query.getResultList();   
                return list;    
            }
        }    
        {% endhighlight %}

끝입니다. 간단해 보이나요? @SqlResultSetMapping을 이용한 방법의 단점은 DTO를 만들고 생성자 또한 만드는 작업이 매우 귀찮으며, 결국에는 영속성에 의존한다는 점입니다. (@Entity 클래스에 선언해야 함)

귀찮은 작업이 두 가지나 있는데, 불가피 하게 귀찮은 작업을 해야 한다면 두가지 보다 한 가지만 하는 게 낫겠죠?

그래서 나온 게 **QLRM**이라는 라이브러리입니다.

## QLRM을 이용한 방법

QLRM을 이용하면 DTO 생성 및 생성자만 만들어주면 됩니다.

1. Dependency 추가 

    	compile 'org.qlrm:qlrm:2.0.2'

2. 내가 이용하고 싶은 DTO를 만든다. ( 생성자도 만들어 줘야 함 ) 

        {% highlight java %}
        @Data
        public class RecommandPostList {
            
            private BigInteger postId;
            private BigInteger userId;
            private Integer phoneNumber;
            private Date registerTime;
            
            public RecommandPostList(
                    BigInteger postId,
                    BigInteger userId,
                    Integer phoneNumber,
                    Date registerTime
                    ) {
                this.postId = postId;
                this.userId = userId;
                this.phoneNumber = phoneNumber;
                this.registerTime = registerTime;
            }

        }        
        {% endhighlight %}

3. 사용

        {% highlight java %}
        @Service
        public class DtoSampleService{

            @PersistenceContext
            EntityManager em;
            
            public List<RecommandPostList> getList() {
                
                String query = "생략";
                
                JpaResultMapper result = new JpaResultMapper();
                Query query = em.createNativeQuery(q);
                List<RecommandPostList> list = result.list(query, RecommandPostList.class);
                return list;    
            }
        }        
        {% endhighlight %}

## 주의사항

**QLRM**을 사용하기 위한 DTO 부분 보시면 **Long**이 아닌 **BigInteger**로 선언한거 보이시나요?

**QLRM** 라이브러리의 [ClassGenerator.class](https://github.com/72services/qlrm/blob/master/src/main/java/org/qlrm/generator/ClassGenerator.java)를 보시면 DB의 Data Type을 가져오는거 같습니다. 그리고 내부 클래스를 통해 **Java Primitive Type**으로 변환을 시켜주더라고요.

다만 Mysql Point 타입 같은거는 지원을 안해줘서 이러한 타입들을 이용하신다면 **@SqlResultSetMapping**를 이용해야 합니다.


---
#### 참고 및 출처

QLRM
- <https://github.com/72services/qlrm>

EntityManager
- <https://docs.oracle.com/javaee/7/api/javax/persistence/EntityManager.html#createNativeQuery-java.lang.String->



example 
- <https://www.programcreek.com/java-api-examples/?class=javax.persistence.EntityManager&method=createNativeQuery﻿>
