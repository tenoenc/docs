## 자료구조 기초 및 복잡도

### 데이터의 물리적 구조
#### 메모리 레이아웃
##### 연속 할당 (Contiguous)
##### 불연속 할당 (Non-contiguous)
#### 데이터 정렬 (Alignment)
##### 워드 단위 정렬 (Word Alignment)
##### 패딩 (Padding) 및 패킹 (Packing)

### 추상화와 설계
#### 자료형 (Data Type)
##### 기본 자료형 (Primitive)
##### 파생 자료형 (Derived)
#### 추상 데이터 타입 (ADT)
##### 인터페이스와 구현의 분리
##### 캡슐화와 정보 은닉

### 알고리즘 효율성 분석
#### 성능 측정 지표
##### 시간 복잡도 (Time Complexity)
##### 공간 복잡도 (Space Complexity)

#### 점근적 표기법 (Asymptotic Notation)
##### Big-O ($O$)
##### Big-Omega ($\Omega$)
##### Big-Theta ($\Theta$)
##### Little-o ($o$) 및 Little-omega ($\omega$)
#### 복잡도 분석 규칙
##### 합의 법칙 (Sum Rule)
##### 곱의 법칙 (Product Rule)
##### 지배적 항 (Dominant Term)

### 재귀와 복잡도
#### 재귀 알고리즘 (Recursion)
##### 베이스 케이스 (Base Case)
##### 재귀 단계 (Recursive Step)
#### 점화식 (Recurrence Relation)
##### 치환법 (Substitution Method)
##### 재귀 트리 (Recursive Tree)
##### 마스터 정리 (Master Theorem)

### 특수 분석 기법
#### 분할 상환 분석 (Amortized Analysis)
##### 집합 분석 (Aggregate Analysis)
##### 회계 방법 (Accounting Method)
##### 포텐셜 방법 (Potential Method)

### 복잡도 클래스 (Complexity Classes)
#### 결정론적 알고리즘 (P)
#### 비결정론적 알고리즘 (NP)
##### NP-Complete
##### NP-Hard

### 하드웨어와 성능 최적화
#### 메모리 계층 구조의 영향
##### 참조 지역성 (Locality of Reference)
##### 캐시 적중(Hit) 및 미스(Miss)
#### 실행 환경의 제약
##### 메모리 대역폭 (Bandwidth)
##### 지연 시간 (Latency)

## 선형 자료구조 (Linear)

### 배열 (Array)
#### 물리적 특성
##### 정적 배열 (Static Array)
##### 동적 배열 (Dynamic Array)
#### 배열의 확장
##### 다차원 배열 (Multi-dimensional)
##### 희소 배열 (Sparse Array)
#### 배열 연산의 효율성
##### 임의 접근 (Random Access)
##### 삽입 및 삭제 오버헤드

### 연결 리스트 (Linked List)
#### 노드 구조 (Node Structure)
##### 데이터 필드 (Data Field)
##### 링크 필드 (Link Field)
#### 리스트 유형
##### 단일 연결 리스트 (Singly Linked List)
##### 이중 연결 리스트 (Doubly Linked List)
##### 원형 연결 리스트 (Circular Linked List)

#### 리스트 연산
##### 검색 (Search)
##### 삽입 및 삭제 (Insertion & Deletion)
#### 성능 분석 및 비교
##### 메모리 오버헤드
##### 참조 지역성 분석

### 스택 (Stack)
#### 추상 모델
##### LIFO (Last-In-First-Out)
#### 구현 매커니즘
##### 배열 기반 스택
##### 연결 리스트 기반 스택
#### 스택 연산
##### Push 및 Pop
##### Peek 및 Top

#### 스택의 활용
##### 함수 호출과 복귀 (Call Stack)
##### 수식 계산 (Infix, Prefix, Postfix)
##### 구문 분석 (Parsing)
##### 실행 취소 (Undo / Redo)

### 큐 (Queue)
#### 추상 모델
##### FIFO (First-In-First-Out)
#### 구현 매커니즘
##### 선형 큐 (Linear Queue)
##### 원형 큐 (Circular Queue)
##### 연결 리스트 기반 큐

#### 큐 연산
##### Enqueue 및 Dequeue
##### Front 및 Rear 포인터
#### 덱 (Deque)
##### 양방향 삽입 및 삭제
##### 입력 제한 덱 (Scroll)
##### 출력 제한 덱 (Shelf)

