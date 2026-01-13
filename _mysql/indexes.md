## 아키텍처

### MySQL 엔진
#### 커넥션 핸들러
##### 스레드 모델
##### 스레드 풀
##### 연결 제한 관리
##### 클라이언트 인증
#### SQL 인터페이스
##### 쿼리 수신 기능
##### 결과 반환 프로토콜
#### SQL 파서
##### 렉시컬 분석
##### 구문 분석
##### 파서 트리 생성
##### 키워드 및 예약어 검증
#### SQL 옵티마이저
##### 비용 기반 최적화 모델
##### 통계 정보 참조
##### 쿼리 재작성
##### 실행 계획 후보군 산출
#### 쿼리 실행기
##### 핸들러 API 연동
##### 단계별 연산 제어

### 스토리지 엔진
#### 플러그인 아키텍처
##### 동적 라이브러리 로딩
##### 엔진 교체 메커니즘
#### 핸들러 API
##### 로우 기반 인터페이스
##### 인덱스 스캔 인터페이스
#### 스토리지 엔진 개별 특성
##### InnoDB
##### MyISAM
##### Memory
##### CSV

### 메모리 구조
#### 글로벌 메모리 영역
##### InnoDB 버퍼 풀
##### InnoDB 리두 로그 버퍼
##### 어댑티브 해시 인덱스
#### 로컬 메모리 영역
##### 정렬 버퍼
##### 조인 버퍼
##### 바이너리 로그 캐시
##### 네트워크 버퍼

### 스레딩 모델
#### 포어그라운드 스레드
##### 사용자 세션 관리
##### 스레드 캐시
#### 백그라운드 스레드
##### 마스터 스레드
##### 입출력 스레드
##### 퍼지 스레드
##### 페이지 클리너 스레드
##### 로그 쓰기 스레드

### 파일 시스템 레이아웃
#### 설정 파일
#### 로그 파일
##### 에러 로그
##### 일반 쿼리 로그
##### 슬로우 쿼리 로그
##### 바이너리 로그
##### 리두 로그
##### 언두 로그
#### 데이터 파일
##### 시스템 테이블스페이스
##### 개별 테이블스페이스
##### 임시 테이블스페이스
##### 언두 테이블스페이스

## 트랜잭션과 잠금

### 트랜잭션
#### ACID 특성
##### 원자성 (Atomicity)
##### 일관성 (Consistency)
##### 격리성 (Isolation)
##### 지속성 (Durability)
#### 트랜잭션 생명주기
##### 시작 및 커밋
##### 롤백 메커니즘
##### 세이브포인트 (Savepoint)

### 트랜잭션 격리 수준
#### READ UNCOMMITTED
##### Dirty Read 현상
#### READ COMMITTED
##### Non-Repeatable Read 현상
#### REPEATABLE READ
##### Phantom Read 현상
##### MVCC 연동 원리
#### SERIALIZABLE
##### 잠금 기반 동시성 제한

### 잠금 메커니즘
#### 잠금 범위 분류
##### 글로벌 락
##### 테이블 락
##### 네임드 락
##### 메타데이터 락
#### 잠금 모드 분류
##### 공유 잠금 (Shared Lock)
##### 배타 잠금 (Exclusive Lock)
##### 의도 잠금 (Intention Lock)

### InnoDB 잠금 세부 구현
#### 레코드 잠금 (Record Lock)
##### 프라이머리 키 기반 잠금
##### 세컨더리 인덱스 기반 잠금
#### 갭 잠금 (Gap Lock)
##### 인덱스 레코드 사이 공간 제어
#### 넥스트 키 잠금 (Next-Key Lock)
##### 레코드 잠금과 갭 잠금의 결합
#### 자동 증가 잠금 (Auto-Increment Lock)
##### AUTO_INCREMENT 속성 처리
##### innodb_autoinc_lock_mode 설정

### 동시성 제어 (MVCC)
#### 언두 로그 (Undo Log)
##### 데이터 버전 관리
##### 이전 버전 레코드 탐색
#### 비잠금 일관된 읽기 (Non-locking Consistent Read)
##### Read View 생성 시점
##### 일관된 스냅샷 조회

### 데드락 (Deadlock)
#### 발생 기전
##### 상호 배제 조건
##### 점유 및 대기 조건
##### 비선점 조건
##### 순환 대기 조건
#### 데드락 탐지
##### Wait-for Graph 알고리즘
##### 탐지 주기 설정
#### 데드락 해결
##### 희생자(Victim) 선정 알고리즘
##### 롤백 비용 계산

### 잠금 성능 모니터링
#### 시스템 변수 추적
##### innodb_lock_wait_timeout
##### innodb_deadlock_detect
#### 정보 스키마 조회
##### INNODB_TRX
##### INNODB_LOCKS
##### INNODB_LOCK_WAITS
#### 성능 스키마 활용
##### data_locks 테이블
##### data_lock_waits 테이블

