## NoSQL 개념 및 이론

### 데이터 패러다임 전환
#### 정의 및 명칭 (Not Only SQL)
#### RDBMS의 한계
##### 확장성 제약 (Scalability)
##### 데이터 구조의 경직성
#### NoSQL의 핵심 가치
##### 수평적 확장성 (Scale-out)
##### 유연한 데이터 모델링
##### 고성능 읽기/쓰기

### 데이터 모델링 기초
#### 스키마 유연성
##### 강한 스키마 (Rigid)
##### 스키마리스 (Schemaless)
#### 데이터 관계 처리
##### 비정규화 (Denormalization)
##### 참조 vs 내장 (Embedding)
##### 조인(Join)의 부재와 해결

### 분산 시스템 정리 (Theorems)
#### CAP 정리
##### 일관성 (Consistency)
##### 가용성 (Availability)
##### 분할 내성 (Partition Tolerance)
#### CAP 조합 및 선택
##### CP 모델 (Consistency)
##### AP 모델 (Availability)
##### CA 모델의 제약
#### PACELC 정리
##### 장애 상황 (P-A-C)
##### 정상 상황 (E-L-C)

### 일관성 모델 기초
#### 강한 일관성 (Strong)
#### 완화된 일관성 (Weak)
#### 최종 일관성 (Eventual)

### 데이터 상태 및 전이 이론
#### BASE 특성
##### 기본적 가용성 (BA)
##### 소프트 상태 (S)
##### 최종 일관성 (E)
#### 상태 전이 모델
##### 멱등성 (Idempotency)
##### 교환 법칙 (Commutativity)

### NoSQL 트랜잭션 모델
#### ACID와의 철학적 차이
#### 원자성 범위 (Scope)
##### 단일 문서 원자성
##### 다중 문서 트랜잭션 제약
#### 분산 트랜잭션 이론
##### 2단계 커밋 (2PC)의 한계
##### TCC (Try-Confirm-Cancel) 개념

### 성능 및 평가 지표
#### 시스템 성능 지표
##### 처리량 (Throughput)
##### 지연 시간 (Latency)
#### 워크로드 특성
##### 읽기 집약 (Read-intensive)
##### 쓰기 집약 (Write-intensive)

### 선정 및 비교 방법론
#### 데이터 적합성 평가
##### 데이터 정형성 (Structuredness)
##### 관계 복잡도 (Relationship)
#### 비즈니스 요구사항 분석
##### 데이터 무결성 중요도
##### 동시성 제어 수준
#### 기술 선택 전략
##### 폴리글랏 퍼시스턴스 (Polyglot)
##### 총 소유 비용 (TCO)

## 데이터 모델 및 저장 분류

### Key-Value 모델
#### 기본 아키텍처
##### 단순 매핑 구조
##### O(1) 접근성
#### 데이터 불투명성
##### 값(Value)의 비해석
##### 블롭(Blob) 저장장치
#### 핵심 활용 패턴
##### 세션 관리
##### 분산 캐싱
##### 단순 환경설정

### Document 모델
#### 데이터 표현 방식
##### JSON / BSON 구조
##### 계층적 데이터 트리
#### 스키마 설계 특징
##### 필드 동적 확장
##### 데이터 내장 (Embedding)
##### 참조 모델 (Referencing)

### Wide-Column 모델
#### 물리적 구조
##### 행 키 (Row Key)
##### 컬럼 패밀리 (Column Family)
##### 컬럼 한정자 (Qualifier)
##### 타임스탬프 (Timestamp)
#### 데이터 저장 특성
##### 희소 행렬 (Sparsity)
##### 다차원 맵 (Multidimensional)
##### 쓰기 효율성 (LSM 기반)

### Graph 모델
#### 요소 구성
##### 노드 (Node / Vertex)
##### 관계 (Relationship / Edge)
##### 속성 (Property)
#### 탐색 매커니즘
##### 인덱스 없는 인접성 (Index-free Adjacency)
##### 그래프 순회 (Traversal)
##### 가변 길이 경로 검색

### 특수 목적 저장소
#### 시계열 (Time-Series) 모델
##### 시간 축 인덱싱
##### 데이터 보존 정책 (Retention)
##### 데이터 다운샘플링 (Downsampling)
#### 검색 엔진 (Search Engine) 모델
##### 역색인 구조 (Inverted Index)
##### 토큰화 및 분석 (Tokenization)
##### 랭킹 및 스코어링 (Scoring)

### 하이브리드 및 멀티 모델
#### 멀티 모델 아키텍처
##### 다중 인터페이스 지원
##### 통합 저장 엔진

## 분산 시스템 구조

### 노드 구성 모델
#### 마스터-슬레이브 구조 (Master-Slave)
##### 단일 마스터 제약
##### 읽기 전용 복제본 (Read Replica)
#### 피어 투 피어 구조 (Peer-to-Peer)
##### 비중앙집권형 통신
##### 대칭형 노드 역할
#### 하이브리드 토폴로지
##### 리더 선출 모델

