---
title: "ELK - elasticSearch"
categories: 
  - ELK
tags:
  - ELK
  - elasticSearch
last_modified_at: 2020-07-09T16:00:00+09:00
author_profile: true
sidebar:
  - nav: elk-nav
---

## 정의

[ELK](https://www.elastic.co/kr/what-is/elk-stack)(ElasticSearch + Logstash + Kibana) 에서 [ElasticSearch](https://www.elastic.co/kr/what-is/elasticsearch)에 대한 글 입니다.

- 간단히 말하면 데이터 검색 엔진입니다. 
  - [Apache Lucene](https://lucene.apache.org/core/downloads.html)으로 구현됨
- REST API형태로 CRUD(Create Read Update Delete)를 합니다.
  - Select -> GET
  - Insert -> POST
  - Update -> PUT
  - Delete -> DELETE 
- JSON로 Serialized된 데이터를 저장합니다. 
- [Full Text Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html)를 통한 검색 가능
- [Inverted Index - 역색인](https://www.elastic.co/guide/en/elasticsearch/guide/master/inverted-index.html)구조를 사용하여  빠른 검색 가능
  - [Inverted Index란?](https://ko.wikipedia.org/wiki/%EC%97%AD%EC%83%89%EC%9D%B8)

## Rdbms, ElasticSearch 비교

ElasticSearch는 Rdbms와 다른 구조를 가지지만 이해하기 쉽게 다음과 같이 비교를 하곤한다.
![1](/assets/img/posts/ELK/elasticSearch/1.png)

## 설치 [이동](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html)
- 필자는 개인적으로 Docker 또는 Kubernetes를 이용하여 사용한다.[helm](https://hub.helm.sh/charts/elastic/elasticsearch)

## 검색 문법

- [ElasticSearch Sql](https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-overview.html)

- [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
  - 세분화된 query를 이용하여 aggrigation을 할 때
  - [lucene의 score](https://lucene.apache.org/core/3_5_0/scoring.html)를 이용하여 연관도를 계산해 Data를 가져온다.

## 간단 예제

**ElasticSrarch**는 **RestAPI**로 진행을 함으로써 [Postman](https://www.postman.com/)으로 **data**를 조회할 수 있다.

1. **GET** \<elasticSearch Host\>:\<port\>/ 을 호출하면 다음과 같은 data를 얻을 수 있다.

        # GET / HTTP/1.1
        # Host: 192.168.1.13:30092

        {
          "name": "hhdB3AM",
          "cluster_name": "docker-cluster",
          "cluster_uuid": "Nqe0ylKiSeufckzlxkjbAQ",
          "version": {
            "number": "6.2.2",
            "build_hash": "10b1edd",
            "build_date": "2018-02-16T19:01:30.685723Z",
            "build_snapshot": false,
            "lucene_version": "7.2.1",
            "minimum_wire_compatibility_version": "5.6.0",
            "minimum_index_compatibility_version": "5.0.0"
          },
          "tagline": "You Know, for Search"
        }

    ![2](/assets/img/posts/ELK/elasticSearch/2.png)

2. **GET** \<elasticSearch Host\>:\<port\>/_cat/ 을 호출하면 호출 가능한 **Command List**가 나온다.

        # GET /_cat/ HTTP/1.1
        # Host: 192.168.1.13:30092

        =^.^=
        /_cat/allocation
        /_cat/shards
        /_cat/shards/{index}
        /_cat/master
        /_cat/nodes
        /_cat/tasks
        /_cat/indices # 클러스터의 인덱스 목록 조회
        /_cat/indices/{index}
        /_cat/segments
        /_cat/segments/{index}
        /_cat/count
        /_cat/count/{index}
        /_cat/recovery
        /_cat/recovery/{index}
        /_cat/health
        /_cat/pending_tasks
        /_cat/aliases
        /_cat/aliases/{alias}
        /_cat/thread_pool
        /_cat/thread_pool/{thread_pools}
        /_cat/plugins
        /_cat/fielddata
        /_cat/fielddata/{fields}
        /_cat/nodeattrs
        /_cat/repositories
        /_cat/snapshots/{repository}
        /_cat/templates

3. **GET** \<elasticSearch Host\>:\<port\>/\<index\>/_search 를 호출하면 조건없는 검색이 된다.

        # GET /wins-2020.07.09/_search HTTP/1.1
        # Host: 192.168.1.13:30092

        {
            "took": 1,
            "timed_out": false,
            "_shards": {
                "total": 5,
                "successful": 5,
                "skipped": 0,
                "failed": 0
            },
            "hits": {
                "total": 6193,
                "max_score": 1.0,
                "hits": [
                    {
                        "_index": "wins-2020.07.09",
                        "_type": "doc",
                        "_id": "1594254635926",
                        "_score": 1.0,
                        "_source": {
                            "adm": "",
                            "index": "wins",
                            "consumer_group": "logstash1",
                            "lon": "-90.398177",
                            "@version": "1",
                            "lat": "39.884116",
                            "location": "39.884116,-90.398177",
                            "document_id": "1594254635926",
                            "timestamp": 1594254635936,
                            "forward": "http:://192.168.1.16:8080/rtb/win//WEB/nexage/0.85/39.884116/-90.398177/2/6/1594254635926",
                            "price": 0.85,
                            "cost": "1.0",
                            "adId": "2",
                            "@timestamp": "2020-07-09T00:30:35.936Z",
                            "pubId": "nexage",
                            "cridId": "6",
                            "topic": "wins",
                            "bidtype": "WEB",
                            "hash": "1594254635926",
                            "domain": "",
                            "origin": "worker1",
                            "adtype": "banner",
                            "serialClass": "com.jacamars.dsp.rtb.pojo.WinObject",
                            "type": "wins"
                        }
                    }
                    // 생략 ....
                ]
            }
        }

4. **3**에서 **POST**로 변경 후 **body**에 간단한 **query dsl**를 주어 호출

        # POST /wins-2020.07.09/_search HTTP/1.1
        # Host: 192.168.1.13:30092
        # Content-Type: application/json

        {
          "query": {
            "bool": {
                "must": {
                    "match": { "hash": "1594279302581" }            
                }
            }
          }
        }

        // 결과는 3과 비슷


## 마무리

다음에 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)을 이용한 호출을 정리해보겠다.


---
#### 참고 및 출처

ElasticSearch
- https://www.elastic.co/kr/what-is/elasticsearch

Rdbms vs ElasticSearch
- https://www.slideshare.net/gruter/elastic-search-performance-optimization-deview-2014