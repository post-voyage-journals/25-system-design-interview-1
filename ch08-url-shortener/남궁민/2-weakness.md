# 설계 허점 및 개선 방안

## URL 관련 데이터를 관계형 데이터베이스에 저장하는 것이 적절할까?

- 책에서는 단축 URL 및 원본 URL을 관계형 데이터베이스에 저장하는 방식을 추천함
- 하지만 테이블 간 복잡한 관계가 필요 없으며, 컬럼 숫자도 적음

### 개선 방안

- Redis, DynamoDB, MongoDB, Cassandra 등의 NoSQL 기반 저장소를 활용하면 높은 가용성 및 낮은 지연 속도를 보장할 수 있을 것
- 다만 데이터 무결성이 중요하고, 통계 데이터와의 연관관계가 필요하다면 관계형 데이터베이스가 더 적합할 수 있음
- 실제로는 캐시 계층 이외에도 2가지 방식의 DB를 혼용해서 사용하기도 함 (Bitly → MySQL + MongoDB)
  - [Bitly: A Deep Dive into the URL Shortening Service](https://medium.com/@entrustech/bitly-a-deep-dive-into-the-url-shortening-service-ec1ffd4cd3f4)
