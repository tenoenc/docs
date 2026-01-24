## SQL 표준과 관계형 모델 (Foundations)

### SQL 표준의 역사와 진화
#### ANSI/ISO 표준 연혁
##### SQL-86 (SQL1)
##### SQL-89
##### SQL-92 (SQL2)
##### SQL:1999 (SQL3)
##### SQL:2011
##### SQL:2023 (최신 표준)
#### SQL의 언어적 본질
##### 선언적 패러다임 (Declarative)
##### 집합 지향성 (Set-Oriented)
##### 비절차적 실행 환경

### 관계형 모델의 이론적 기초
#### 에드거 F. 커드(E.F. Codd)의 12규칙
##### 정보의 규칙 (Information Rule)
##### 데이터 독립성 원칙
#### 도메인과 원자성
##### 도메인 (Domain)의 정의
##### 속성값의 원자성 (Atomicity)

### 릴레이션의 구조적 정의
#### 릴레이션 (Relation)
#### 튜플 (Tuple)
#### 속성 (Attribute)
#### 수량적 지표
##### 차수 (Degree)
##### 카디널리티 (Cardinality)

### 관계형 모델의 제약 조건 (Integrity Constraints)
#### 키(Key)의 개념과 종류
##### 슈퍼 키 (Super Key)
##### 후보 키 (Candidate Key)
##### 기본 키 (Primary Key)
##### 대체 키 (Alternate Key)
##### 외래 키 (Foreign Key)
#### 무결성 규정 (Integrity Rules)
##### 개체 무결성 (Entity Integrity)
##### 참조 무결성 (Referential Integrity)
##### 도메인 무결성 (Domain Integrity)

### 관계 대수 (Relational Algebra)
#### 일반 집합 연산
##### 합집합 (Union)
##### 교집합 (Intersection)
##### 차집합 (Difference)
##### 카테시안 곱 (Cartesian Product)
#### 순수 관계 연산
##### 셀렉트 (Select, σ)
##### 프로젝트 (Project, π)
##### 조인 (Join, ⋈)
##### 디비전 (Division, ÷)
#### 확장된 관계 대수
##### 세타 조인 (Theta Join)
##### 동등 조인 (Equi Join)
##### 자연 조인 (Natural Join)
##### 외부 조인 (Outer Join)

### 관계 해석 (Relational Calculus)
#### 튜플 관계 해석 (TRC)
#### 도메인 관계 해석 (DRC)

### 데이터 독립성 및 스키마 구조
#### 3단계 스키마 아키텍처
##### 외부 스키마 (External)
##### 개념 스키마 (Conceptual)
##### 내부 스키마 (Internal)
#### 데이터 독립성 (Independence)
##### 논리적 데이터 독립성
##### 물리적 데이터 독립성

## 데이터 타입과 도메인 정의 (Data Types)

### 수치 데이터 타입 (Numeric Types)
#### 정수형 (Exact Numerics - Integer)
##### SMALLINT
##### INTEGER (INT)
##### BIGINT
#### 고정 소수점 실수 (Exact Numerics - Decimal)
##### NUMERIC
##### DECIMAL (DEC)
#### 부동 소수점 실수 (Approximate Numerics)
##### FLOAT
##### REAL
##### DOUBLE PRECISION

### 문자 데이터 타입 (Character Types)
#### 고정 길이 문자열
##### CHARACTER (CHAR)
#### 가변 길이 문자열
##### CHARACTER VARYING (VARCHAR)
#### 대용량 객체 (LOB)
##### CHARACTER LARGE OBJECT (CLOB)
##### NATIONAL CHARACTER LARGE OBJECT (NCLOB)

### 날짜 및 시간 데이터 타입 (Temporal Types)
#### 시점 데이터 (Point in Time)
##### DATE
##### TIME (WITHOUT/WITH TIME ZONE)
##### TIMESTAMP (WITHOUT/WITH TIME ZONE)
#### 시간 간격 (Intervals)
##### YEAR TO MONTH
##### DAY TO SECOND

### 부울 데이터 타입 (Boolean Type)
#### BOOLEAN
##### 3치 논리 (TRUE, FALSE, UNKNOWN)

### 이진 데이터 타입 (Binary Types)
#### 고정 길이 이진 문자열
##### BINARY
#### 가변 길이 이진 문자열
##### BINARY VARYING (VARBINARY)
#### 대용량 이진 객체 (BLOB)
##### BINARY LARGE OBJECT

