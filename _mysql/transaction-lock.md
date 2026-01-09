## 트랜잭션과 잠금

### ACID 원칙

**ACID 원칙**은 데이터베이스 트랜잭션이 안전하게 수행됨을 보장하기 위한 네 가지 핵심 속성인 원자성(Atomicity), 일관성(Consistency), 격리성(Isolation), 영속성(Durability)의 집합이다. 이는 단순한 이론적 정의를 넘어, 스토리지 엔진 내부의 언두 로그, 리두 로그, 잠금 시스템 및 데이터 정합성 검사 로직을 통해 물리적으로 구현되는 공학적 규약이다.

#### 데이터 정합성 보장을 위한 물리적 구현체 대조

ACID의 각 속성은 스토리지 엔진의 서로 다른 메커니즘을 통해 실체화된다. 각 속성이 어떠한 기술적 토대 위에 서 있는지 분석한다.

| 속성 | 핵심 정의 | 주요 구현 메커니즘 | 보장 실패 시 결과 |
| :--- | :--- | :--- | :--- |
| **원자성 (A)** | 트랜잭션 내 모든 연산의 '전부 혹은 전무(All or Nothing)' 실행 | 언두 로그 (Undo Log) | 부분적인 데이터 반영으로 인한 정합성 파괴 |
| **일관성 (C)** | 실행 전후의 데이터베이스가 정의된 규칙(제약 조건 등)을 준수 | 데이터 사전 및 제약 조건(Constraints) | 비즈니스 로직 및 스키마 위반 데이터 발생 |
| **격리성 (I)** | 동시에 실행되는 트랜잭션들이 서로 간섭하지 않음 | MVCC 및 잠금(Locking) | 더티 리드(Dirty Read), 팬텀 리드 등의 현상 |
| **영속성 (D)** | 성공적으로 커밋된 결과는 영구적으로 보존 | 리두 로그 (Redo Log) 및 Double Write Buffer | 시스템 장애 시 커밋된 데이터 유실 |

#### 트랜잭션 생명주기와 ACID 유지 단계 (Step-by-Step Trace)

하나의 수정 트랜잭션이 시작되어 완료되기까지 ACID가 어떻게 단계별로 적용되는지 추적한다.

1.  **시작 및 원자성 준비**: 트랜잭션 시작 시 언두 세그먼트를 할당한다. 모든 변경 직전의 데이터를 언두 로그에 기록하여 언제든 '전무(Nothing)' 상태로 복구할 수 있는 원자적 토대를 마련한다.
2.  **데이터 조작 및 격리 적용**: 데이터를 수정하는 동안 필요에 따라 레코드 잠금을 획득하거나 MVCC를 통해 다른 트랜잭션에 일관된 읽기 뷰를 제공한다. 이를 통해 격리성을 확보한다.
3.  **무결성 검사 (일관성)**: 데이터 수정 과정에서 외래 키, Not Null, Unique 제약 조건을 실시간으로 검증한다. 규칙 위반 시 즉시 에러를 발생시키고 원자성 메커니즘(언두 로그)을 호출하여 작업을 중단한다.
4.  **로그 기록 (영속성 준비)**: 변경된 데이터를 리두 로그 버퍼에 기록한다. 실제 데이터 파일이 수정되기 전, 로그가 먼저 영속화(WAL)되어야만 영속성이 보장된다.
5.  **커밋 또는 롤백**:
    * **커밋 시**: 리두 로그를 디스크로 플러시하여 영속성을 확정한다.
    * **롤백 시**: 언두 로그를 역순으로 적용하여 원자성을 실현하고 모든 변경을 무효화한다.

![](/docs/assets/img/2026-01-09-21-21-32.png)

#### 논리적 심층 분석 및 통찰

> **Q: ACID의 '일관성(Consistency)'과 CAP 이론의 '일관성'은 같은 개념인가?**
> 
> A: 명칭은 같으나 지향점은 다르다. ACID의 일관성은 데이터베이스 스키마에 정의된 규칙(PK, FK 등)을 준수하는 '상태의 유효성'에 집중한다. 즉, 트랜잭션이 끝났을 때 데이터가 모순되지 않아야 함을 뜻한다. 반면, 분산 시스템의 CAP 이론에서 말하는 일관성은 여러 노드에 복제된 데이터가 모든 클라이언트에게 '동일한 값'으로 보이는 '가시성의 동기화'에 집중한다. 따라서 ACID의 일관성은 데이터베이스 엔진 내부의 규칙 엔진에 가깝고, CAP의 일관성은 분산 합의 알고리즘에 가깝다.
{: .block-warning }

