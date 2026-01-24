## 알고리즘 설계 및 분석 기초 (Foundations)

### 알고리즘의 본질적 속성
#### 성립 요건 및 제약
##### 입력과 출력의 인터페이스
##### 유한성과 정지 문제
##### 수행 가능성(Effectiveness)
#### 품질 평가 기준
##### 논리적 정확성(Correctness)
##### 자원 효율성(Efficiency)
##### 명확성(Definiteness)

### 알고리즘 분석 체계
#### 정적 분석 (Static Analysis)
##### 코드 복잡도 분석
##### 최악/최선/평균 케이스
#### 동적 분석 (Dynamic Analysis)
##### 프로파일링(Profiling)
##### 실제 실행 시간 측정

### 점근적 복잡도 분석 (Asymptotic Analysis)
#### 점근적 표기법의 수학적 정의
##### Big-O ($O$): 점근적 상한
##### Big-Omega ($\Omega$): 점근적 하한
##### Big-Theta ($\Theta$): 점근적 상하한
#### 복잡도 계층 구조
##### 다항 시간(Polynomial) 클래스
##### 지수 및 계승(Exponential / Factorial)
##### 로그 및 선형 로그(Log-linear)

### 재귀 및 순환 관계 분석 (Recurrence Analysis)
#### 재귀 알고리즘의 논리 구조
##### 분할 및 병합 비용 산출
##### 기저 조건의 완전성
#### 순환 관계식(Recurrence) 해법
##### 반복 대입법 (Iteration)
##### 치환법 (Substitution)
##### 재귀 트리 모델링 (Recursion Tree)
##### 마스터 정리 (Master Theorem)

### 특수 분석 및 증명 기법
#### 분할 상환 분석 (Amortized)
##### 집약 분석 (Aggregate)
##### 회계 방법 (Accounting)
##### 포텐셜 방법 (Potential)
#### 정당성 증명 기법
##### 수학적 귀납법 (Induction)
##### 루프 불변성 (Loop Invariant)
##### 모순 증명법 (Contradiction)

### 복잡도 클래스와 계산 이론 기초
#### 결정 문제와 복잡도 분류
##### P 클래스: 다항 시간 해법
##### NP 클래스: 비결정적 다항 시간
##### NP-완비(Complete) 및 NP-난해(Hard)
#### 계산 이론적 한계
##### 환원성(Reducibility)의 개념
##### 정지 문제(Halting Problem)의 불가능성
##### 근사 가능성(Approximability)에 따른 분류

## 정렬 및 탐색 (Sorting & Searching)

### 비교 기반 정렬 (Comparison Sort)
#### 기초적 정렬 알고리즘 ($O(n^2)$)
##### 버블 정렬 (Bubble Sort)
##### 선택 정렬 (Selection Sort)
##### 삽입 정렬 (Insertion Sort)
#### 효율적 정렬 알고리즘 ($O(n \log n)$)
##### 합병 정렬 (Merge Sort)
##### 퀵 정렬 (Quick Sort)
##### 힙 정렬 (Heap Sort)

### 비비교 기반 정렬 (Non-comparison Sort)
#### 선형 시간 정렬 ($O(n)$)
##### 계수 정렬 (Counting Sort)
##### 기수 정렬 (Radix Sort)
##### 버킷 정렬 (Bucket Sort)

### 정렬의 특성 및 실무 최적화
#### 정렬 상태의 속성
##### 안정 정렬(Stable)의 논리 구조
##### 제자리 정렬(In-place)의 메모리 제약
#### 하이브리드 및 실전 알고리즘
##### 팀 정렬 (Timsort)
##### 인트로 정렬 (Introsort)
##### 블록 정렬 (Block Sort)

### 기초 탐색 알고리즘 (Basic Searching)
#### 정렬 상태에 따른 탐색
##### 순차 탐색 (Linear Search)
##### 이진 탐색 (Binary Search)
#### 수치 예측 기반 탐색
##### 보간 탐색 (Interpolation Search)
##### 지수 탐색 (Exponential Search)

### 선택 및 순위 통계 (Selection & Order Statistics)
#### 순위 통계량 탐색
##### 최솟값 및 최댓값 동시 탐색
##### 무작위 선택 알고리즘 (Quickselect)
##### 최악의 경우 선형 시간 선택 (Median of Medians)
#### K-번째 원소 추출 최적화
##### 힙 기반 선택 기법
##### 부분 정렬을 이용한 최적화

