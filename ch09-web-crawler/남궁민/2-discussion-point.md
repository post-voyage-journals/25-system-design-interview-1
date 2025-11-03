## 1. 크롤러 실제 사용 사례

- 이전에는 크롤러가 일회성 스크립트 위주일 것이라고 생각해왔음
- 크롤러는 일반 사용자의 접근 패턴이 아니기 때문에 다소 부정적인 이미지
- 실제로 크롤러를 분산 환경으로 설계 및 구현하는 실제 사례가 있을까?

### 사례 찾아보기

- Google
  - 수천 대 서버로 구성된 분산 크롤러
  - 매일 수백억 페이지 처리
- Bing
- IRLbot

## 2. 크롤러를 구성하는 컴포넌트들의 실제 인프라 구성

- 책에서는 크롤러 시스템을 구성하는 핵심 컴포넌트들의 역할을 소개
- 실제 구현체에 대한 소개는 없음

### 컴포넌트 구성해보기

- Seed URL & 미수집 URL 저장소: Redis Sorted Set
- HTML 다운로더: Python, Node.js, Go 등을 활용한 Worker 구현
- 콘텐츠 파서: Python Worker
- 중복 콘텐츠 판별 장치: Redis Set + Bloom Filter
- 콘텐츠 저장소: S3(HTML 원본) + RDBMS(메타데이터)
- URL 추출기: Python Worker
- 방문 URL 저장소: RDBMS