## 데이터 압축

### 페이지 압축 (Transparent Page Compression)
#### 펀치 홀 (Punch Hole) 메커니즘
##### 희소 파일 (Sparse File) 아키텍처
##### 파일 시스템 종속성
##### 운영체제 시스템 콜 (fallocate)
#### 성능 특성
##### I/O 대역폭 절감
##### 지연 시간 변동성

### 테이블 압축 (ROW_FORMAT=COMPRESSED)
#### 압축 알고리즘
##### zlib 라이브러리 연동
##### 데이터 블록 단위 처리
#### KEY_BLOCK_SIZE 설정
##### 페이지 크기 결정 논리
##### 압축 효율과 성능의 상관관계
#### 압축 데이터 저장 구조
##### 압축 페이지 구성 요소
##### 수정 로그 (Modification Log) 영역

### 압축 동작 프로세스
#### 페이지 분할 (Page Split)
##### 압축 실패 처리 메커니즘
##### 데이터 재배치 (Reorganization)
#### 버퍼 풀 관리
##### 압축 페이지 캐싱 전략
##### 비압축 페이지(Uncompressed Page) 유지 정책

### 압축 오버헤드 및 트레이드오프
#### CPU 부하 분석
##### 압축/해제 사이클 소모
##### 병렬 처리 한계
#### 메모리 단편화
##### 버퍼 풀 효율 저하
##### LRU 리스트 관리 복잡도
#### 데이터 타입별 압축률
##### 수치형 데이터 특성
##### 문자열 데이터 특성
##### 이진 데이터(BLOB/TEXT) 특성

### 압축 효율 모니터링
#### 정보 스키마 (Information Schema)
##### INNODB_CMP
##### INNODB_CMP_PER_INDEX
#### 성능 지표 분석
##### 압축 성공률 (Success Rate)
##### 압축 소요 시간 추적

## 데이터 암호화

### TDE (Transparent Data Encryption)
#### 암호화 아키텍처
##### 계층적 키 구조
##### 마스터 키 (Master Key)
##### 테이블스페이스 키 (Tablespace Key)
#### 데이터 페이지 암호화
##### 쓰기 시점 암호화 프로세스
##### 읽기 시점 복호화 프로세스
##### 암호화 알고리즘 (AES-256)

### 키 관리 시스템 (Key Management)
#### 키링(Keyring) 플러그인
##### keyring_file 플러그인
##### keyring_encrypted_file 플러그인
#### 외부 KMS 연동
##### AWS KMS 플러그인
##### HashiCorp Vault 플러그인
#### 키 로테이션 (Key Rotation)
##### 마스터 키 갱신 절차
##### 테이블스페이스 키 재암호화

### 로그 및 임시 파일 암호화
#### 리두 로그 (Redo Log) 암호화 -
##### 로그 레코드 보안 처리
#### 언두 로그 (Undo Log) 암호화
##### 트랜잭션 롤백 데이터 보호
#### 바이너리 로그 (Binary Log) 암호화
##### 복제 데이터 보안
#### 임시 테이블 (Temporary Table) 암호화
##### 중간 연산 결과 보호 -

### 통신 암호화 (Data-in-Transit)
#### SSL/TLS 프로토콜
##### 지원 버전 명세
##### 암호화 스위트 (Cipher Suite) 선정
#### 인증서 관리
##### CA 인증서 체인
##### 서버 및 클라이언트 인증서
#### 연결 보안 수준 (ssl-mode)
##### REQUIRED 모드
##### VERIFY_CA 모드
##### VERIFY_IDENTITY 모드

### 암호화 성능 영향도
#### CPU 오버헤드
##### 암호 연산 가속 (AES-NI)
##### 커널 수준 컨텍스트 스위칭
#### 스토리지 처리량 변동
##### 입출력 지연 시간(Latency) 증가율
#### 메모리 사용량 변화
##### 복호화 페이지 캐싱 효율

### 보안 운영 및 제약 사항
#### 암호화 적용 대상 식별
##### 테이블 단위 설정
##### 테이블스페이스 단위 설정
#### 하부 시스템 제약
##### 파일 시스템 압축과의 호환성
##### 운영체제 백업 툴 호환성

## 인덱스

### B-Tree 인덱스
#### 자료구조적 특성
##### 루트 노드
##### 브랜치 노드
##### 리프 노드
##### 노드 페이지 헤더
#### 인덱스 조작 메커니즘
##### 인덱스 탐색 (Index Seek)
##### 인덱스 키 추가 및 정렬
##### 페이지 분할 (Page Split)
##### 페이지 병합 (Page Merge)
##### 인덱스 키 삭제 및 변경
#### 인덱스 스캔 방식
##### 인덱스 레인지 스캔 (Index Range Scan)
##### 인덱스 풀 스캔 (Index Full Scan)
##### 루스 인덱스 스캔 (Loose Index Scan)
##### 인덱스 스킵 스캔 (Index Skip Scan)