### 반정형 데이터 타입 (Semi-structured Types)
#### XML 데이터 타입
##### XML (SQL:2003 표준)
#### JSON 데이터 타입
##### JSON (SQL:2016/2023 표준)
#### JSON 조작 함수 (SQL:2016)
##### JSON_VALUE / JSON_TABLE
##### JSON 생성 (OBJECT / ARRAY)

### 시계열 데이터 관리 (Temporal SQL)
#### 시스템 버전화 테이블 (System-Versioning)
#### 시점 조회 (FOR SYSTEM_TIME AS OF)

### 사용자 정의 및 도메인 정의 (User-Defined & Domains)
#### 도메인 (Domain)
##### 기본값 정의 (DEFAULT)
##### 도메인 제약 조건 (CHECK)
#### 사용자 정의 타입 (UDT)
##### 고유 타입 (Distinct Types)
##### 구조체 타입 (Structured Types)

### 컬렉션 및 참조 타입 (Collections & References)
#### 배열 (ARRAY)
#### 멀티셋 (MULTISET)
#### 참조 (REF)

## 데이터 스키마 정의 (DDL)

### 스키마 및 카탈로그 체계
#### 카탈로그 (Catalog)의 역할
#### 스키마 (Schema) 관리
##### 스키마 생성 (CREATE SCHEMA)
##### 스키마 제거 (DROP SCHEMA)
##### 스키마 경로 (PATH) 정의

### 테이블 구조 정의 및 관리
#### 테이블 생성 (CREATE TABLE)
##### 영구 테이블 (Base Table)
##### 임시 테이블 (Global/Local Temporary Table)
##### 컬럼 정의 (Column Definition)
##### 기본값 명시 (DEFAULT Clause)
#### 테이블 변형 (ALTER TABLE)
##### 컬럼 추가 (ADD)
##### 컬럼 정의 수정 (ALTER/SET)
##### 컬럼 삭제 (DROP)
#### 테이블 제거 및 초기화
##### 테이블 파기 (DROP TABLE)
##### 데이터 즉시 절단 (TRUNCATE TABLE)

### 가상 테이블 및 유도 객체 관리
#### 뷰 정의 (CREATE VIEW)
##### 단순 뷰 (Simple View)
##### 복합 뷰 (Complex View)
##### 체크 옵션 (WITH CHECK OPTION)
#### 뷰 제거 (DROP VIEW)

### 보조 데이터 객체 관리
#### 인덱스 정의 (CREATE INDEX)
##### 고유 인덱스 (UNIQUE)
##### 복합 인덱스 (Composite)
#### 인덱스 제거 (DROP INDEX)
#### 시퀀스 및 생성기 (Sequence & Generator)
##### 시퀀스 생성 (CREATE SEQUENCE)
##### 시퀀스 변형 (ALTER SEQUENCE)
##### 시퀀스 제거 (DROP SEQUENCE)

### 객체 메타데이터 및 의존성 관리
#### 코멘트 관리 (Metadata)
##### 객체 주석 생성 (COMMENT ON)
#### 객체 파기 옵션 (Drop Behavior)
##### RESTRICT (제한적 삭제)
##### CASCADE (연쇄적 삭제)

## 데이터 조작 및 기초 질의 (DML & Basic DQL)

### 데이터 변경 및 생명주기 (DML)
#### 데이터 삽입 (INSERT)
##### 단일 행 삽입 (VALUES)
##### 서브쿼리를 이용한 삽입 (INSERT INTO ... SELECT)
#### 데이터 수정 (UPDATE)
##### 단순 값 수정 (SET)
##### 상관 서브쿼리 활용 수정
#### 복합 데이터 조작 (Upsert)
##### MERGE (MATCHED / NOT MATCHED)
#### 데이터 삭제 (DELETE)
##### 조건부 행 삭제 (WHERE)

### 기초 질의 구조 (Basic SELECT)
#### SELECT 절 (Projection)
##### 모든 컬럼 참조 (*)
##### 특정 컬럼 명시
##### 중복 제거 (DISTINCT)
##### 컬럼 별칭 (AS)
##### 산술 표현식 및 리터럴
#### FROM 절 (Source)
##### 테이블 참조
##### 테이블 별칭 (Correlation Name)
#### WHERE 절 (Selection/Filtering)
##### 비교 연산자 (=, <>, <, <=, >, >=)
##### 논리 연산자 (AND, OR, NOT)
##### 연산자 우선순위와 괄호
#### 조건부 표현식 (Conditional Expressions)
##### CASE (Simple / Searched)
##### NULL 처리 함수 (COALESCE, NULLIF)