### 이진 탐색의 확장 및 응용
#### 경계값 및 범위 탐색
##### 하한(Lower Bound) 및 상한(Upper Bound)
##### 중복 원소의 구간 산출
#### 매개 변수 탐색 (Parametric Search)
##### 최적화 문제의 결정 문제 전환
##### 해의 고립 및 수렴 조건

### 외부 정렬 및 대규모 데이터 처리 (External Sorting)
#### 메모리 제약 하의 정렬 매커니즘
##### 외부 합병 정렬 (External Merge Sort)
##### 런(Run) 생성 및 최적 병합 순서
#### 병합 효율 최적화 기법
##### 다방향 합병 (Multi-way Merging)
##### 선택 트리 (Selection Tree) 활용
##### 대체 선택 (Replacement Selection)

### 탐색 성능 고도화 전략
#### 구조 기반 탐색 로직
##### 인덱스 기반 탐색 (Index-based Search)
##### 역인덱스 (Inverted Index) 구성 원리
#### 하드웨어 최적화 탐색
##### 병렬 탐색 (Parallel Search)
##### 캐시 인지 탐색 (Cache-conscious Search)

## 분할 정복 (Divide and Conquer)

### 분할 정복의 메커니즘
#### 3단계 설계 프로세스
##### 분할(Divide): 하위 문제로의 분해
##### 정복(Conquer): 재귀적 해결 및 기저 사례
##### 결합(Combine): 부분해의 통합
#### 설계 시 고려사항
##### 부분 문제의 독립성 보장
##### 제어 흐름의 오버헤드 산출
##### 데이터 분할의 균형성(Balance)

### 고전적 분할 정복 알고리즘
#### 리스트 기반 분할
##### 이진 탐색의 재귀적 관점
##### 퀵 정렬의 피벗 선택 및 분할
##### 합병 정렬의 안정적 결합

### 수치 계산 및 대수적 분할 정복
#### 효율적 대용량 연산
##### 카라추바 곱셈 (Karatsuba)
##### 고속 푸리에 변환 (FFT)
#### 행렬 연산의 최적화
##### 쉬트라센 알고리즘 (Strassen)

### 기하학적 분할 정복
#### 평면 및 공간 분할
##### 최근접 점의 쌍 (Closest Pair)
##### 볼록 껍질 (Convex Hull) 탐색

### 분할 정복의 최적화 및 구조적 제약
#### 재귀 제어 및 최적화
##### 꼬리 재귀 최적화 (Tail Call Optimization)
##### 비재귀적(Iterative) 구현으로의 전환
##### 하이브리드 접근: 소규모 구간의 삽입 정렬 전환
#### 패러다임의 한계
##### 중복 부분 문제(Overlapping Subproblems)의 발생
##### 스택 오버플로우와 메모리 오버헤드
##### 결합(Combine) 단계의 병목 현상

## 동적 계획법 (Dynamic Programming)

### 동적 계획법의 핵심 성질
#### 최적 부분 구조 (Optimal Substructure)
##### 부분 해의 최적성을 활용한 전체 해 구성
#### 중복되는 부분 문제 (Overlapping Subproblems)
##### 동일한 하위 문제의 반복 발생 및 결과 재사용

### 설계 패러다임 및 구현 기법
#### 탑다운(Top-down) 접근
##### 재귀 구조와 메모이제이션 (Memoization)
#### 바텀업(Bottom-up) 접근
##### 반복문 기반 타뷸레이션 (Tabulation)
##### 상태 전이 함수(State Transition) 정의

### 고전적 동적 계획법 유형
#### 자원 배분 및 선택 문제
##### 0/1 배낭 문제 (Knapsack)
##### 동전 거스름돈 및 조합 최적화
#### 문자열 및 서열 분석
##### 최장 공통 부분 수열 (LCS)
##### 최장 증가 부분 수열 (LIS)
##### 편집 거리 (Edit Distance)
#### 행렬 및 경로 최적화
##### 연쇄 행렬 곱셈 (Matrix Chain)
##### 격자판 경로 최적화

### 동적 계획법 최적화 전략
#### 공간 복잡도 저감 기법
##### 슬라이딩 윈도우 (Sliding Window DP)
##### 토글링 (Toggling) 변수 활용
#### 상태 정의 및 압축
##### 비트마스크를 이용한 상태 표현
##### 불필요한 차원 제거 (Dimension Reduction)

## 탐욕 알고리즘 (Greedy Method)