### Hash 인덱스
#### 해시 함수 알고리즘
#### 버킷(Bucket) 및 슬롯(Slot) 구조
#### 해시 충돌(Collision) 해결 기전
#### 메모리 기반 성능 특성

### InnoDB 특화 인덱스 아키텍처
#### 클러스터링 인덱스 (Clustered Index)
##### 프라이머리 키 선택 전략
##### 물리적 데이터 배치 논리
##### 클러스터링 테이블 구조적 이점
#### 세컨더리 인덱스 (Secondary Index)
##### 프라이머리 키 참조 방식
##### 세컨더리 인덱스 탐색 비용 분석
#### 어댑티브 해시 인덱스 (Adaptive Hash Index)
##### 인덱스 접근 패턴 감지 알고리즘
##### 내부 래치(Latch) 경합 및 오버헤드
##### 동적 활성화 제어

### 확장 인덱스 기술
#### 복합 인덱스 (Composite Index)
##### 컬럼 배치 순서와 변별력
##### 다중 컬럼 정렬 논리
#### 함수 기반 인덱스 (Function-based Index)
##### 가상 컬럼 인덱스
##### 함수 결과값 영속화 메커니즘
#### 멀티 밸류 인덱스 (Multi-value Index)
##### JSON 배열 원소 인덱싱
##### MEMBER OF() 연산 최적화
#### 내림차순 인덱스 (Descending Index)
##### 인덱스 정렬 방향 정의
##### 역방향 스캔(Backward Scan) 오버헤드

### 인덱스 설계 및 전략
#### 카디널리티 (Cardinality) 분석
##### 유니크 수치와 선택도 (Selectivity)
#### 인덱스 가용성 판단
##### 조건절 위치에 따른 인덱스 활용성
##### 인덱스 사용 불가 시나리오
#### 커버링 인덱스 (Covering Index)
##### 인덱스만 이용한 쿼리 처리
##### 데이터 페이지 접근 생략 효과

### 인덱스 성능 최적화 및 유지보수
#### 인덱스 통계 정보 (Index Statistics)
##### 통계 정보 수집 알고리즘
##### 영구적 통계 정보 관리
#### 인덱스 단편화 (Fragmentation)
##### 빈 공간(Fill Factor) 제어
##### 인덱스 재구성 (Optimize Table)
#### 인덱스 경합 관리
##### 인덱스 쓰기 버퍼링 (Change Buffer)
##### 쓰기 부하와 읽기 성능의 트레이드오프

## 옵티마이저와 힌트

### 옵티마이저 아키텍처
#### 비용 기반 최적화 (CBO)
##### 논리적 최적화 단계
##### 물리적 최적화 단계
#### 규칙 기반 최적화 (RBO)
##### 레거시 아키텍처의 유산
##### 우선순위 기반 실행 경로

### 비용 모델 (Cost Model)
#### 통계 정보 (Statistics)
##### 인덱스 통계 수집 알고리즘
##### 데이터 히스토그램 (Histogram)
##### 싱글톤 및 등간격 버킷
#### 연산 비용 계산
##### I/O 비용 (Disk Fetch)
##### CPU 비용 (Memory Row Process)
##### 메모리 비용 (Temporary Table)

### 쿼리 재작성 (Query Transformation)
#### 조건절 상수화 (Constant Folding)
#### 불필요한 조건 제거
#### 서브쿼리 최적화
##### 세미 조인 (Semi-join) 변환
##### 파생 테이블 (Derived Table) 병합
#### 카테시안 조인 변환

### 옵티마이저 스위치 (Optimizer Switches)
#### 인덱스 머지 (Index Merge)
##### 교집합 (Intersection) 방식
##### 합집합 (Union) 방식
##### 정렬-합집합 (Sort-Union) 방식
#### 인덱스 컨디션 푸시다운 (ICP)
##### 스토리지 엔진 레벨 조건 필터링
#### 멀티 레인지 읽기 (MRR)
##### 디스크 I/O 정렬 및 최적화
#### 해시 조인 (Hash Join)
##### 빌드 단계 (Build Phase)
##### 프로브 단계 (Probe Phase)

### 인덱스 힌트 (Index Hints)
#### 인덱스 강제 적용
##### USE INDEX
##### FORCE INDEX
##### IGNORE INDEX
#### 힌트 적용 범위 (Scope)
##### 조인 대상 인덱스
##### 정렬 대상 인덱스
##### 그룹화 대상 인덱스

### 옵티마이저 힌트 (Optimizer Hints)
#### 전역 힌트
##### MAX_EXECUTION_TIME
##### SET_VAR
#### 조인 순서 제어
##### JOIN_ORDER
##### LEADING
#### 조인 알고리즘 제어
##### BNL_HASH
##### NO_BNL_HASH
#### 인덱스 레벨 힌트
##### GROUP_REPLICATION
##### INDEX_MERGE

