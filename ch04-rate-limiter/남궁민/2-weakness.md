# 설계 허점 및 개선 방안

## 1. Sorted Set 만으로 동시성 제어를 할 수 있는가?

> 락 대신 쓸 수 있는 해결책이 두 가지 있는데, 하나는 루아 스크립트(Lua script)이고 다른 하나는 정렬 집합(sorted set)이라 불리는 레디스 자료구조를 쓰는 것이다.
> ⎯ P. 71 (3단계 상세 설계 - '경쟁 조건' 부분)

- 책에서는 [System Design — Rate limiter and Data modelling](https://medium.com/@saisandeepmopuri/system-design-rate-limiter-and-data-modelling-9304b0d18250) 글을 인용하면서, Redis Sorted Set만으로 동시성 제어가 가능하다고 주장함
- 하지만 인용글에서는 Lua Script 없이 MULTI/EXEC 방식으로 처리
- 이 방식은 여러 커넥션이 동시에 성립된 경우 명령어들 간의 인터리빙으로 인해 동시성 제어가 실패할 수 있음

**개선 방안**

- Lua Script와 함께 Sorted Set 자료구조 활용

## 2. API Gateway 활용 방식에만 초점이 맞춰져 있다

- 책에서는 API Gateway를 미들웨어로 활용하는 방식에만 주안점을 두고 있음

**개선 방안**

- API Gateway 이전에 CDN 계층에서도 Rate Limit을 적용할 수 있을 것
- 애플리케이션 계층에도 Rate Limit을 적용할 수 있지만, 기능별 세밀한 제어가 필요할 때 도입해도 충분할 것