### 멤버십 및 상태 관리
#### 가십 프로토콜 (Gossip Protocol)
##### 전파 매커니즘 (Epidemic)
##### 수렴 속도 및 대역폭 제어
#### 장애 감지 (Failure Detection)
##### 하트비트 (Heartbeat)
##### 의심 및 확정 상태 (Suspicion)

### 요청 라우팅 및 조정
#### 코디네이터 (Coordinator)
##### 역할 및 책임
##### 토폴로지 인식 (Topology Awareness)
#### 라우팅 인터페이스
##### 클라이언트 측 라우팅
##### 프록시 기반 라우팅
##### 서버 측 전달 (Server-side Forwarding)

### 쿼럼 및 참여 제어
#### 쿼럼 모델 (Quorum)
##### 읽기/쓰기 쿼럼 구성 (R+W > N)
##### 정적 쿼럼 vs 동적 쿼럼
#### 노드 멤버십 관리
##### 동적 노드 가입/탈퇴
##### 시드 노드 (Seed Nodes) 역할

## 데이터 일관성 제어

### 분산 합의 (Consensus)
#### Paxos 프로토콜
##### 역할 군 (Proposer, Acceptor, Learner)
##### 단계별 합의 매커니즘
#### Raft 프로토콜
##### 리더 선출 (Leader Election)
##### 로그 복제 (Log Replication)
##### 멤버십 변경 관리

### 시간 및 순서 보장
#### 물리적 시간의 한계
##### 클럭 드리프트 (Clock Drift)
##### 네트워크 지연 변동성
#### 논리적 시간 (Logical Time)
##### 램포트 시계 (Lamport Clock)
##### 벡터 시계 (Vector Clock)

### 데이터 충돌 해결 (Conflict Resolution)
#### 비결정적 해결
##### LWW (Last Write Wins)
#### 결정적 해결
##### 버전 벡터 (Version Vectors)
##### CRDT (Conflict-free Replicated Data Types)
#### 수동 해결
##### 병합 관리 (Application-side Merge)

### 세부 일관성 보장 모델
#### 클라이언트 중심 일관성
##### 자기 쓰기 읽기 (Read-after-write)
##### 단조적 읽기 (Monotonic Reads)
##### 단조적 쓰기 (Monotonic Writes)
#### 인과적 일관성 (Causal Consistency)
##### 선후 관계 의존성

## 저장 엔진 및 물리 구조

### 로그 기반 저장 매커니즘
#### WAL (Write-Ahead Log)
##### 순차 쓰기 (Sequential I/O)
##### 원자적 기록 보장
#### LSM-Tree (Log-Structured Merge-Tree)
##### 멤테이블 (Memtable)
##### SSTable (Sorted String Table)
##### 불변성 (Immutability) 특성

### 저장 최적화 기술
#### 컴팩션 (Compaction)
##### 레벨드 컴팩션 (Leveled)
##### 사이즈 티어드 컴팩션 (Size-tiered)
#### 블룸 필터 (Bloom Filter)
##### 확률적 멤버십 확인
##### 디스크 I/O 감소 효과

### 인덱스 구조 (Index Structure)
#### B-Tree 기반 엔진
##### 페이지 관리
##### 리프 노드 연결
#### 메모리 기반 엔진
##### 해시 인덱스 (Hash Index)
##### 인메모리 스냅샷

### 물리적 데이터 배치
#### 행 중심 저장 (Row-oriented)
##### OLTP 워크로드 최적화
#### 열 중심 저장 (Column-oriented)
##### 분석 쿼리 (OLAP) 성능
##### 컬럼별 압축 효율

### 데이터 직렬화 및 압축
#### 데이터 직렬화 (Serialization)
##### 스키마 기반 포맷 (Avro, Protobuf)
##### 스키마리스 포맷 (JSON, MessagePack)
#### 데이터 압축 (Compression)
##### 블록 수준 압축
##### 인코딩 기법 (Dictionary, Delta)

## 쿼리 및 인덱싱

### 데이터 접근 패턴
#### 키 기반 조회 (Key-value Lookup)
#### 범위 스캔 (Range Scan)
#### 전체 스캔 (Full Scan)

### 보조 인덱스 (Secondary Index)
#### 인덱스 분포 모델
##### 로컬 인덱스 (Local Index)
##### 전역 인덱스 (Global Index)
#### 갱신 매커니즘
##### 동기식 인쇄 (Synchronous)
##### 비동기식 인쇄 (Asynchronous)

### 특수 목적 인덱싱
#### 역색인 (Inverted Index)
##### 텍스트 토큰화
##### 전문 검색 (Full-text Search)
#### 공간 인덱스 (Geospatial Index)
##### Geohash
##### 쿼드 트리 (Quadtree)
#### 다차원 인덱스
##### 복합 인덱스 (Compound)

### 고급 쿼리 수행 모델
#### 분산 처리 엔진
##### 맵리듀스 (MapReduce)
##### 파이프라인 집계 (Aggregation Pipeline)
#### 분산 조인 (Distributed Join)
##### 클라이언트 조인
##### 비정규화 조인
##### 데이터 복제 조인

## 확장성 및 샤딩