### 옵티마이저 트레이스 (Optimizer Trace)
#### 추적 활성화 설정
#### JSON 산출물 구조 분석
##### join_preparation
##### join_optimization
##### join_execution
#### 비용 계산 근거 추적

## 실행 계획

### EXPLAIN 명령어
#### 구문 및 사용법
#### 출력 포맷 옵션
##### TEXT 포맷 (전통적 방식)
##### JSON 포맷 (상세 구조화 방식)
##### TREE 포맷 (계층적 실행 순서 방식)

### 실행 계획 공통 컬럼
#### id
##### 쿼리 식별자 및 실행 순서
#### select_type
##### SIMPLE
##### PRIMARY
##### UNION
##### SUBQUERY
##### DERIVED
##### MATERIALIZED
#### table
##### 대상 테이블 명칭
##### 파생 테이블 별칭

### 접근 방법 (type 컬럼)
#### 성능 우수군
##### system
##### const
##### eq_ref
##### ref
#### 인덱스 활용군
##### fulltext
##### ref_or_null
##### index_merge
##### unique_subquery
##### index_subquery
##### range
#### 성능 취약군
##### index (인덱스 풀 스캔)
##### ALL (풀 테이블 스캔)

### 인덱스 및 필터링 지표
#### possible_keys
##### 사용 가능 인덱스 후보
#### key
##### 최종 선택 인덱스
#### key_len
##### 선택된 인덱스의 바이트 길이
#### ref
##### 비교 대상 컬럼 또는 상수
#### rows
##### 예상 레코드 조사 수치
#### filtered
##### 조건 필터링 효율(%)

### Extra 컬럼 (상세 동작 정보)
#### 데이터 추출 최적화
##### Using index (커버링 인덱스)
##### Using where
##### Using index condition (ICP)
#### 성능 병목 식별
##### Using filesort
##### Using temporary
##### Using join buffer
##### Distinct
##### Impossible WHERE

### 실행 계획 분석 도구
#### EXPLAIN ANALYZE
##### 실제 실행 시간 측정 (Actual Time)
##### 루프 반복 횟수 분석
##### 로우 카운트 실측
#### 비주얼 익스플레인 (Visual Explain)
##### 연산자 그래프 가시화
#### 쿼리 코스트 분석
##### query_cost 수치 해석

## 쿼리 작성 및 최적화

### SELECT 최적화
#### 필드 선정
##### 컬럼 프로젝션(Column Projection) 제한
##### 불필요한 데이터 직렬화 오버헤드
#### LIMIT 처리
##### 상위 N개 레코드 추출 논리
##### OFFSET 페이징의 선형적 비용 증가
##### 지연된 조인(Deferred Join) 기법

### WHERE 절 최적화
#### SARG (Searchable Arguments) 준수
##### 컬럼 가공에 따른 인덱스 활용 제약
##### 인덱스 가용 연산자 분석
#### 데이터 타입 비교
##### 묵시적 형변환(Implicit Type Conversion) 결함
##### 문자셋 및 콜레이션(Collation) 불일치
#### 논리 연산자 처리
##### AND 조건과 인덱스 결합
##### OR 조건과 Index Merge

### 정렬 및 그룹화 최적화
#### ORDER BY 처리
##### 인덱스 이용 정렬 (Index-based Sort)
##### 드라이빙 테이블 기반 정렬
##### 임시 테이블 기반 정렬
##### 정렬 버퍼(Sort Buffer) 할당 및 반환
#### GROUP BY 처리
##### 타이트 인덱스 스캔 (Tight Index Scan)
##### 루스 인덱스 스캔 (Loose Index Scan)
##### 임시 테이블 및 정렬 처리
#### DISTINCT 처리
##### 인덱스 활용 중복 제거
##### 정렬 없는 중복 제거

### JOIN 최적화
#### 조인 순서 결정
##### 드라이빙(Driving) 테이블 선정
##### 드리븐(Driven) 테이블 인덱스 전략
#### 조인 알고리즘
##### 네스티드 루프 조인 (Nested Loop Join)
##### 블록 네스티드 루프 조인 (Block Nested Loop Join)
##### 해시 조인 (Hash Join)
#### 조인 형태별 최적화
##### 내부 조인 (Inner Join)
##### 외부 조인 (Outer Join)의 인덱스 제약
##### 안티 조인 (Anti Join) 및 세미 조인 (Semi Join)

### 서브쿼리 최적화
#### 서브쿼리 분류
##### 스칼라 서브쿼리 캐싱
##### 상관 서브쿼리 (Correlated Subquery) 오버헤드
#### 쿼리 변환 (Transformation)
##### 서브쿼리 조인 변환 (Subquery Unnesting)
##### 파생 테이블 구체화 (Materialization)
##### EXISTS vs IN 성능 차이