#### 기술적 트레이드오프: 격리 수준과 시스템 처리량

격리성(Isolation)은 ACID 중 성능과 가장 직접적으로 상충하는 속성이다.

* **완벽한 격리 (Serializable)**: 데이터 간의 모든 경합을 잠금으로 제어하여 완벽한 안전성을 제공하지만, 동시 처리량이 극도로 낮아져 현대의 고성능 시스템에서는 거의 사용되지 않는다.
* **완화된 격리 (Read Committed 등)**: 일부 현상(Non-repeatable read 등)을 허용하는 대신 MVCC와 같은 비잠금 기술을 활용하여 처리량을 높인다. 이는 ACID 원칙 중 격리성의 '수준'을 조절함으로써 시스템의 실용적 균형을 찾는 공학적 선택이다.

```cpp
/**
 * Storage Engine: Core Transaction Logic (Conceptual)
 * ---------------------------------------------------------
 * This pseudocode illustrates how ACID properties are 
 * physically enforced during a single update operation.
 */

Status TransactionExecutor::execute_update(Row row, UpdateValue val) {
    try {
        // [A] Atomicity: Store before-image to Undo Log
        UndoLog::record_before_image(row);

        // [C] Consistency: Validate Schema Constraints (e.g., NOT NULL, FK)
        if (!ConstraintChecker::validate(row, val)) {
            throw ConsistencyViolationException();
        }

        // [I] Isolation: Acquire locks or manage row versions
        LockManager::acquire_exclusive_lock(row.id);

        // [D] Durability (Stage 1): Record after-image to Redo Buffer
        RedoLog::record_after_image(row.id, val);

        // Perform actual memory update in Buffer Pool
        row.apply_value(val);

        return Status::OK;
    } catch (const Exception& e) {
        // [A] Atomicity: Roll back using Undo Log on failure
        UndoLog::rollback(this->trx_id);
        return Status::FAILED;
    }
}
```

### MySQL 엔진 잠금

**MySQL 엔진 잠금**은 스토리지 엔진(InnoDB, MyISAM 등)의 내부 데이터 잠금과는 별개로, MySQL 서버 계층에서 관리되는 상위 레벨의 잠금 메커니즘이다. 이는 주로 데이터베이스 서버 전체, 특정 테이블의 구조(Schema), 또는 사용자가 정의한 임의의 리소스에 대한 접근을 제어하여 시스템 수준의 정합성을 보장하는 역할을 수행한다.

#### 엔진 수준 잠금의 유형별 특성 대조

MySQL 엔진이 관리하는 잠금은 그 대상과 범위에 따라 다음과 같이 분류된다.

| 잠금 유형 | 대상 범위 | 주요 특징 및 용도 | 획득 방식 |
| :--- | :--- | :--- | :--- |
| **글로벌 락 (Global Lock)** | 서버 전체 | `FLUSH TABLES WITH READ LOCK` 명령으로 획득. 서버의 모든 변경 작업을 멈추고 읽기 전용 상태로 만듦. 주로 전역 일관성 백업에 사용. | 명시적 실행 |
| **테이블 락 (Table Lock)** | 개별 테이블 객체 | 특정 테이블에 대해 읽기(`READ`) 또는 쓰기(`WRITE`) 잠금을 설정. MyISAM 등에서는 DML 수행 시 암묵적으로 발생하나, InnoDB에서는 실효성이 낮아 거의 사용되지 않음. | 명시적(`LOCK TABLES`) 또는 암묵적 |
| **네임드 락 (Named Lock)** | 사용자 정의 문자열 | `GET_LOCK()` 함수를 통해 임의의 문자열에 대한 뮤텍스(Mutex)를 획득. 데이터베이스 객체가 아닌 외부 자원이나 애플리케이션 로직 동기화에 활용. | 명시적 함수 호출 |
| **메타데이터 락 (Metadata Lock, MDL)** | 테이블 스키마 | 테이블의 구조 정보가 변경되는 동안 쿼리 실행을 막거나, 쿼리 실행 중 구조 변경을 막기 위해 자동으로 획득. | 암묵적 자동 획득 |

#### 메타데이터 락(MDL)의 작동 기작과 동시성 제어