#### 우선순위 큐 (Priority Queue: Linear)
##### 배열 기반 우선순위 큐
##### 연결 리스트 기반 우선순위 큐
#### 선형 자료구조의 실제 활용
##### 버퍼 관리 (Buffer Management)
##### 스케줄링 (Scheduling)

## 트리 (Trees)

### 트리 기초 및 용어
#### 계층 구조 정의
##### 노드(Node) 및 간선(Edge)
##### 루트(Root) 및 리프(Leaf)
#### 노드 관계 및 속성
##### 부모(Parent), 자식(Child), 형제(Sibling)
##### 조상(Ancestor) 및 후손(Descendant)
#### 측정 및 지표
##### 레벨(Level), 높이(Height), 깊이(Depth)
##### 차수(Degree)

### 이진 트리 (Binary Tree)
#### 형태에 따른 분류
##### 정 이진 트리 (Full)
##### 완전 이진 트리 (Complete)
##### 포화 이진 트리 (Perfect)
##### 편향 이진 트리 (Degenerate)
#### 구현 방식
##### 배열 기반 구현
##### 포인터 기반 구현

### 트리 순회 (Traversal)
#### 깊이 우선 탐색 (DFS)
##### 전위 순회 (Pre-order)
##### 중위 순회 (In-order)
##### 후위 순회 (Post-order)
#### 너비 우선 탐색 (BFS)
##### 레벨 순회 (Level-order)

### 이진 탐색 트리 (BST)
#### 기본 원리
##### 탐색 속성 (Key Property)
#### 기본 연산
##### 검색 (Search)
##### 삽입 (Insertion)
##### 삭제 (Deletion)
#### 한계점
##### 균형 붕괴 및 성능 저하

### 자가 균형 이진 탐색 트리 (Self-Balancing BST)
#### 트리 회전 (Rotation)
##### LL 회전 및 RR 회전
##### LR 회전 및 RL 회전
#### AVL 트리
##### 균형 인수 (Balance Factor)
##### 리밸런싱 메커니즘
#### 레드-블랙 트리 (Red-Black Tree)
##### 노드 색상 속성
##### 삽입 및 삭제 규칙 (Restructuring / Recoloring)

### 다원 탐색 트리 (M-way Search Tree)
#### B-트리 계열 (B-Tree Family)
##### B-트리
##### B+ 트리
##### B* 트리
#### 저장 장치 최적화
##### 디스크 I/O와 페이지 단위
##### 인덱싱과 탐색 효율

### 트리 기반 집합 관리
#### 분리 집합 (Disjoint Set)
##### 서로소 집합 자료구조
##### Union-Find 연산
##### 경로 압축 (Path Compression)
##### 랭크 기반 합치기 (Union by Rank)

#### 트리 자료구조의 실제 활용
##### 파일 시스템 계층 (Hierarchical FS)
##### 수식 트리 (Expression Tree)
##### 데이터베이스 인덱싱 (Indexing)

## 그래프 (Graphs)

### 그래프 기초 및 분류
#### 구성 요소
##### 정점 (Vertex) 및 간선 (Edge)
##### 가중치 (Weight)
#### 그래프 유형
##### 유향 그래프 (Directed)
##### 무향 그래프 (Undirected)
##### 가중치 그래프 (Weighted)
##### 순환 및 비순환 그래프 (Cyclic / Acyclic)

### 그래프 표현 (Representation)
#### 인접 행렬 (Adjacency Matrix)
##### 물리 구조 및 공간 복잡도
##### 간선 존재 확인 (Edge Lookup)
#### 인접 리스트 (Adjacency List)
##### 물리 구조 및 공간 복잡도
##### 정점의 차수 계산 (Degree Calculation)
#### 기타 표현 방식
##### 간선 리스트 (Edge List)
##### 근접 행렬 (Incidence Matrix)

### 그래프 탐색 (Traversal)
#### 깊이 우선 탐색 (DFS)
##### 스택 및 재귀 기반 구현
##### 타임스탬프 (Discovery / Finish Time)
#### 너비 우선 탐색 (BFS)
##### 큐 기반 구현
##### 최단 경로(가중치 없음)의 특성

### 신장 트리 (Spanning Tree)
#### 최소 신장 트리 (MST)
##### Kruskal 알고리즘
##### Prim 알고리즘
#### 사이클 및 연결성
##### 사이클 탐지 (Cycle Detection)
##### 강결합 요소 (SCC)