### 특수 비교 연산자 (Special Predicates)
#### NULL 비교
##### IS NULL
##### IS NOT NULL
#### 범위 및 목록 비교
##### 범위 지정 (BETWEEN ... AND)
##### 목록 포함 (IN)
#### 패턴 매칭 (Pattern Matching)
##### 와일드카드 문자 (LIKE)
##### 이스케이프 절 (ESCAPE)

### 결과 데이터 정렬 및 제한
#### 정렬 제어 (ORDER BY)
##### 오름차순 및 내림차순 (ASC / DESC)
##### 다중 컬럼 정렬
##### NULL 정렬 순서 (NULLS FIRST / LAST)
#### 결과 집합 제한
##### 행 스킵 (OFFSET)
##### 반환 행 제한 (FETCH FIRST / NEXT)

## 집합 연산과 조인 (Sets & Joins)

### 집합 연산자 (Set Operators)
#### 연산의 전제 조건
##### 대응성 (Union Compatibility)
#### 합집합 연산
##### UNION (중복 제거)
##### UNION ALL (중복 허용)
#### 교집합 연산
##### INTERSECT (중복 제거)
##### INTERSECT ALL
#### 차집합 연산
##### EXCEPT (중복 제거)
##### EXCEPT ALL

### 조인 (Join)의 기초
#### 카테시안 곱 (CROSS JOIN)
#### 내부 조인 (INNER JOIN)
##### 등가 조인 (Equi-Join)
##### 비등가 조인 (Non-Equi Join)
##### 자연 조인 (NATURAL JOIN)
#### 조인 조건 명시
##### ON 절 (조건식)
##### USING 절 (컬럼명)

### 외부 조인 (Outer Join)
#### 왼쪽 외부 조인 (LEFT OUTER JOIN)
#### 오른쪽 외부 조인 (RIGHT OUTER JOIN)
#### 완전 외부 조인 (FULL OUTER JOIN)

### 특수 조인 및 관계 조작
#### 셀프 조인 (Self Join)
#### 안티 조인 및 세미 조인
##### 세미 조인 (Semi-Join)
##### 안티 조인 (Anti-Join)
#### 다중 조인 (Multiple Joins)
##### 조인 순서와 결합성

## 집계 함수와 그룹화 (Aggregation & Grouping)

### 집계 함수 (Aggregate Functions)
#### 표준 통계 함수
##### 개수 산출 (COUNT)
##### 합계 산출 (SUM)
##### 평균 산출 (AVG)
##### 최댓값 및 최솟값 (MAX / MIN)
#### 집계 데이터 제어
##### 중복 제외 집계 (DISTINCT)
##### NULL 처리 원칙 (Ignore NULLs)

### 데이터 그룹화 (Grouping)
#### GROUP BY 절
##### 단일 컬럼 그룹화
##### 다중 컬럼 그룹화
##### 그룹화 컬럼의 제약 (Non-aggregated Columns)

### 그룹 필터링 (Group Filtering)
#### HAVING 절
##### 집계 결과 기반 필터링
##### WHERE 절과의 논리적 실행 순서 차이

### 다차원 데이터 집계 (Advanced Grouping)
#### 집계 집합 (Grouping Sets)
##### ROLLUP (계층적 소계 및 합계)
##### CUBE (모든 조합의 교차 집계)
##### GROUPING SETS (특정 조합 명시)
#### 집계 상태 판별
##### GROUPING 함수 (소계 행 식별)

## 서브쿼리와 공통 테이블 식(CTE)

### 서브쿼리 (Subqueries)의 분류
#### 반환 형태별 원자 단위
##### 스칼라 서브쿼리 (Scalar)
##### 행 서브쿼리 (Row)
##### 테이블 서브쿼리 (Table)
#### 질의 내 위치별 분류
##### SELECT 절 서브쿼리
##### FROM 절 서브쿼리 (Inline View)
##### WHERE / HAVING 절 서브쿼리 (Nested)

### 서브쿼리 연산 및 술어
#### 존재 확인 (EXISTS / NOT EXISTS)
#### 다중 행 비교 연산
##### 목록 포함 (IN / NOT IN)
##### 부분 일치 (ANY / SOME)
##### 전체 일치 (ALL)

### 공통 테이블 식 (Common Table Expressions)
#### CTE의 기본 정의 (WITH Clause)
##### 단일 CTE 정의
##### 다중 CTE 정의 (Comma Separated)
#### CTE의 활용 이점
##### 가독성 및 유지보수성
##### 서브쿼리와의 차이점

