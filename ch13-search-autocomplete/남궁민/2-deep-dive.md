## Trie의 한계

- 공통 prefix만 관리하는 방식
- 구현이 간단하며 직관적
- 하지만 시스템이 커질수록 메모리 공간 차지가 심해짐

## DAWG (Deterministic Acyclic Word Graph)

- Trie의 prefix 트리와 suffix 압축 트리를 결합한 구조
  - 각 트리들을 개별적으로 활용하는 것보다 적은 엣지와 노드만 활용하게 됨
- Trie에 비해 메모리 공간을 4배 가량 절약할 수 있음
- 맞춤법 검사기 등에서도 활용됨

## FST (Finite State Transducer)

- DAWG의 발전형
- 가중치 기반으로 저장하는 방식
- Elasticsearch에서 사용하는 방식
  - 각 세그먼트별로 인덱스 시점에 FST를 구축하고 디스크에 저장

## References

- [Coding The World’s Fastest Scrabble Program in Python](https://medium.com/@aydinschwa/coding-the-worlds-fastest-scrabble-program-in-python-2aa09db670e3)
- [Elasticsearch: You Complete Me](https://www.elastic.co/kr/blog/you-complete-me)
- [Elasticsearch in Action: Autocomplete suggesters](https://weng.gitbooks.io/elasticsearch-in-action/content/appendixf_using_suggesters_for_autocomplete_and_di/f2autocomplete_suggesters.html)