### 최단 경로 (Shortest Path)
#### 단일 출발 최단 경로
##### Dijkstra 알고리즘
##### Bellman-Ford 알고리즘
#### 모든 쌍 최단 경로
##### Floyd-Warshall 알고리즘

### 그래프 위상 및 특수 목적
#### 위상 정렬 (Topological Sort)
#### 네트워크 유량 (Network Flow)
##### 최대 유량 최소 컷 정리

## 해싱 (Hashing)

### 해시 테이블의 원리
#### 해시 함수 (Hash Function)
##### 결정론적 속성 (Determinism)
##### 효율성 및 균일성 (Uniformity)
#### 해싱 알고리즘
##### 나눗셈법 (Division Method)
##### 곱셈법 (Multiplication Method)
##### 유니버설 해싱 (Universal Hashing)

### 충돌 해결 전략 (Collision Resolution)
#### 개방 주소법 (Open Addressing)
##### 선형 조사법 (Linear Probing)
##### 이차 조사법 (Quadratic Probing)
##### 이중 해싱 (Double Hashing)

#### 분리 연결법 (Separate Chaining)
##### 연결 리스트 기반 체이닝
##### 자가 균형 트리 기반 체이닝
#### 해시 테이블 관리
##### 적재율 (Load Factor)
##### 재해싱 (Rehashing) 및 동적 확장
##### 삭제 처리와 더미 노드 (Ghost Entry)

### 고급 해싱 기법
#### 완벽 해싱 (Perfect Hashing)
##### 정적 키 집합 최적화
##### 2단계 해싱 구조
#### 일관된 해싱 (Consistent Hashing)
##### 분산 시스템의 데이터 매핑
##### 노드 추가 및 삭제 시 재배치 최소화
#### 암호학적 해싱 (Cryptographic Hash)
##### 일방향성 및 충돌 저항성
##### 블록체인 및 메시지 인증 (MAC)

### 해싱의 실제 활용
#### 검색 및 저장 최적화
##### 데이터베이스 인덱싱
##### 심볼 테이블 (Symbol Table)
#### 무결성 및 보안
##### 체크섬 (Checksum)
##### 비밀번호 저장 및 솔팅 (Salting)

## 힙과 우선순위 큐 (Heaps)

### 힙의 개념 및 성질
#### 기본 정의
##### 완전 이진 트리 구조
##### 부모-자식 간 대소 관계 (Heap Property)
#### 물리적 구현
##### 배열 기반 표현
##### 인덱스 산출 공식

### 이진 힙 (Binary Heap)
#### 유형 분류
##### 최대 힙 (Max Heap)
##### 최소 힙 (Min Heap)
#### 힙 기본 연산
##### 삽입 (Up-Heap / Percolate Up)
##### 삭제 (Down-Heap / Percolate Down)
##### 힙 생성 (Heapify)

### 우선순위 큐 (Priority Queue: Heap-based)
#### 연산 효율성
##### 삽입 및 추출 복잡도
##### 최우선 순위 확인 ($O(1)$)
#### 구현 고려사항
##### 가변 우선순위 (Decrease Key)
##### 임의 원소 삭제 및 갱신

### 고급 힙 구조 (Advanced Heaps)
#### 이항 힙 (Binomial Heap)
##### 이항 트리 구조
##### 힙 병합 (Merge) 연산 최적화
#### 피보나치 힙 (Fibonacci Heap)
##### 지연 통합 (Lazy Consolidation)
##### 분할 상환 복잡도 최적화

#### 힙의 실제 활용
##### 힙 정렬 (Heap Sort)
##### K-최대/최소 원소 추출
##### 중앙값 추적 (Median Tracking)
##### 실시간 스케줄링 및 버퍼 관리

## 문자열 자료구조 (Strings)

### 문자열 처리 기초
#### 메모리 표현 방식
##### Null-terminated String
##### Length-prefixed String (Pascal)
#### 문자열 불변성 (Immutability)
##### 스트링 풀 (String Pool)
##### 복사 시점 복사 (Copy-on-Write)

### 트라이 (Trie)
#### 기본 원리
##### 접두사 트리 (Prefix Tree)
##### 노드 및 포인터 구조
#### 트라이 연산
##### 삽입 및 검색 복잡도
##### 접두사 검색 (Prefix Search)
#### 트라이의 변형
##### 압축 트라이 (Compressed Trie)
##### 테르나리 탐색 트리 (TST)

