## C10K/C10M 문제

_동시에 1만 개, 1천만 개의 동시 WebSocket 커넥션을 어떻게 유지할 것인가?_

### C10K 문제

- 초기에는 connection-per-thread 모델 사용
  - 각 커넥션마다 새로운 Thread 사용
  - Context Switching 오버헤드 존재
  - Thread 고갈 시 새로운 커넥션을 맺을 수 없음
- Non-blocking I/O
  - Linux - `epoll()`
  - BSD/macOS - `kqueue()`
  - Windows - IOCP
  - Node.js, Nginx에서 지원되는 기술
  - Java 진영에서는 Netty 프레임워크를 통해 NIO 구현
  - select/poll: O(N) 복잡도로 모든 파일 디스크립터 순회
  - epoll: O(1) 복잡도로 준비된 소켓 사용

### C10M 문제

커널 레벨 병목

- File Discriptor 기본 ulimit 초과 문제
- TCP 연결 추적 오버헤드
- 수백만 개 작은 버퍼 할당으로 인한 메모리 파편화

네트워크 스택 병목

- 초당 수백만 패킷에 대한 interrupt 처리 불가능

**Kernel Bypass**

- DPDK(Data Plane Development Kit)
  - user mode에서 직접 NIC(Network Interface Controller)와 통신
- kernel mode로의 시스템 콜을 줄일 수 있음
- 쉽게 말해서, Linux의 커널 네트워크 스택을 우회하는 과정

그밖의 접근 방식들

- Zero-copy
  - kernel-user 간 데이터 복사 최소화
  - DMA(Direct Memory Access)를 활용한 직접 전송
- 커넥션 풀링 및 샤딩

Reference

- [올리브영 테크블로그 - 고전 돌아보기, C10K 문제 (C10K Problem)](https://oliveyoung.tech/2023-10-02/c10-problem)
- [삵 - C10k 문제란 무엇이며 해결책은?](https://sarc.io/index.php/miscellaneous/2381-c10k)
- [EVEerNew - DPDK의 원리와 커널 우회(Kernel bypass)](https://everenew.tistory.com/488)

## 무중단 배포 전략

### Graceful Shutdown

1. 로드밸런서가 서버 상태를 Draining으로 변경
2. 새로운 연결 요청은 다른 서버로 라우팅
3. 기존 활성화 연결에 "서버 점검 예정" 메시지 전송
4. 타임아웃 이후 남아 있는 연결들 강제 종료
5. 클라이언트의 자동 재연결 로직 실행

### Connection Draining

- 한번에 모든 연결을 다른 서버로 이전하면 부하 급증
- 부하를 모니터링하면서 속도를 조절하는 방식
- Redis/Kafka 영속 큐로 메시지 무결성 보장

## 세션 관리 방법

### Sticky Session

- 사용자가 최초 연결한 서버에 계속 연결되도록 유지
- 장점
  - 구현이 단순
  - 세션 정보를 서버 메모리에 보관 가능
  - 서버 간 상태 동기화 불필요
- 단점
  - 활성 유저가 많은 서버에 과부하
  - 서버 장애 시 해당 서버의 모든 사용자 연결 끊김
  - 수평 확장 제약

### Connection Migration

- 연결과 세션 상태 분리
- 세션 정보는 Redis 등 외부 저장소에 보관
- 장점
  - 완벽한 부하 분산 가능
  - 무중단 배포 용이
- 단점
  - 외부 저장소 접근으로 인한 네트워크 latency 증가
  - 외부 저장소가 SPOF가 될 수 있음

### 하이브리드 접근 방식

- Regional Sticky Session
  - 지역 단위 서버 클러스터 구성
  - 클러스터 내에서는 Sticky Session 유지
- Hot/Warm State 분리
  - Hot state: 서버 메모리 (최근 메시지, 활성 유저)
  - Warm state: Redis (세션 정보, 읽음 상태)
  - Cold state: DB (전체 메시지 히스토리)
- 배포 시 Connection Migration 활성화

## 메시지 동기화 메커니즘

### Sync Cursor

- 각 디바이스가 어디까지 동기화했는지 추적하는 포인터
- `last_seen_message_id`: 마지막으로 본 메시지 ID
- `sync_cursor`: 서버와 동기화된 지점
- `channel_cursors`: 채널별 동기화 포인터
- 장점
  - 전체 메시지가 아닌 델타 부분만 전송
  - 멱등성 보장

### Resume

- 네트워크 끊김이나 앱 백그라운드 전환 시 재연결 처리 문제
- Sequence Number 혹은 Timestamp 방식을 활용해서 체크포인트 지정 (Snowflake도 활용 가능)
- 재연결 이후 메시지 resume

## 메시지 순서 보장

- 두 사용자가 동시에 메시지를 전송했을 때, 네트워크 지연 차이, 서버 처리 시간 차이, 시계 동기화 오차로 인해 순서가 뒤바뀔 수 있음

### Lamport Timestamp

- 각 프로세스는 로컬 카운터 유지
- 이벤트 발송 시 카운터를 증가시키며, 메시지 전송 시 현재 카운터를 포함해서 전송
- 메시지 수신 시: `local = max(local, received) + 1`
- 하지만 독립적인 이벤트 구분 및 동시성 판단 불가능

### Vector Clock

- 각 노드의 논리적 시계를 벡터로 관리
- 하지만 채팅 시스템에는 과도하게 복잡할 수 있음
  - 1만 명 그룹 = 1만 차원 벡터
  - 네트워크 오버헤드 및 저장 공간 낭비

### 하이브리드 접근 방식

**Server-assigned sequence + Client timestamp**

```text
메시지 구조:
{
  "id": "snowflake_id",          // 서버가 생성
  "server_seq": 12345,            // 채널별 sequence
  "client_ts": 1234567890123,     // 클라이언트 timestamp
  "server_ts": 1234567890125      // 서버 수신 시각
}

정렬 규칙:
1차: server_seq (채널 내 절대 순서)
2차: server_ts (동일 seq일 경우)
3차: id (tie-breaking)
```