### 고급 SQL 최적화
#### 윈도우 함수 (Window Function)
##### 윈도우 프레임 정의 비용
##### 정렬 및 파티셔닝 오버헤드
#### CTE (Common Table Expression)
##### 비재귀적 CTE의 가독성과 성능
##### 재귀적 CTE의 무한 루프 제약 및 성능
#### 가상 컬럼 및 생성 컬럼 최적화

### 쿼리 프로파일링
#### SET PROFILING 활용
#### SHOW PROFILE 세부 항목 분석
##### CPU 및 I/O 소모량 추적
##### 컨텍스트 스위칭 발생 지점 파악

## 확장 검색

### 전문 검색 (Full-Text Search)
#### 인덱스 아키텍처
##### 역 인덱스(Inverted Index) 구조
##### 토크나이저(Tokenizer) 분석
###### 내장 파서 (Built-in Parser)
###### N-gram 파서 메커니즘
###### MeCab 형태소 분석기 플러그인
##### 불용어(Stopword) 처리 시스템
#### 검색 모드 및 연산
##### 자연어 검색 (Natural Language Search)
##### 불리언 모드 검색 (Boolean Mode Search)
##### 쿼리 확장 검색 (Query Expansion Search)
#### 성능 최적화 및 제약
##### 인덱스 캐시 및 델타 버퍼링
##### 최소/최대 단어 길이 설정
##### 전문 검색 가용성 판단 기준

### 공간 정보 인덱싱 (Spatial Indexing)
#### 공간 데이터 모델
##### WKT(Well-Known Text) 표현식
##### WKB(Well-Known Binary) 물리 저장
##### SRID(Spatial Reference System Identifier) 관리
#### R-Tree 인덱스 아키텍처
##### MBR(Minimum Bounding Rectangle) 논리
##### MBR 계층적 중첩 및 분할 알고리즘
##### 공간 데이터 타입별 인덱싱 제약
#### 공간 연산 및 최적화
##### ST_ 연산자 체계 (ST_Contains, ST_Distance 등)
##### 공간 함수 필터링 및 조인 최적화
##### 공간 인덱스 가용 범위 및 예외 케이스

### 확장 검색의 트레이드오프
#### 인덱스 관리 비용
##### 전문 검색 인덱스의 쓰기 증폭 현상
##### 공간 인덱스의 페이지 분할 오버헤드
#### 정밀도와 성능의 균형
##### N-gram 토큰 크기에 따른 저장 공간 변화
##### MBR 근사치 계산에 따른 실제 필터링 비용

## 파티션

### 파티션 아키텍처
#### 파티션 정의 및 동작 원리
##### 물리적 파일 분할 구조
##### 스토리지 엔진 레이어의 파티션 핸들링
#### 파티션 사용의 이점
##### 성능 최적화 (I/O 분산)
##### 관리 편의성 (Data Aging)
##### 가용성 향상

### 파티션 유형
#### 레인지 파티션 (Range Partitioning)
##### 범위 기반 분할 논리
##### NULL 값 처리 방식
##### MAXVALUE 활용 전략
#### 리스트 파티션 (List Partitioning)
##### 명시적 값 목록 매핑
##### 다중 컬럼 리스트 파티션
#### 해시 파티션 (Hash Partitioning)
##### 해시 함수 및 모듈로(Modulo) 연산
##### 선형 해시 파티션 (Linear Hash Partitioning)
#### 키 파티션 (Key Partitioning)
##### 내부 해시 함수 알고리즘
#### 서브 파티션 (Subpartitioning)
##### 복합 파티션 설계 (Composite Partitioning)

### 파티션 프루닝 (Partition Pruning)
#### 프루닝 메커니즘
##### 쿼리 조건절 분석 및 파티션 선정
##### 정적 프루닝과 동적 프루닝
#### 실행 계획과 파티션
##### partitions 컬럼 식별
##### 프루닝 실패 시나리오 분석

### 파티션 관리 연산
#### 파티션 추가 및 삭제
##### ADD PARTITION의 물리적 영향
##### DROP PARTITION을 이용한 데이터 대량 삭제
#### 파티션 재구성 및 수정
##### REORGANIZE PARTITION 메커니즘
##### COALESCE PARTITION (해시/키 파티션 축소)
#### 파티션 데이터 교환 (Exchange Partition)
##### 테이블과 파티션 간의 데이터 스와핑
##### 무중단 데이터 이관 전략

### 파티션 제약 사항 및 주의 사항
#### 인덱스 및 키 제약
##### 프라이머리 키와 파티션 컬럼의 관계
##### 유니크 인덱스 구성 시의 제약 조건
#### 세컨더리 인덱스 오버헤드
##### 로컬 인덱스(Local Index) 구조적 한계
##### 파티션 개수 증가에 따른 인덱스 탐색 비용
#### 리소스 제약
##### 오픈 파일 핸들러(Open File Handler) 관리
##### 파티션 개수와 메모리 점유량의 상관관계