### 탐욕적 설계의 원칙
#### 핵심 성립 요건
##### 탐욕적 선택 속성 (Greedy Choice Property)
##### 최적 부분 구조 (Optimal Substructure)
#### 정당성 증명 전략
##### 탐욕적 선택의 선행성 증명
##### 교환 논법 (Exchange Argument)
##### 수학적 귀납법을 이용한 최적성 증명

### 고전적 탐욕 알고리즘
#### 자원 및 일정 관리
##### 활동 선택 문제 (Activity Selection)
##### 회의실 배정 및 스케줄링
#### 데이터 압축 및 부호화
##### 허프만 코딩 (Huffman Coding)
##### 접두사 코드 (Prefix Code) 최적화

### 탐욕 알고리즘의 분석 및 확장
#### DP와의 설계 비교
##### 의사결정 시점의 차이
##### 분할 가능 배낭(Fractional)과 0/1 배낭의 비교
#### 근사 알고리즘(Approximation)으로의 활용
##### 최적해와의 오차율(Approximation Ratio) 분석
##### NP-난해 문제에 대한 탐욕적 접근 전략

## 백트래킹 및 상태 공간 탐색 (Backtracking & State Space)

### 상태 공간 탐색의 기초
#### 상태 공간 트리 (State Space Tree)
##### 상태 정의 및 노드 전이
##### 해 공간(Solution Space)의 구조
#### 탐색 기법의 핵심
##### 제약 충족 문제 (CSP)
##### 탐색 트리와 재귀적 호출

### 백트래킹 메커니즘 (Backtracking)
#### 가지치기 (Pruning)
##### 유망성(Promising) 판단 알고리즘
##### 탐색 공간의 조기 차단
#### 상태 복구 전략
##### 깊이 우선 탐색(DFS)과의 관계
##### 노드 방문 및 상태 복원 (Backtrack)

### 백트래킹의 전형적 사례
#### 제약 충족 문제 (CSP)의 실제
##### N-Queens 문제
##### 그래프 색칠 문제 (Graph Coloring)
##### 스도쿠(Sudoku) 및 퍼즐 풀이
#### 경로 및 조합 탐색
##### 해밀턴 경로 (Hamiltonian Path)
##### 미로 찾기 (Maze Solving)
##### 부분 집합 합 문제 (Subset Sum)

### 분기 한정법 (Branch and Bound)
#### 설계 원리 및 백트래킹과의 차이
##### 최적화 문제로의 특화
##### 한계값(Bound)을 이용한 가지치기
#### 탐색 제어 전략
##### 너비 우선(BFS) 및 최우선(Best-first) 탐색
##### 우선순위 큐와 큐 기반 구현
#### 주요 적용 모델
##### 외판원 문제 (TSP)의 분기 한정 해법
##### 0/1 배낭 문제의 분기 한정 최적화

## 그래프 알고리즘 (Graph Algorithms)

### 그래프 탐색 및 순회 (Traversal)
#### 깊이 우선 탐색 (DFS)의 알고리즘적 응용
##### 타임스탬프와 정점 채색 (Discovery/Finish Time)
##### 간선의 분류 (Tree, Back, Forward, Cross Edge)
#### 너비 우선 탐색 (BFS)의 알고리즘적 응용
##### 가중치 없는 그래프의 최단 경로 성질
##### BFS 트리의 계층적 구조 분석

### 연결성 및 네트워크 구조 분석
#### 강결합 요소 (Strongly Connected Components)
##### 타잔 알고리즘 (Tarjan's)
##### 코사라주 알고리즘 (Kosaraju's)
#### 단절점 및 단절선 (Articulation Points & Bridges)
##### DFS 트리를 이용한 우회로 판별 로직

### 최소 신장 트리 (Minimum Spanning Tree)
#### MST의 성질과 탐욕적 선택
##### 안전 간선(Safe Edge)과 컷(Cut) 속성
#### Kruskal 알고리즘
##### 간선 중심 탐색 및 정렬
##### 서로소 집합(Disjoint Set)을 이용한 사이클 방지
#### Prim 알고리즘
##### 정점 중심 탐색 및 확장
##### 우선순위 큐 기반 최소 가중치 관리

### 단일 출발 최단 경로 (Single Source Shortest Path)
#### Dijkstra 알고리즘
##### 탐욕적 최단 거리 확정 기법
##### 우선순위 큐를 이용한 최적화
##### 음수 가중치 환경에서의 제약
#### Bellman-Ford 알고리즘
##### 간선 완화(Relaxation)의 반복 수행
##### 음수 사이클(Negative Cycle) 판별 로직

