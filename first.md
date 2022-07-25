Elasticsearch 공부 내용 정리

# 1. Elasticsearch

## 색인
Elasticsearch를 이해하기 위해선 `색인`의 개념을 이해해야 한다.
색인은 데이터에 덧붙여 생성하는 자료 구조이고 빠르게 검색하도록 해준다. RDBMS의 index와 유사하지만 다른 점은 일반적인 색인이 아니라 `역 색인` 구조를 가진다는 점이다. 역 색인은 각각의 단어가 어디에 속해 있는지 목록을 유지하는 자료구조를 생성하는 것을 의미한다.
예를 들어 아래와 같은 형태의 raw data가 존재한다고 가정해보자

```json
{"id": 1, "content": "Lorem ipsum dolor sit amet"}
{"id": 2, "content": "Lorem dolor amet"}
{"id": 3, "content": "dolor sit amet"}
{"id": 4, "content": "Lorem sit"}
```
데이터는 id 와 content로 구성되어 있다. 이것을 역색인으로 표현하면 아래처럼 표현할 수 있다.

| 태그 | id |
| --- | --- |
| "Lorem" | 1, 2, 4 |
| "ipsum" | 1 |
| "dolor" | 1, 2, 3 |
| "sit" | 1, 3, 4 |
| "amet" | 1, 2, 3 |

역 색인은 단순히 데이터를 찾는것에 그치지 않고 문서의 개수도 얻을 수 있다.

## Elasticsearch 설치

- elasticsearch는 jdk 기반으로 작성되었기에 jdk 8 이상이 필요하다
- homebrew
    - ```bash
        $ brew install elasticsearch
      ```
- docker
    - ```bash
        $ docker run -d  \
            -p 9200:9200 \
            -p 9300:9300 -e "discovery.type=single-node"  \
            --name elasticsearch7 docker.elastic.co/elasticsearch/elasticsearch:7.9.1
      ```


실행후 `http://localhost:9200/` 접속하면 elasticsearch의 정보를 볼 수 있다.
```bash
$ curl --request GET 'http://localhost:9200/'

### Response ###
{
  "name" : "619f52a39ab4",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "bt1dv-jLRjCDQP53EYS7FQ",
  "version" : {
    "number" : "7.9.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "083627f112ba94dffc1232e8b42b73492789ef91",
    "build_date" : "2020-09-01T21:22:21.964974Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

## Document

### 색인

Elasticsearch는 JSON 형태의 문서를 저장할 수 있으며 스키마리스이기에 문서를 미리 정의할 필요는 없다.


```bash
$ curl --request PUT 'http://localhost:9200/user/_doc/1?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
    "username": "halin.lee"
}'


### Response ###
{
    "_index": "user",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "result": "created",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 0,
    "_primary_term": 1
}
```

동일한 명령어로 다시 요청해보자

```bash
$ curl --request PUT 'http://localhost:9200/user/_doc/1?pretty' \
--header 'Content-Type: application/json' \
--data-raw '{
    "username": "halin.lee"
}'

### Response ###
{
  "_index" : "user",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated", # ... 1
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```
`(1)`을 보면 `created` 가 아니라 `update`임을 확인할 수 있다. Elasticsearch는 기본적으로 upsert로 작동한다. `_id`를 key로 인식하며 동일한 데이터가 존재할 경우 삭제 후 삽입한다.


### 삭제

```bash
$ curl --request DELETE 'http://localhost:9200/user/_doc/1?pretty'

### Response ###
{
    "_index": "user",
    "_type": "_doc",
    "_id": "1",
    "_version": 4,
    "result": "deleted",
    "_shards": {
        "total": 2,
        "successful": 1,
        "failed": 0
    },
    "_seq_no": 3,
    "_primary_term": 1
}
```


### 조회

```bash
$ curl --request GET 'http://localhost:9200/user/_doc/1?pretty'

### Response ###
{
    "_index": "user",
    "_type": "_doc",
    "_id": "1",
    "_version": 1,
    "_seq_no": 4,
    "_primary_term": 1,
    "found": true,
    "_source": {
        "username": "halin.lee"
    }
}
```