### 파티션 성능 최적화 전략
#### 파티션 키 선정 기준
##### 데이터 분포도(Cardinality) 고려
##### 쿼리 패턴과의 일치성
#### 파티션 유지보수
##### 파티션 통계 정보 관리
##### 인덱스 단편화 및 재구성

## 스토어드 프로그램

### 스토어드 프로시저 (Stored Procedure)
#### 프로시저 정의 및 호출
##### 매개변수 모드 (IN, OUT, INOUT)
##### CALL 문체 및 리턴값 처리
#### 제어 구조
##### 조건문 (IF, CASE)
##### 반복문 (LOOP, REPEAT, WHILE)
##### 커서 (Cursor) 제어 및 결과 집합 순회
#### 에러 핸들링
##### 컨디션 핸들러 (Condition Handler)
##### 진단 영역 (Diagnostics Area) 분석

### 스토어드 함수 (Stored Function)
#### 함수 정의 및 특성
##### RETURNS 절 및 데이터 타입
##### 결정론적 함수 (DETERMINISTIC) 여부
#### 함수 제약 사항
##### SQL 문장 제약
##### 사이드 이펙트(Side Effect) 관리
#### 사용자 정의 함수 (UDF) 연동
##### 외부 라이브러리 로딩 메커니즘

### 트리거 (Trigger)
#### 실행 시점 및 이벤트
##### BEFORE 및 AFTER 트리거
##### INSERT, UPDATE, DELETE 이벤트 매핑
#### 트리거 데이터 참조
##### NEW 및 OLD 레코드 의사 행(Pseudo-row)
#### 트리거 연쇄 및 제한
##### 재귀적 트리거 방지
##### 트리거 내 트랜잭션 제어 제약

### 이벤트 (Event)
#### 이벤트 스케줄러 아키텍처
##### 스케줄러 스레드 활성화 (event_scheduler)
#### 시간 기반 실행 정의
##### 단발성 실행 (AT)
##### 반복 실행 (EVERY)
#### 이벤트 상태 관리
##### ENABLE, DISABLE, SLAVESIDE_DISABLED

### 실행 환경 및 아키텍처
#### 파싱 및 컴파일 메커니즘
##### 프로시저 본문 프리파싱 (Pre-parsing)
##### 실행 시점 최적화 및 캐싱
#### 메모리 관리
##### 스레드 로컬 영역 할당
##### 스토어드 프로그램 캐시 (Stored Program Cache)
#### 컨텍스트 스위칭
##### SQL 엔진과 프로시저 실행기 간의 제어권 전환

### 성능 및 보안 최적화
#### 실행 권한 모델
##### DEFINER 기반 실행
##### INVOKER 기반 실행
#### 성능 오버헤드 분석
##### 인터프리터 실행 방식의 한계
##### 루프 내 SQL 실행의 I/O 증폭
#### 디버깅 및 프로파일링
##### SHOW PROCEDURE STATUS
##### 성능 스키마 기반 스태틱 분석

## 데이터 타입

### 숫자 타입
#### 정수형 (Integer)
##### 저장 공간 및 유효 범위
##### UNSIGNED 속성 및 산술 연산 비헤이비어
##### 제로필(ZEROFILL)과 표시 폭
#### 부동 소수점 (Floating-Point)
##### IEEE 754 표준 수용
##### FLOAT 및 DOUBLE의 정밀도 한계
##### 근사치 저장에 따른 비교 연산 주의점
#### 고정 소수점 (Fixed-Point)
##### DECIMAL 타입의 바이너리 저장 구조
##### 정밀도(Precision)와 스케일(Scale) 제어
##### 금융 데이터 처리를 위한 정밀도 보장 논리

### 문자열 타입
#### CHAR 및 VARCHAR
##### 고정 길이와 가변 길이의 물리적 레이아웃
##### 가변 길이 정보 저장용 추가 바이트(1~2 bytes)
##### 문자셋에 따른 바이트 소비량 가변성
#### TEXT 및 BLOB
##### 대용량 데이터의 오프 페이지(Off-page) 저장 메커니즘
##### 인라인 저장 임계치와 포인터 구조
##### 부분 업데이트 성능 최적화 제약
#### ENUM 및 SET
##### 정수 매핑 기반 내부 저장 방식
##### 스키마 변경(ALTER TABLE) 시의 비용 분석

### 날짜 및 시간 타입
#### DATETIME 및 TIMESTAMP
##### 시간대(Timezone) 종속성 및 변환 로직
##### 내부 저장 바이트 및 분수 초(Fractional Seconds) 정밀도
##### 2038년 문제(Unix Epoch)와 타입 선택 전략
#### DATE, TIME, YEAR
##### 날짜 연산 최적화 및 인덱스 가용성

### 이진 데이터 타입
#### BINARY 및 VARBINARY
##### 패딩(Padding) 및 비교 연산 특성
#### 비트 값 타입 (BIT)
##### 비트 필드 저장 및 마스킹 연산

