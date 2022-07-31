# 3. aggregation

집계는 metrics와 bucket이라는 두 가지 분류로 나눌 수 있다. 지표 집계는 문서 그룹의 통계 분석을
나타낸다. 버킷 집합은 하나 또는 여러 개의 버킷에 일치하는 문서를 나눈 다음 각 버킷의
문서의 수를 돌려준다.

## Aggrgation 규칙

- json 요청에 집계를 정의하고 `aggregation` 또는 `aggs`로 표현한다.
- 집계는 질의 결과에 실행한다. 즉 질의에 일치하지 않는 문서는 처리되지 않는다.

## terms aggregation

문서에서 사용하는 상위 몇개의 정보를 가지고 올 때 사용한다. 문서의 필드를 기반으로 값이 아닌 단어에 대해 계산한다. `text`필드에 대해서는 사용할 수 없으며 반드시 `keyword`타입만 가능하다.

```json
GET /_search
{
  "aggs": {
    "genres": {
      "terms": { "field": "genre" }
    }
  }
}
```

```json
{
  "aggregations": {
    "genres": {
      "doc_count_error_upper_bound": 0,   
      "sum_other_doc_count": 0,           
      "buckets": [                        
        {
          "key": "electronic",
          "doc_count": 6
        },
        {
          "key": "rock",
          "doc_count": 3
        },
        {
          "key": "jazz",
          "doc_count": 2
        }
      ]
    }
  }
}
```

만약 문서의양 혹은 검색되는 필드에 단어양이 많다면 메모리에 적재될 수 있도록 heap size를 조절하는 것이 중요하다.
기본적으로 10개의 단어만 돌려주는데 `size`파라미터를 `0`으로 설정하면 모든 단어를 돌려주게된다. 하지만 높은 cardinality 가진 필드에서 사용할 경우 cpu를 많이 사용하고 네트워크를 포화시킬 수 있기에 위험하다.

### shard_size
집계를 하는 방식은 각 샤드에서 상위 데이터를 뽑아낸 뒤 그것을 합쳐서 다시 집계하는 방식으로 진행된다. 따라서 일부 샤드에 등록되지 못한 키워드들이 존재하게 되고 이것은 결과의 정확도를 떨어트릴수 있다.


|   |   |   |   
|---|---|---|
| 노드1 | 노드2 | 노드3 |
| java:11 | C:10 | kotlin:15|
| c:5 | java:9 | c:3 |
| kotlin:4 | | 

위와 같이 식으로 데이터가 저장되어 있고 `size`를 2롤 설정한다면  `java:20`, `kotlin:19` 순으로 기대값을 예상할 것이다. 그러나 shard_size에 따라서 집계는 변경될 수 있는데 만약, shard_size가 2라면 elasticsearch는 내부적으로 샤드 1의 `kotlin:4` 를 집계하지 않을 것이고 실제 결과는  `java:20`, `c:18`이 될 것이다. 이런한 내용을 방지하기 위해선 `shard_size`를 증가시켜주는 것으로 해결할 수 있다. 다만 성능 하락이 올 수 있음에 유의한다.