### 재귀적 CTE (Recursive CTE)
#### 재귀 구조의 구성 요소
##### 앵커 멤버 (Anchor Member - 초기값)
##### 재귀 멤버 (Recursive Member - 반복부)
##### 결합 연산자 (UNION ALL)
#### 재귀 실행 제어
##### 종료 조건 (Termination Condition)
##### 최대 재귀 횟수 제한
#### 재귀 제어 옵션 (SQL:1999)
##### SEARCH (탐색 순서: DEPTH/BREADTH)
##### CYCLE (루프 방지 및 탐지)

## 고급 분석 및 윈도우 함수 (Window Functions)

### 윈도우 함수의 구문 구조
#### OVER 절 (Window Specification)
##### PARTITION BY (분석 범위 분할)
##### ORDER BY (정렬 및 연산 순서)
#### 윈도우 프레임 (Window Frame)
##### 프레임 단위 (ROWS / RANGE / GROUPS)
##### 시작과 끝 경계 (BETWEEN ... AND)
##### 현재 행 및 무한 범위 (CURRENT ROW / UNBOUNDED)

### 순위 및 분포 함수 (Ranking Functions)
#### 기본 순위 산출
##### RANK (중복 순위 다음 건너뜀)
##### DENSE_RANK (중복 순위 다음 연속)
##### ROW_NUMBER (고유 번호 부여)
#### 분포 및 비율
##### NTILE (n-등분)
##### PERCENT_RANK (상대적 백분율)
##### CUME_DIST (누적 분포)

### 탐색 및 위치 기반 함수 (Offset Functions)
#### 상대 위치 참조
##### LAG (이전 행 데이터)
##### LEAD (다음 행 데이터)
#### 프레임 내 위치 참조
##### FIRST_VALUE (첫 번째 행)
##### LAST_VALUE (마지막 행)
##### NTH_VALUE (N번째 행)

### 집계형 윈도우 연산 (Aggregate Window Functions)
#### 시계열 분석 및 요약
##### 누적 합계 (Cumulative SUM)
##### 이동 평균 (Moving Average)
#### 프레임 통계
##### 범위 내 최대/최소 (MAX / MIN)

### 윈도우 정의 재사용
#### WINDOW 절 (Named Windows)

## 트랜잭션 제어 및 격리 수준 (TCL & Isolation)

### 트랜잭션의 기본 성질 (ACID)
#### 원자성 (Atomicity)
##### All-or-Nothing 원칙
#### 일관성 (Consistency)
##### 무결성 제약 조건 준수
#### 격리성 (Isolation)
##### 동시 실행 트랜잭션 간 독립성
#### 지속성 (Durability)
##### 성공적 완료 후의 영구적 기록

### 트랜잭션 제어 언어 (TCL)
#### 트랜잭션 경계 설정
##### 시작 (START TRANSACTION / BEGIN)
##### 확정 (COMMIT)
##### 취소 (ROLLBACK)
#### 정밀 제어 메커니즘
##### 세이브포인트 (SAVEPOINT)
##### 세이브포인트로 롤백 (ROLLBACK TO)
##### 세이브포인트 해제 (RELEASE SAVEPOINT)

### 동시성 이상 현상 (Concurrency Phenomena)
#### Dirty Read (오손 읽기)
#### Non-repeatable Read (비반복 읽기)
#### Phantom Read (팬텀 읽기)

### ANSI 격리 수준 (Isolation Levels)
#### 격리 수준별 특성
##### READ UNCOMMITTED
##### READ COMMITTED
##### REPEATABLE READ
##### SERIALIZABLE
#### 트랜잭션 모드 설정
##### 격리 수준 지정 (SET TRANSACTION ISOLATION LEVEL)
##### 읽기 전용/쓰기 가능 모드 (READ ONLY / READ WRITE)
##### 진단 영역 크기 (DIAGNOSTICS SIZE)

## 무결성 제약 조건과 트리거 (Constraints & Triggers)

### 무결성 제약 조건 (Integrity Constraints)
#### 제약 조건의 정의 수준
##### 컬럼 레벨 정의 (Inline)
##### 테이블 레벨 정의 (Out-of-line)
#### 핵심 제약 조건 종류
##### NOT NULL (비널 제약)
##### UNIQUE (고유성 제약)
##### PRIMARY KEY (기본키 제약)
##### CHECK (범위 및 도메인 제약)
#### 제약 조건 관리 (Management)
##### 제약 조건 명명 (CONSTRAINT Name)
##### 제약 조건 활성화/비활성화
##### 제약 조건 삭제 (DROP CONSTRAINT)