이 중 가장 빈번하게 작동하며 시스템에 중대한 영향을 미치는 것은 **메타데이터 락(MDL)**이다. MDL은 사용자가 명시적으로 제어할 수 없으며, SQL이 실행될 때 엔진이 자동으로 획득하고 해제한다.

* **DML 쿼리 시**: `SELECT`, `INSERT`, `UPDATE` 등의 DML 쿼리가 실행되는 동안, MySQL은 해당 테이블의 구조가 변경되지 않도록 **MDL 공유 잠금(Shared Lock)**을 획득한다. 이 잠금은 트랜잭션이 종료될 때까지 유지된다.
* **DDL 쿼리 시**: `ALTER TABLE`, `DROP TABLE` 등의 DDL 쿼리를 실행하려면, 테이블 구조를 안전하게 변경하기 위해 **MDL 배타적 잠금(Exclusive Lock)**이 필요하다.

핵심은 공유 잠금과 배타적 잠금이 서로 양립할 수 없다는 점이다. 따라서 긴 시간이 소요되는 DML 쿼리가 실행 중이라면(공유 잠금 획득), 이후 실행된 DDL 쿼리는 배타적 잠금을 얻지 못해 대기 상태에 빠진다.

![](/docs/assets/img/2026-01-09-21-21-44.png)

#### MDL로 인한 연쇄적 차단(Blocking Chain) 현상 분석

> **Q: 단순히 테이블을 조회하는 `SELECT` 쿼리가 어떻게 전체 서비스 장애를 유발하는 `ALTER TABLE` 차단의 원인이 될 수 있는가?**
> 
> A: 이는 MDL의 대기열 우선순위와 관련이 있다.
> 1.  **세션 A**가 긴 `SELECT` 트랜잭션을 시작하여 **MDL 공유 잠금**을 획득한다.
> 2.  **세션 B**가 `ALTER TABLE`을 시도하여 **MDL 배타적 잠금**을 요청하지만, 세션 A 때문에 대기열에 들어간다.
> 3.  이후 **세션 C, D, E**가 가벼운 `SELECT`나 `INSERT`를 요청하여 **MDL 공유 잠금**을 시도한다.
> 
> 이때 MySQL은 기아 상태(Starvation)를 방지하기 위해, 먼저 대기열에 들어온 배타적 잠금 요청(세션 B)에게 우선순위를 부여한다. 결과적으로 세션 C, D, E는 세션 B가 잠금을 획득하고 작업을 마칠 때까지 모두 대기하게 되며, 세션 A가 종료되지 않는 한 이 대기 체인은 해소되지 않는다. 사소해 보이는 트랜잭션 하나가 거대한 DDL 대기열을 만들고 후속 쿼리를 전면 차단하는 메커니즘이 바로 이것이다.
{: .block-warning }

### InnoDB 스토리지 엔진 잠금

**InnoDB 스토리지 엔진 잠금**은 데이터베이스의 가장 하위 레벨에서 데이터의 무결성과 동시성을 제어하는 핵심 기제이다. MySQL 엔진 잠금이 테이블이나 서버 수준의 거시적 제어를 담당한다면, InnoDB 잠금은 레코드 단위의 미시적 제어를 수행하여 높은 동시 처리 성능을 보장한다. InnoDB 잠금의 가장 큰 특징은 잠금이 실제 데이터 레코드가 아닌 **인덱스 레코드**에 걸린다는 점이다.

#### 잠금의 스코프 및 유형별 특성

InnoDB는 상황에 따라 다양한 범위와 강도의 잠금을 복합적으로 사용한다. 주요 잠금 유형을 아래 표로 정리한다.

| 잠금 유형 | 기술적 명칭 | 적용 대상 및 범위 | 주요 특징 및 목적 |
| :--- | :--- | :--- | :--- |
| **레코드 락** | Record Lock | 인덱스 레코드 자체 (단일 행) | 가장 기본적인 잠금. PK 또는 유니크 인덱스를 통한 정확한 조 시 해당 레코드만 잠금. |
| **갭 락** | Gap Lock | 레코드와 레코드 사이의 간격 (인덱스 갭) | 실제 데이터가 없는 공간을 잠가 새로운 레코드의 삽입을 방지함. 팬텀 리드(Phantom Read) 현상 방지가 주목적. |
| **넥스트 키 락** | Next-Key Lock | 레코드 락 + 해당 레코드 앞의 갭 락 | 레코드 자체와 그 앞의 간격을 함께 잠그는 방식. REPEATABLE READ 격리 수준에서 범위 검색 시 기본으로 사용됨. |
| **자동 증가 락** | Auto-Inc Lock | `AUTO_INCREMENT` 칼럼 | 새 레코드 삽입 시 중복되지 않는 일련번호를 발급하기 위한 테이블 수준의 특수 잠금. 짧은 순간만 유지됨. |