### JSON 타입
#### 바이너리 JSON 구조
##### 트리 형태의 구조화된 데이터 저장
##### 키-값 검색 가속화를 위한 내부 인덱싱
#### JSON 부분 업데이트 (Partial Update)
##### 전체 문서 재기록 방지 메커니즘
##### 리두 로그(Redo Log) 효율화
#### 가상 컬럼 및 멀티 밸류 인덱스 연동

### 문자셋 및 콜레이션
#### 멀티바이트 문자셋 (UTF-8mb4)
##### 가변 길이 인코딩 엔진의 성능 영향도
#### 콜레이션 (Collation)
##### 정렬 가중치(Weight)와 비교 논리
##### CI(Case Insensitive) 및 CS(Case Sensitive) 처리
#### 바이너리 콜레이션
##### 코드 포인트 기반 고속 비교

### 데이터 타입 선택 전략
#### 저장 효율과 쿼리 성능의 트레이드오프
##### 정렬 성능을 고려한 고정 길이 타입 활용
##### 가독성과 공간 효율 사이의 타입 결정

## 복제

### 복제 아키텍처
#### 소스 및 레플리카 역할 정의
#### 바이너리 로그 (Binary Log)
##### 로그 이벤트 구조
##### 로그 로테이션 및 퍼지(Purge) 정책
#### 릴레이 로그 (Relay Log)
##### 중간 저장소의 역할
##### 로그 재생 및 인덱스 관리

### 복제 스레딩 모델
#### 소스 서버 스레드
##### 바이너리 로그 덤프 스레드
#### 레플리카 서버 스레드
##### I/O 스레드 (네트워크 수신)
##### SQL 스레드 (로그 재생)
#### 멀티 스레드 복제 (Parallel Replication)
##### 데이터베이스 단위 병렬화
##### 논리적 시계(Logical Clock) 기반 병렬화
##### 쓰기 세트(Write Set) 기반 병렬화

### 복제 포맷
#### 문장 기반 복제 (Statement Based Replication)
##### 비확정적 쿼리(Non-deterministic Query) 결함
#### 로우 기반 복제 (Row Based Replication)
##### 이미지 포맷(Full, Minimal, Noblob) 제어
##### 데이터 일관성 보장 수준
#### 믹스드 포맷 (Mixed Format)
##### 자동 전환 로직 및 판단 기준

### 복제 동기화 방식
#### 비동기 복제 (Asynchronous Replication)
##### 성능 이점과 데이터 손실 위험
#### 반동기 복제 (Semi-synchronous Replication)
##### ACK 응답 메커니즘
##### 소스 타임아웃 및 비동기 전환
##### AFTER_SYNC 및 AFTER_COMMIT 차이

### GTID (Global Transaction Identifier)
#### GTID 구조
##### UUID 및 트랜잭션 ID 조합
#### GTID 기반 복제 관리
##### 복제 토폴로지 자동 복구
##### 빈 구멍(Gap) 발생 시의 처리 논리
#### 저장 및 정합성
##### gtid_executed 및 gtid_purged 시스템 변수

### 복제 토폴로지 구성
#### 단방향 복제 (Master-Slave)
#### 양방향 복제 (Master-Master)
##### 충돌 감지 및 방지 제약
#### 연쇄 복제 (Chained Replication)
##### log_slave_updates 옵션의 역할
#### 멀티 소스 복제 (Multi-Source Replication)
##### 복제 채널(Channel) 관리

### 복제 필터링 및 관리
#### 필터링 규칙
##### binlog-do-db 및 binlog-ignore-db
##### replicate-wild-do-table
#### 복제 지연(Replication Lag) 분석
##### Seconds_Behind_Master 지표의 허구와 실체
##### 하트비트(Heartbeat) 메커니즘

### 복제 보안 및 최적화
#### 복제 통신 암호화
##### SSL/TLS 인증서 연동
#### 압축 복제
##### 네트워크 대역폭 절감 효율
#### 복제 사용자 권한
##### REPLICATION SLAVE 권한 제어

## InnoDB 클러스터

### 클러스터 아키텍처
#### 핵심 구성 요소
##### 그룹 복제 (Group Replication)
##### MySQL 라우터 (MySQL Router)
##### MySQL 셸 (MySQL Shell)
#### 데이터 흐름 및 상호작용
##### 클라이언트 연결 흐름
##### 노드 간 상태 동기화

### 그룹 복제 (Group Replication)
#### 합의 알고리즘
##### Paxos 기반 분산 합의 메커니즘
##### 쿼럼(Quorum) 결정 논리
#### 복제 모드
##### 단일 주 서버 모드 (Single-Primary Mode)
##### 다중 주 서버 모드 (Multi-Primary Mode)
#### 데이터 정합성 보장
##### 낙관적 복제 (Optimistic Replication)
##### 인증 프로세스 (Certification Process)
##### 충돌 감지 및 해결 알고리즘
#### 그룹 멤버십 관리
##### 뷰 변경 (View Change) 메커니즘
##### 장애 탐지 및 노드 퇴출