### 참조 무결성 및 연쇄 동작 (Referential Integrity)
#### 외래 키 참조 (Foreign Key)
##### 자기 참조 (Self-referencing)
#### 참조 무결성 작업 (Referential Actions)
##### ON DELETE / ON UPDATE
##### CASCADE (연쇄 적용)
##### SET NULL / SET DEFAULT (상태 변경)
##### NO ACTION / RESTRICT (제한)

### 제약 조건 평가 시점
#### 즉시 평가 (IMMEDIATE)
#### 지연 평가 (DEFERRED)
##### DEFERRABLE 옵션
##### SET CONSTRAINTS 명령

### 트리거 (Triggers)
#### 트리거 정의 (CREATE TRIGGER)
#### 실행 이벤트 (Event)
##### INSERT / UPDATE / DELETE
#### 실행 시점 (Timing)
##### BEFORE / AFTER / INSTEAD OF
#### 실행 범위 (Orientation)
##### 행 트리거 (FOR EACH ROW)
##### 문장 트리거 (FOR EACH STATEMENT)

## 프로그래밍 객체 및 루틴 (Stored Routines & Advanced Views)

### 저장 프로시저 (Stored Procedures)
#### 프로시저 정의 및 호출
##### 프로시저 생성 (CREATE PROCEDURE)
##### 프로시저 호출 (CALL)
#### 매개변수 모드 (Parameter Modes)
##### 입력 전용 (IN)
##### 출력 전용 (OUT)
##### 입출력 겸용 (INOUT)

### 사용자 정의 함수 (User-Defined Functions)
#### 함수의 특성
##### 스칼라 함수 (Scalar Functions)
##### 테이블 반환 함수 (Table-valued Functions)
##### 결정적/비결정적 함수 (DETERMINISTIC)
#### 반환 구조
##### RETURNS 절 (반환 타입 명시)
##### RETURN 문 (실제 값 반환)

### 절차적 제어 및 흐름 (Procedural Logic)
#### 변수 및 스코프
##### 지역 변수 선언 (DECLARE)
##### 값 할당 (SET / SELECT INTO)
#### 제어문 (Control Statements)
##### 조건 분기 (IF-THEN-ELSE / CASE)
##### 반복 제어 (WHILE / LOOP / REPEAT)
##### 커서 제어 (CURSOR - OPEN/FETCH/CLOSE)

### 고급 뷰 및 예외 관리 (Advanced Views & Exception Handling)
#### 구체화된 뷰 (Materialized View)
##### 데이터 물리 저장 및 성능 최적화
##### 새로고침 전략 (REFRESH ON COMMIT / ON DEMAND)
#### 예외 및 상태 핸들링 (Error Handling)
##### 핸들러 선언 (DECLARE HANDLER)
##### 상태 조건 정의 (CONDITION)
##### 오류 신호 발생 (SIGNAL / RESIGNAL)

## 사용자 권한 및 보안 (DCL & Security)

### 권한 및 접근 제어 (Access Control)
#### 데이터 제어 언어 (DCL)
##### 권한 부여 (GRANT)
##### 권한 회수 (REVOKE)
#### 보안 객체 및 주체 관리
##### 사용자 (USER) 계정 생성 및 파기
##### 역할 (ROLE) 정의 및 그룹화
#### 권한 전파 및 위임
##### 관리 권한 포함 부여 (WITH GRANT OPTION)

### 정교한 보안 및 감사 (Advanced Security & Auditing)
#### 행 단위 보안 (Row-Level Security)
##### 보안 정책 (Policy) 정의
##### 사용자 컨텍스트 기반 필터링
#### 데이터 감시 및 감사 (Auditing)
##### 접근 기록 및 변경 이력 추적
#### 데이터 마스킹 (Data Masking)
##### 민감 정보의 동적 가상화

## 시스템 카탈로그 및 메타데이터 (Catalog & Metadata)

### 정보 스키마 (Information Schema)
#### 표준 메타데이터 뷰
##### TABLES / COLUMNS (테이블 및 컬럼 정보)
##### VIEWS (뷰 정의 정보)
##### TABLE_CONSTRAINTS (제약 조건 정보)
##### ROUTINES (프로시저 및 함수 정보)
#### 객체 정의 조회
##### 데이터 사전 (Data Dictionary) 활용