### 모든 쌍 최단 경로 (All-Pairs Shortest Path)
#### Floyd-Warshall 알고리즘
##### 동적 계획법 기반 경로 갱신
##### 경유지 중심의 상태 전이
#### Johnson 알고리즘
##### Reweighting을 통한 음수 가중치 보정
##### 희소 그래프 최적화

### 위상 정렬 (Topological Sort)
#### 유향 비순환 그래프(DAG)의 선형 순서
##### 진입 차수(Indegree) 기반 결정 로직
#### 위상 정렬 구현 기법
##### Kahn 알고리즘 (Queue 기반)
##### DFS 기반 순서 스택 활용

### 네트워크 유량 (Network Flow)
#### 유량 시스템의 기초 원리
##### 용량(Capacity) 및 유량(Flow)의 보존 법칙
##### 잔여 네트워크(Residual Network)와 역간선
#### 최대 유량 알고리즘
##### 포드-풀커슨(Ford-Fulkerson) 방법론
##### 에드몬드-카프(Edmonds-Karp) 알고리즘
##### 디닉(Dinic) 알고리즘과 레벨 그래프
#### 유량 문제의 변형 및 응용
##### 최대 유량 최소 컷 정리 (Max-flow Min-cut)
##### 최소 비용 최대 유량 (MCMF)

### 이분 매칭 (Bipartite Matching)
#### 이분 그래프의 대응 관계
##### 최대 매칭(Max Matching)의 알고리즘적 정의
##### DFS 기반 이분 매칭 구현
#### 매칭 이론 및 확장
##### 쾨니그의 정리 (Kőnig's Theorem)
##### 홉크로프트-카프(Hopcroft-Karp) 알고리즘

## 문자열 및 패턴 매칭 (String & Pattern Matching)

### 문자열 매칭의 기초 및 소박한 접근
#### 매칭 문제의 정의
##### 패턴(Pattern)과 텍스트(Text)의 인터페이스
##### 완전 일치와 부분 일치 명세
#### 소박한 문자열 매칭 (Naive Matching)
##### 전수 조사 기반 비교 로직
##### 불필요한 중복 비교의 한계

### 효율적 패턴 매칭 알고리즘
#### KMP 알고리즘 (Knuth-Morris-Pratt)
##### 실패 함수(Failure Function)의 정의
##### 접두사와 접미사의 일치 정보를 이용한 도약
#### 보이어-무어 알고리즘 (Boyer-Moore)
##### 나쁜 문자(Bad Character) 이동 규칙
##### 착한 접미사(Good Suffix) 이동 규칙
##### 오른쪽에서 왼쪽으로의 비교 메커니즘

### 해시 및 다중 패턴 매칭
#### 라빈-카프 알고리즘 (Rabin-Karp)
##### 롤링 해시(Rolling Hash) 연산 모델
##### 해시 충돌(Collision) 검증 및 위양성 제어
#### 아호-코라식 알고리즘 (Aho-Corasick)
##### 트라이(Trie) 기반 유한 오토마타 구성
##### 실패 링크(Failure Link)의 동적 전이
##### 출력 링크(Output Link)를 통한 다중 패턴 탐색

### 접미사 구조 및 최적화 탐색
#### 접미사 배열 및 트리 구성 알고리즘
##### 접미사 배열의 효율적 구축 (Manber-Myers)
##### LCP(Longest Common Prefix) 배열 산출 로직
#### 문자열 압축 및 근사 매칭
##### 런 길이 부호화(RLE) 알고리즘
##### 편집 거리(Edit Distance) 기반 유사도 탐색

## 계산 기하학 및 수학적 알고리즘 (Geometry & Mathematics)

### 계산 기하학의 기초 (Computational Geometry)
#### 기하학적 원자 및 연산
##### 점(Point)과 벡터(Vector)의 표현
##### 외적(Cross Product)과 내적(Dot Product)의 응용
##### CCW(Counter Clockwise) 알고리즘
#### 선분 및 다각형 연산
##### 두 선분의 교차 여부 판별
##### 다각형의 내부/외부 판별 (Point in Polygon)
##### 다각형의 넓이 계산 (신발끈 공식)

### 볼록 껍질 및 근접 탐색
#### 볼록 껍질 (Convex Hull) 알고리즘
##### 그레이엄 스캔 (Graham Scan)
##### 자비스 행보 (Jarvis March)
#### 기하학적 최적화
##### 회전하는 캘리퍼스 (Rotating Calipers)
##### 평면 스윕 (Plane Sweep) 기법