### 접미사 구조 (Suffix Structures)
#### 접미사 트리 (Suffix Tree)
##### 문자열 패턴 매칭
##### 중복 문자열 탐색

#### 접미사 배열 (Suffix Array)
##### 인덱스 배열 구성
##### LCP 배열 (Longest Common Prefix)

### 문자열 매칭 알고리즘
#### 소박한 매칭 (Naive Matching)
#### KMP 알고리즘
##### 실패 함수 (Failure Function)
#### 보이어-무어 (Boyer-Moore)
##### 나쁜 문자 규칙 (Bad Character)
##### 착한 접미사 규칙 (Good Suffix)
#### 라빈-카프 (Rabin-Karp)
##### 롤링 해시 (Rolling Hash)

## 고급 및 특수 자료구조 (Advanced)

### 구간 쿼리 자료구조 (Range Query)
#### 세그먼트 트리 (Segment Tree)
##### 구간 합 및 최소/최대 최적화
##### 느린 갱신 (Lazy Propagation)
#### 펜윅 트리 (Fenwick Tree / BIT)
##### 누적 합 (Prefix Sum) 관리
##### 비트 연산을 활용한 효율화

### 다차원 및 공간 자료구조 (Spatial)
#### 쿼드 트리 (Quad Tree) 및 옥트 트리 (Octree)
##### 2D/3D 공간 분할
##### 충돌 감지 및 가시성 검사
#### K-D 트리 (K-Dimensional Tree)
##### 다차원 포인트 탐색
##### 최근접 이웃 검색 (Nearest Neighbor)

### 확률적 자료구조 (Probabilistic)
#### 블룸 필터 (Bloom Filter)
##### 집합 포함 여부 검사
##### 위양성(False Positive) 확률
#### 카운트-미 스케치 (Count-Min Sketch)
##### 빈도수 추정 및 오차 범위
#### 하이퍼로그로그 (HyperLogLog)
##### 카디널리티(Cardinality) 추정

### 비트 기반 자료구조
#### 비트셋 (Bitset)
##### 공간 효율적 플래그 관리
##### 비트 마스킹 연산
#### 비트 조작 (Bit Manipulation)
##### 비트 카운팅 (Popcount)
##### 최하위 비트 (LSB) 연산

### 고급 연결 구조
#### 스킵 리스트 (Skip List)
##### 다중 레벨 인덱싱
##### 탐색 및 삽입의 확률적 보장
#### 자기 구성 리스트 (Self-organizing List)
##### 참조 국부성 활용
##### Move-to-Front 전략

## 실무 설계 및 최적화 (Practical)

### 언어별 표준 라이브러리 (Standard Libraries)
#### C++ STL (Standard Template Library)
##### 컨테이너(Sequence, Associative, Unordered)
##### 반복자(Iterator)와 알고리즘의 결합
#### Java Collections Framework
##### 인터페이스 기반 설계 (List, Set, Map)
##### 원시 타입과 래퍼 클래스의 성능 차이
#### Python 자료구조의 내부
##### 리스트(Dynamic Array)와 딕셔너리(Hash Table)의 특징
##### 시간 복잡도의 실제 오버헤드

### 하드웨어 친화적 최적화 (System-Level)
#### 캐시 지역성 (Cache Locality)
##### 연속 메모리 배치(Array) vs 포인터 체이닝(Linked List)
##### 공간적 및 시간적 지역성 활용
#### 메모리 관리
##### 단편화(Fragmentation)와 할당 비용
##### 풀링(Pooling) 및 재사용 전략
##### 가비지 컬렉션(GC)의 영향과 자료구조 크기

### 상황별 자료구조 선택 (Decision Making)
#### 트레이드오프 분석
##### 시간 복잡도($O(n)$) vs 공간 복잡도($O(s)$)
##### 읽기 집약 vs 쓰기 집약 (Read/Write Ratio)
#### 데이터 특성 기반 선택
##### 데이터 규모에 따른 확장성(Scalability)
##### 데이터의 수명(Lifetime) 고려

### 동시성 및 멀티스레딩 (Concurrency)
#### 스레드 안전성 (Thread-safety)
##### 뮤텍스(Mutex) 및 세마포어(Semaphore) 기반 동기화
#### 고성능 동시성 자료구조
##### 읽기-쓰기 잠금 (Read-Write Lock)
##### 락-프리(Lock-free) 및 대기-프리(Wait-free) 구조