#### 갭 락과 넥스트 키 락의 물리적 메커니즘

갭 락과 넥스트 키 락은 InnoDB가 동시성 제어의 난제인 '팬텀 리드'를 해결하기 위해 고안한 독창적인 메커니즘이다. 이는 "현재 존재하는 것" 뿐만 아니라 "앞으로 존재할 수도 있는 것"까지 잠금의 대상으로 포괄한다.

* **갭 락의 본질**: 갭 락은 데이터를 잠그는 것이 아니라 **'삽입 의도(Insert Intention)'를 제어**하는 장치이다. 예를 들어, `SELECT ... WHERE id > 100 FOR UPDATE` 쿼리가 실행되면, InnoDB는 100보다 큰 레코드들 사이의 모든 간격에 갭 락을 걸어 다른 트랜잭션이 그 사이에 새로운 레코드를 끼워 넣지 못하게 막는다.
* **넥스트 키 락의 작동**: 넥스트 키 락은 'Gap Lock before the record'와 'Record Lock on the record'의 결합이다. 인덱스 값이 10, 20, 30인 상황에서 20에 대한 넥스트 키 락은 `(10, 20]` 범위를 잠근다. 즉, 10 초과 20 미만의 갭과 20이라는 레코드 자체를 동시에 보호한다.

![](/docs/assets/img/2026-01-09-21-21-51.png)

#### 인덱스 의존성 심층 분석

> **Q: 인덱스가 없는 칼럼을 조건으로 `UPDATE` 쿼리를 실행하면 왜 테이블 전체가 잠기는가?**
> 
> A: InnoDB의 모든 잠금은 반드시 **인덱스 레코드**를 대상으로 하기 때문이다.
> 만약 `UPDATE members SET status = 'inactive' WHERE age < 20;` 이라는 쿼리가 있을 때, `age` 칼럼에 인덱스가 없다면 InnoDB는 어떤 레코드를 잠가야 할지 특정할 수 없다. 따라서 스토리지 엔진은 테이블의 모든 레코드를 하나씩 스캔(Full Table Scan)하면서, **클러스터링 인덱스(PK)의 모든 레코드에 대해 넥스트 키 락을 걸어버린다.** 이는 실질적으로 테이블 전체에 배타적 잠금을 거는 것과 동일한 효과를 낳아 극심한 동시성 저하를 초래한다. 올바른 인덱스 설계가 성능뿐 아니라 잠금 범위 제어에도 결정적인 이유가 여기에 있다.
{: .block-warning }

### 격리 수준

**격리 수준(Isolation Level)**은 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있는 허용 범위를 결정하는 설정이다. 이는 ACID 원칙 중 격리성(I)을 물리적으로 구현하는 방식이며, 데이터의 일관성 보장 정도와 시스템의 동시 처리 성능(Concurrency) 사이의 정밀한 트레이드오프를 조절하는 핵심 제어판 역할을 한다.

#### 동시성 부정합 현상과 격리 수준의 상관관계

격리 수준이 낮을수록 더 많은 부정합 현상을 허용하는 대신 높은 성능을 얻고, 수준이 높을수록 엄격한 데이터 보호를 수행한다. 각 수준에서 발생할 수 있는 주요 현상을 다음과 같이 대조한다.

| 격리 수준 | 더티 리드 (Dirty Read) | 비반복 읽기 (Non-repeatable Read) | 팬텀 리드 (Phantom Read) |
| :--- | :---: | :---: | :---: |
| **READ UNCOMMITTED** | 발생 | 발생 | 발생 |
| **READ COMMITTED** | 방지 | 발생 | 발생 |
| **REPEATABLE READ** | 방지 | 방지 | 방지 (InnoDB 한정) |
| **SERIALIZABLE** | 방지 | 방지 | 방지 |