### 정수론 및 수치 알고리즘 (Number Theory & Numerical)
#### 소수 및 약수 관련 알고리즘
##### 에라토스테네스의 체 (Sieve of Eratosthenes)
##### 소수 판별법 (Primality Test)
##### 밀러-라빈 소수 판별법 (Miller-Rabin)
#### 최대공약수 및 모듈러 연산
##### 유클리드 호제법 (Euclidean Algorithm)
##### 확장 유클리드 호제법 (Extended Euclidean)
##### 모듈러 거듭제곱 (Modular Exponentiation)
#### 행렬 및 선형 대수 알고리즘
##### 가우스 소거법 (Gaussian Elimination)
##### 행렬 거듭제곱을 이용한 점화식 최적화

## 확률적 및 근사 알고리즘 (Randomized & Approximation)

### 확률적 알고리즘 (Randomized)
#### 확률적 설계의 범주
##### 라스베이거스 알고리즘 (Las Vegas): 정확성 보장
##### 몬테카를로 알고리즘 (Monte Carlo): 수행 시간 보장
#### 무작위화 핵심 기법
##### 무작위 피벗 선택 및 퀵 정렬
##### 무작위 선택 알고리즘 (Quickselect)
##### 밀러-라빈의 확률적 소수 판별

### 근사 알고리즘 (Approximation)
#### 근사 해법의 성능 지표
##### 근사 비율(Approximation Ratio) 분석
##### 다항 시간 근사 스키마 (PTAS / FPTAS)
#### 주요 최적화 문제의 근사
##### 정점 커버(Vertex Cover)의 2-근사
##### 외판원 문제(TSP)의 삼각 부등식 근사
##### 집합 커버(Set Cover)의 탐욕적 근사

## 휴리스틱 및 메타휴리스틱 (Heuristics & Metaheuristics)

### 휴리스틱 탐색 (Heuristic Search)
#### 정보 이용 탐색 기법
##### 최적 우선 탐색 (Best-first Search)
##### A* 알고리즘: 실제 거리 $g(n)$와 추정 거리 $h(n)$의 결합
##### IDA* (Iterative Deepening A*): 메모리 효율적 A*
#### 휴리스틱 함수의 설계
##### 허용성(Admissibility)과 일관성(Consistency)
##### 맨해튼 거리 및 유클리드 거리의 활용

### 국소 탐색 및 전역 최적화 (Local Search)
#### 기본 국소 탐색
##### 언덕 오르기 (Hill Climbing)와 지역 최적점(Local Optima) 문제
#### 메타휴리스틱 전략
##### 담금질 기법 (Simulated Annealing): 확률적 하강과 냉각 스케줄링
##### 타부 탐색 (Tabu Search): 최근 경로 기억을 통한 루프 방지

### 자연 모방 알고리즘 (Nature-inspired)
#### 진화 연산 (Evolutionary Computing)
##### 유전 알고리즘 (Genetic Algorithm): 선택, 교차, 변이
#### 군집 지능 (Swarm Intelligence)
##### 입자 군집 최적화 (PSO)
##### 개미 집단 알고리즘 (ACO)

## 계산 복잡도 이론 및 NP-완전성 (Computational Complexity)

### 문제의 분류와 복잡도 클래스
#### 시간 및 공간 복잡도 계층
##### $DTIME$과 $NTIME$의 정규 모델
##### 공간 복잡도 클래스 ($PSPACE$, $L$, $NL$)
#### 핵심 복잡도 클래스
##### 클래스 $P$: 다항 시간 내 해결 가능한 문제
##### 클래스 $NP$: 다항 시간 내 검증 가능한 문제
##### 결정론적 vs 비결정론적 알고리즘의 차이

### 환원(Reduction)과 완전성
#### 다항 시간 환원 (Polynomial-time Reduction)
##### 문제 $A$를 문제 $B$로 변환하는 논리 구조
##### 난이도의 하한(Lower Bound) 증명
#### NP-완전 (NP-Complete)
##### 쿡-레빈 정리 (Cook-Levin Theorem): SAT 문제의 상징성
##### 주요 NP-완전 문제군 (3-SAT, Clique, TSP, Knapsack 등)
#### NP-난해 (NP-Hard)
##### NP 문제보다 '최소한 더 어려운' 문제의 정의

### 현대 알고리즘의 난제와 대응
#### P vs NP 문제
##### 현대 과학의 7대 난제와 그 함의
#### 어려운 문제에 대한 실전적 전략
##### 다항 시간 내의 근사 해법 (Approximation)
##### 특정 파라미터 고정 탐색 (FPT)
##### 휴리스틱과 확률적 모델의 결합