### 수평적 확장성 (Horizontal Scaling)
#### 스케일 아웃 (Scale-out) 원리
#### 데이터 파티셔닝 (Partitioning)
##### 수직 파티셔닝
##### 수평 파티셔닝 (Sharding)

### 데이터 분산 알고리즘
#### 해시 기반 분산 (Hash-based)
##### 모듈로 함수 (Modulo)
##### 일관된 해싱 (Consistent Hashing)
##### 가상 노드 (Virtual Nodes)
#### 범위 기반 분산 (Range-based)
##### 키 공간 분할 (Key Space)

### 샤딩 키 전략
#### 키 선정 기준
##### 카디널리티 (Cardinality)
##### 부하 분산도
#### 핫스팟 (Hotspot) 제어
##### 읽기/쓰기 편향
##### 솔팅 (Salting) 기법

### 데이터 재분산 (Rebalancing)
#### 재분산 트리거
#### 수행 매커니즘
##### 파티션 분할 (Split)
##### 파티션 병합 (Merge)
##### 데이터 마이그레이션

## 복제 및 가용성

### 복제 동기화 모델
#### 데이터 전송 방식
##### 동기 복제 (Synchronous)
##### 비동기 복제 (Asynchronous)
##### 반동기 복제 (Semi-synchronous)
#### 복제 지연 (Replication Lag)
##### 지연 발생 원인
##### 읽기 일관성 영향

### 복제 토폴로지
#### 리더 기반 복제
##### 단일 리더 (Single-leader)
##### 다중 리더 (Multi-leader)
#### 무리더 복제 (Leaderless)
##### 쓰기 정족수 (Write Quorum)
##### 읽기 복구 (Read Repair)

### 가용성 유지 매커니즘
#### 장애 감지 (Failure Detection)
##### 하트비트 (Heartbeat) 체크
##### 시간 제한 (Timeout) 임계치
#### 장애 조치 (Failover)
##### 리더 승출 (Leader Election)
##### 데이터 정합성 확인
#### 고가용성 제약
##### 스플릿 브레인 (Split-brain)
##### 펜싱 (Fencing) 매커니즘

## 운영 및 성능 관리

### 성능 모니터링
#### 핵심 성능 지표 (KPI)
##### 처리량 (Throughput)
##### 응답 지연 (Latency)
##### 오류율 (Error Rate)
#### 자원 사용량
##### CPU 및 I/O 대기 (Wait)
##### 메모리 사용량 (Resident Set)
##### 디스크 사용량 및 증가율

### 백업 및 데이터 보호
#### 백업 전략
##### 전체 백업 (Full)
##### 증분 백업 (Incremental)
#### 백업 방식
##### 논리적 백업 (Dump)
##### 물리적 백업 (Snapshot)
#### 복구 목표
##### 복구 지점 목표 (RPO)
##### 복구 시간 목표 (RTO)

### 유지보수 및 최적화
#### 컴팩션 제어 (Compaction)
##### 컴팩션 스케줄링
##### 스로틀링 (Throttling)
#### 가비지 컬렉션 (GC)
##### 메모리 힙 관리
##### 툼스톤(Tombstone) 정리
#### 성능 튜닝
##### 커널 매개변수 최적화
##### 쿼리 프로파일링 (Profiling)

### 보안 관리
#### 인증 및 인가
##### 사용자 인증 (Authentication)
##### 역할 기반 접근 제어 (RBAC)
#### 데이터 암호화
##### 저장 데이터 암호화 (At-rest)
##### 전송 데이터 암호화 (In-transit)
#### 감사 (Auditing)
##### 작업 로그 기록
##### 규정 준수 (Compliance)

## 아키텍처 설계 패턴

### 데이터 접근 패턴
#### 캐시 최적화
##### Read-through / Write-through
##### Cache Aside
#### 다중 저장소 운용
##### 폴리글랏 퍼시스턴스 (Polyglot)
##### CQRS 패턴 (명령/조회 분리)

### 데이터 모델링 패턴
#### 비정규화 전략
##### 데이터 중복 (Redundancy)
##### 사전 조인 (Pre-join)
#### 애그리거트(Aggregate) 모델링
##### 데이터 경계 정의
##### 루트 엔티티 설계

### 실시간 데이터 흐름
#### 변경 데이터 캡처 (CDC)
##### 로그 기반 캡처
##### 이벤트 스트리밍 연동
#### 이벤트 소싱 (Event Sourcing)
##### 이벤트 로그 영속화
##### 상태 재생 (Replay) 및 스냅샷

### 시스템 통합 및 마이그레이션
#### 데이터 인터페이스
##### 저장소 캡슐화 (Repository)
##### 데이터 접근 계층 (DAO)
#### 마이그레이션 전략
##### 무중단 마이그레이션 (Zero-downtime)
##### 이중 쓰기 (Dual Write)
##### 단계적 전환 (Canary/Blue-green)

### 안티 패턴 (Anti-patterns)
#### 모델링 오류
##### RDBMS 방식의 직접 이식
##### 과도한 비정규화 및 비대칭
#### 운영 오류
##### 핫스팟 키 설계
##### 무분별한 스캔 쿼리