### MySQL 라우터 (MySQL Router)
#### 프록시 및 라우팅 전략
##### 읽기/쓰기 분리 (Read/Write Splitting)
##### 라운드 로빈 로드 밸런싱
#### 메타데이터 캐시
##### 클러스터 상태 실시간 동기화
#### 고가용성 연결 관리
##### 투명한 장애 조치 (Transparent Failover)
##### 커넥션 타임아웃 및 재시도 정책

### MySQL 셸 (MySQL Shell)
#### 클러스터 관리 인터페이스
##### AdminAPI 프로비저닝
##### 클러스터 인스턴스 구성 (ClusterSet)
#### 상태 모니터링 및 진단
##### describe() 및 status() 분석

### 가용성 및 정합성 제어
#### 흐름 제어 (Flow Control)
##### 지연 발생 시 쓰기 처리량 조절
##### 복제 지연 임계치 관리
#### 분할 뇌 (Split-brain) 방지
##### 네트워크 파티션 발생 시 의사결정
##### 강제 쿼럼 형성 및 복구
#### 일관성 수준 (Consistency Levels)
##### EVENTUAL
##### BEFORE_ON_PRIMARY_FAILOVER
##### BEFORE
##### AFTER
##### BEFORE_AND_AFTER

### 장애 복구 및 운영
#### 노드 복구 프로세스
##### 분산 복구 (Distributed Recovery)
##### 바이너리 로그 기반 증분 복구
#### 온라인 구성 변경
##### 주 서버 교체 (Primary Election)
##### 동적 설정 업데이트

### 성능 영향 및 제약 사항
#### 오버헤드 분석
##### 합의 단계의 네트워크 지연
##### CPU 및 메모리 자원 점유량
#### 토폴로지 제약
##### 최대 노드 수 제한
##### 인덱스 및 테이블 제약 (PK 필수)

## Performance 스키마 & Sys 스키마

### Performance 스키마 아키텍처
#### 계측기 (Instruments)
##### 코드 수준의 계측 지점 정의
##### 명명 규칙 및 계층 구조
#### 컨슈머 (Consumers)
##### 이벤트 데이터 저장 대상
##### 필터링 및 집계 로직
#### 데이터 수집 메커니즘
##### 비잠금(Lock-free) 알고리즘
##### 메모리 버퍼 할당 및 관리 정책

### 설정 및 구성
#### setup_instruments 테이블
##### 계측기 활성화 및 타이밍 설정
#### setup_consumers 테이블
##### 계측 데이터의 저장 여부 결정
#### setup_actors 및 setup_objects
##### 모니터링 대상 사용자 및 객체 필터링
#### setup_threads
##### 백그라운드 및 포어그라운드 스레드 모니터링 제어

### 이벤트 모니터링
#### 대기 이벤트 (Wait Events)
##### 디스크 I/O 대기
##### 뮤텍스(Mutex) 및 읽기/쓰기 락(RW-Lock) 경합
#### 단계 이벤트 (Stage Events)
##### 쿼리 실행 단계별 소요 시간 (Parsing, Planning, Executing)
#### 구문 이벤트 (Statement Events)
##### 실행된 SQL 쿼리의 통계 및 속성
#### 트랜잭션 이벤트 (Transaction Events)
##### 격리 수준 및 상태 추적

### 리소스 및 상태 추적
#### 메모리 요항 (Memory Summary)
##### 스토리지 엔진별 메모리 점유량
##### 사용자 세션별 메모리 사용량
#### 변수 및 상태
##### 전역 및 세션 시스템 변수 실시간 모니터링
##### 상태 변수(Status Variables) 추적

### Sys 스키마
#### Sys 스키마 아키텍처
##### Performance 스키마 데이터의 추상화 레이어
#### 주요 뷰 (Views)
##### host_summary: 호스트별 자원 점유
##### user_summary: 사용자별 활동 분석
##### statement_analysis: 쿼리 성능 프로파일링
##### memory_global_by_current_bytes: 메모리 누수 추적
#### 진단 프로시저 (Procedures)
##### diagnostics: 시스템 전체 상태 리포트 생성
##### statement_performance_analyzer: 쿼리 성능 분석
#### 유틸리티 함수 (Functions)
##### 데이터 단위 변환 (format_bytes, format_time)

### 실무 진단 시나리오
#### 병목 쿼리 식별
##### 전체 테이블 스캔(Full Table Scan) 빈도 추적
##### 파일 정렬(Filesort) 발생 쿼리 탐지
#### 잠금 경합 분석
##### 대기 시간이 긴 로우 락(Row Lock) 식별
##### 메타데이터 락(Metadata Lock) 충돌 추적
#### 메모리 최적화
##### 비정상적 메모리 증가 지점 탐색