* **더티 리드**: 다른 트랜잭션이 커밋하지 않은 미확정 데이터를 읽는 현상.
* **비반복 읽기**: 하나의 트랜잭션 내에서 같은 조회를 두 번 수행했을 때 그 결과가 다른 현상.
* **팬텀 리드**: 하나의 트랜잭션 내에서 같은 범위 조회를 두 번 수행했을 때, 첫 번째에 없던 '유령(Phantom)' 레코드가 두 번째에 나타나는 현상.

#### 격리 수준별 내부 작동 및 가시성 메커니즘 (Internal Deep-Dive)

각 격리 수준은 내부적으로 MVCC(다중 버전 동시성 제어)와 읽기 뷰(Read View)를 다르게 운용하여 데이터의 가시성을 제어한다.

1.  **READ COMMITTED**: 쿼리가 실행되는 시점마다 새로운 읽기 뷰를 생성한다. 따라서 트랜잭션 도중이라도 다른 트랜잭션이 커밋을 완료하면, 다음 쿼리는 그 최신 데이터를 즉시 보게 된다. 오라클 등 대다수 DBMS의 기본 설정이다.
2.  **REPEATABLE READ**: 트랜잭션이 시작된 후 **첫 번째 읽기 작업 시점에만 읽기 뷰를 생성**하고 트랜잭션 종료 시까지 이를 유지한다. 이후의 모든 조회는 이 스냅샷(언두 로그의 특정 버전)을 참조하므로 항상 동일한 결과를 보장한다. MySQL InnoDB의 기본 격리 수준이다.
3.  **SERIALIZABLE**: 순수 읽기 작업조차 공유 잠금(S-Lock)을 획득해야 하므로, 읽기와 쓰기가 서로를 완벽히 차단한다. 동시성이 극도로 낮아 실무에서는 거의 배제된다.

![](/docs/assets/img/2026-01-09-21-22-06.png)

#### 논리적 추론 및 통찰

> **Q: 왜 타 DBMS와 달리 InnoDB는 REPEATABLE READ를 기본값으로 사용하며, 어떻게 팬텀 리드까지 방어하는가?**
> 
> A: InnoDB의 독특한 **넥스트 키 락(Next-Key Lock)** 아키텍처 덕분이다. 표준적인 REPEATABLE READ는 기존 레코드의 변경만 막을 뿐 새로운 레코드의 삽입을 막지 못해 팬텀 리드가 발생한다. 하지만 InnoDB는 범위 검색 시 해당 범위의 '간격(Gap)'까지 잠그는 넥스트 키 락을 사용하여 새로운 데이터의 삽입 자체를 원천 봉쇄한다. 이를 통해 InnoDB는 REPEATABLE READ 수준에서도 SERIALIZABLE에 근접한 정합성을 확보하면서도, MVCC를 통해 높은 읽기 성능을 유지하는 공학적 이득을 취한다.
{: .block-tip }

#### 기술적 트레이드오프: 일관성 비용과 성능의 임계점

격리 수준의 선택은 시스템 자원의 소모 양상을 결정한다.

* **READ COMMITTED의 효율성**: 읽기 뷰를 쿼리 단위로 짧게 유지하므로 언두 로그의 수명이 짧다. 시스템의 퍼지(Purge) 작업이 원활하여 메모리 압박이 적다.
* **REPEATABLE READ의 위험성**: 트랜잭션이 길어질 경우, 트랜잭션 시작 시점의 스냅샷을 유지하기 위해 엄청난 양의 언두 로그를 보존해야 한다. 이는 버퍼 풀 효율 저하와 디스크 공간 팽창으로 이어지는 '롱 러닝 트랜잭션(Long-running Transaction)' 문제를 야기한다.

```sql
/**
 * InnoDB Isolation Level: REPEATABLE READ Trace
 * ---------------------------------------------------------
 * Proving how InnoDB maintains the same data snapshot 
 * even after concurrent commits.
 */

-- [Session A]
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
-- (1) Initial Read
SELECT balance FROM accounts WHERE id = 101; -- Result: 5000

-- [Session B]
START TRANSACTION;
UPDATE accounts SET balance = 9999 WHERE id = 101;
COMMIT; -- Changes are now globally committed

-- [Session A]
-- (2) Subsequent Read in the same transaction
-- Even though Session B committed, Session A sees the original value 
-- by traversing the Undo Log via its Read View.
SELECT balance FROM accounts WHERE id = 101; -- Result: 5000 (Consistent!)

-- [Engineer's Note]
-- If this were READ COMMITTED, the second result would be 9999.
-- REPEATABLE READ provides stability for long-running analytics.
```