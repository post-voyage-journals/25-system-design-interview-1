## Facebook - TAO

_The Associations and Objects_

- [DOCX 문서](https://docs.google.com/document/d/1Yu5zk45156favKX79uWfbE66m7--1hXf/edit?usp=sharing&ouid=112543754447173074551&rtpof=true&sd=true)

References

- [TAO: The power of the graph](https://engineering.fb.com/2013/06/25/core-infra/tao-the-power-of-the-graph)
- [TAO: Facebook’s Distributed Data Store for the Social Graph](https://scontent-icn2-1.xx.fbcdn.net/v/t39.8562-6/240856980_267453868292646_5401707815451738223_n.pdf?_nc_cat=111&ccb=1-7&_nc_sid=e280be&_nc_ohc=6JrXYQsI-4cQ7kNvwFjjK-E&_nc_oc=AdmmW35MuqqpawI8xYytvH8rx48AzPsGClITF2LfVFaO88lgj7m9A4x_3W6iB5YNcpY&_nc_zt=14)

## EdgeRank

- Facebook이 사용하던 뉴스 피드 랭킹 알고리즘
- 2011년에서 사용을 중단하고 2013년부터 10만 개 이상의 요소들을 고려하는 ML 기반 랭킹 시스템으로 이전

```js
EdgeRank = Σ(Affinity × Weight × Time Decay)
```

- Affinity Score (친밀도)
  - 사용자와 콘텐츠 제공자와의 관계 강도
  - 상호작용 횟수 기반으로 계산 (좋아요, 댓글, 공유 등)
- Edge Weight (상호작용 가중치)
  - 각 행동(Edge)마다 가중치를 다르게 판단
  - 일반적으로 `Share > Comment > Like > Click`
- Time Decay (시간 감쇠)
  - 게시물의 신선도 반영
  - 사용자의 로그인 빈도에 따라서도 감쇠율 조정
