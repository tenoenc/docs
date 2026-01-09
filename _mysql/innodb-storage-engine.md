## InnoDB 스토리지 엔진

### 프라이머리 키 클러스터링

**프라이머리 키 클러스터링**은 InnoDB 스토리지 엔진의 가장 근본적인 저장 아키텍처로, 테이블의 데이터 레코드 자체가 프라이머리 키(PK)의 정렬 순서에 따라 물리적으로 배치되는 구조를 의미한다. 이는 인덱스가 단순히 데이터의 위치를 가리키는 포인터를 가지는 것이 아니라, **인덱스의 리프 노드가 곧 데이터 페이지**가 되는 물리적 일체화를 뜻한다.

#### 클러스터링 인덱스의 물리적 구조와 데이터 배치

InnoDB에서 모든 테이블은 반드시 하나의 클러스터링 인덱스를 가진다. PK를 명시하지 않을 경우 엔진 내부적으로 고유한 행 아이디(Row ID)를 생성하여 클러스터링을 강제한다. 이러한 구조적 특성으로 인해 다음과 같은 물리적 메커니즘이 발생한다.

* **물리적 정렬 유지**: 새로운 레코드가 삽입될 때, PK 값에 따라 물리적으로 저장될 위치가 결정된다. 만약 PK가 무작위한 값(UUID 등)이라면, 기존 데이터 페이지들 사이에 새로운 공간을 만들기 위해 대규모 데이터 이동이 수반되는 '페이지 분할' 현상이 빈번해진다.
* **리프 노드의 실체화**: 일반적인 비클러스터링 인덱스는 리프 노드에 해당 레코드의 PK 값이나 주소를 담고 있지만, 클러스터링 인덱스의 리프 노드에는 행을 구성하는 모든 칼럼 데이터(Actual Row Data)가 직접 저장된다.
* **이중 조회(Double Lookup)의 기점**: 모든 보조 인덱스(Secondary Index)는 실제 데이터 주소가 아닌 PK 값을 저장한다. 따라서 보조 인덱스로 검색할 경우, 보조 인덱스에서 PK를 찾은 뒤 다시 클러스터링 인덱스를 탐색하는 과정을 거친다.

#### 클러스터링 인덱스 기반 데이터 검색 과정 (Step-by-Step Trace)

PK를 통한 단순 조회 쿼리가 실행될 때, 엔진 내부에서는 다음과 같은 경로를 추적한다.

1.  **루트 노드 접근**: 해당 테이블의 클러스터링 B-Tree 루트 노드를 메모리(Buffer Pool)에서 찾는다.
2.  **브랜치 노드 순회**: PK 값을 비교하며 하위 페이지 번호를 추적, 리프 노드 방향으로 수직적 탐색을 수행한다.
3.  **리프 노드(데이터 페이지) 도달**: 목표 PK가 속한 마지막 페이지에 도달한다.
4.  **페이지 내 레코드 식별**: 페이지 내의 슬롯 가이드를 참조하여 실제 레코드 본문 위치를 특정한다.
5.  **데이터 반환**: 해당 위치의 바이너리 데이터를 읽어 MySQL 엔진으로 즉시 반환한다. 별도의 데이터 파일 참조 과정이 없어 I/O 효율이 극대화된다.

```cpp
/**
 * InnoDB Clustered Index: Row Lookup Logic
 * ---------------------------------------------------------
 * This logic shows how the engine finds actual data using the PK.
 */

RowData* find_row_by_primary_key(IndexTree *clustered_tree, PrimaryKey target_pk) {
    // 1. Traverse from Root to Leaf
    Page *current_page = clustered_tree->get_root_page();
    
    while (!current_page->is_leaf()) {
        current_page = current_page->find_next_child_page(target_pk);
    }
    
    // 2. Locate row in the leaf node (Data Page)
    // In Clustered Index, the leaf node *is* the data storage.
    RowData *row = current_page->get_row_by_pk(target_pk);
    
    if (row != NULL) {
        // Direct access to all columns without an additional data file seek.
        return row; 
    }
    
    return NULL; // Not found
}
```

#### 논리적 심층 분석 및 통찰

> **Q: 프라이머리 키가 단순히 '고유성'만 보장하면 충분한가? 아니면 반드시 '순차성'을 가져야 하는가?**
> 
> A: 성능 관점에서 순차성은 필수적이다. 클러스터링 인덱스는 '데이터의 물리적 배치'와 직결되기 때문이다. 만약 PK가 순차적으로 증가하지 않는 값(예: 무작위 문자열)이라면, 데이터는 디스크 곳곳에 파편화되어 저장된다. 이는 인덱스 정렬을 유지하기 위해 기존 페이지를 쪼개고 데이터를 이동시키는 과도한 부하를 주며, 결과적으로 인서트(Insert) 성능의 급격한 저하와 버퍼 풀 효율성 하락을 초래한다. 따라서 클러스터링 아키텍처에서는 '작고 순차적인 값'을 PK로 선정하는 것이 최적화의 핵심이다.
{: .block-warning }

#### 기술적 트레이드오프: 인덱스 검색 속도 vs 쓰기 비용

프라이머리 키 클러스터링은 읽기 최적화를 위해 쓰기 효율의 일부를 희생하는 전략적 선택이다.

* **압도적인 읽기 성능 (Range Scan)**: PK 범위 조회 시, 데이터가 물리적으로 인접해 있어 디스크의 순차 I/O(Sequential I/O)를 최대한 활용할 수 있다. 이는 대규모 범위 스캔에서 비클러스터링 구조 대비 수십 배 이상의 성능 우위를 점하게 한다.
* **비싼 업데이트 및 삽입 비용**: PK 값이 변경되면 해당 레코드를 물리적으로 다른 위치로 옮겨야 하므로 막대한 비용이 든다. 또한, 모든 보조 인덱스가 PK를 참조하고 있기 때문에 PK의 크기가 커질수록 모든 보조 인덱스의 크기가 덩달아 커져 전체적인 메모리 사용량이 증폭되는 부작용이 있다.

### 외래 키 메커니즘

**외래 키(Foreign Key)**는 단순한 데이터 간의 관계 정의를 넘어, 스토리지 엔진 수준에서 데이터의 무결성(Integrity)을 물리적으로 강제하는 장치이다. InnoDB에서 외래 키는 데이터 정합성을 보장하는 강력한 도구인 동시에, 연관된 테이블 간의 쓰기 작업을 동기화함으로써 동시성 제어와 성능 설계에 복합적인 변수를 제공한다.

#### 제약 조건 검사와 잠금 전파의 심층 기작

외래 키가 설정된 테이블에 변경 작업이 발생하면, InnoDB는 연관된 부모 또는 자식 테이블의 상태를 즉각적으로 확인한다. 이 과정은 단순한 조회를 넘어, 데이터의 일관성을 보호하기 위한 물리적 잠금(Lock)의 전파를 수반한다.

* **참조 무결성 검사**: 자식 테이블에 레코드를 삽입하거나 수정할 때, 부모 테이블에 해당 참조 키가 존재하는지 확인한다. 이때 부모 테이블의 해당 레코드에는 **S-Lock(공유 잠금)**이 걸린다. 이는 검사 도중 다른 트랜잭션이 부모 레코드를 삭제하거나 수정하지 못하도록 방어하기 위함이다.
* **연쇄 동작(Cascade) 처리**: 부모 테이블의 레코드가 삭제되거나 수정될 때, 정의된 규칙(`CASCADE`, `SET NULL` 등)에 따라 자식 테이블의 레코드를 자동으로 갱신한다. 이 작업은 내부적으로 재귀적인 쿼리 실행과 동일한 부하를 발생시킨다.
* **메타데이터 의존성**: 외래 키는 테이블 정의의 일부로 관리되므로, 관련 테이블의 구조를 변경(ALTER TABLE)할 때 의존 관계에 있는 모든 테이블에 대한 메타데이터 락이 발생하여 시스템 전체의 DDL 성능에 영향을 줄 수 있다.

#### 외래 키 제약 조건의 단계별 처리 추적 (Step-by-Step Trace)

부모 레코드 삭제 시 `ON DELETE CASCADE` 옵션이 동작하는 내부 과정은 다음과 같다.

1.  **삭제 요청 수신**: 사용자가 부모 테이블에서 특정 PK를 가진 레코드를 삭제하려고 시도한다.
2.  **의존성 트리 스캔**: InnoDB는 데이터 사전(Data Dictionary)을 참조하여 해당 PK를 외래 키로 참조하고 있는 자식 테이블 리스트를 추출한다.
3.  **자식 레코드 잠금**: 자식 테이블에서 해당 부모 키를 가진 모든 레코드에 대해 **X-Lock(배타적 잠금)**을 획득한다.
4.  **전파 삭제 수행**: 자식 레코드들을 물리적으로 삭제하고, 이에 따른 인덱스 갱신 및 언두 로그 기록을 수행한다.
5.  **부모 레코드 삭제**: 모든 자식 레코드의 처리가 완료된 것을 확인한 후, 비로소 부모 레코드를 삭제하고 트랜잭션을 마친다.

#### 논리적 추론 및 통찰

> **Q: 왜 외래 키가 설정된 환경에서 데드락(Deadlock)이 더 빈번하게 발생하는가?**
> 
> A: 외래 키에 의한 **'암묵적 잠금 전파'** 때문이다. 일반적인 쿼리는 명시적으로 조작하는 테이블에만 잠금을 걸지만, 외래 키가 있는 테이블은 부모 레코드에 S-Lock을 걸거나 자식 레코드에 X-Lock을 거는 등 사용자가 의도하지 않은 잠금을 여러 테이블에 동시에 발생시킨다. 두 트랜잭션이 서로 다른 순서로 부모-자식 관계를 교차하여 접근할 경우, 순환 대기 상태에 빠질 확률이 기하급수적으로 높아진다.
{: .block-warning }

#### 기술적 트레이드오프: 데이터 무결성 vs 쓰기 성능

외래 키의 도입은 시스템의 신뢰성과 성능 사이의 전형적인 트레이드오프 사례이다.

* **무결성의 보장**: 애플리케이션 레벨에서 복잡한 체크 로직을 구현하지 않아도 데이터베이스 수준에서 고아 레코드(Orphan Record) 발생을 원천 차단한다. 이는 시스템의 장기적인 데이터 품질 유지에 결정적인 기여를 한다.
* **성능 저하 및 확장성 제약**: 모든 쓰기 작업마다 연관 테이블을 조회하고 잠금을 획득해야 하므로 순수 쓰기 처리량(Throughput)이 감소한다. 특히 대규모 분산 환경이나 샤딩(Sharding)이 필요한 시스템에서는 외래 키가 물리적 서버 경계를 넘나들 수 없으므로 아키텍처 확장에 걸림돌이 되기도 한다.

```sql
/**
 * Foreign Key Constraint Management
 * ---------------------------------------------------------
 * This logic is used for bulk loading where integrity is 
 * pre-verified to maximize performance.
 */

-- 1. Temporarily disable foreign key checks to speed up data loading
SET session foreign_key_checks = 0;

-- 2. Perform high-speed bulk insert/update
-- (During this period, InnoDB does not acquire S-Locks on parent tables)
LOAD DATA INFILE 'large_dataset.csv' INTO TABLE child_table;

-- 3. Re-enable checks (Important: This does not retroactively check data)
SET session foreign_key_checks = 1;

/**
 * [Engineer's Note]
 * Disabling checks skips the internal S-Lock acquisition on parents 
 * and recursive triggers on children, reducing I/O and lock contention.
 * However, the developer must ensure consistency before re-enabling.
 */
```

### MVCC (Multi-Version Concurrency Control)

**MVCC(다중 버전 동시성 제어)**는 동시 접근이 빈번한 데이터베이스 환경에서 잠금(Lock) 사용을 최소화하고 읽기 작업의 일관성을 보장하기 위한 핵심 아키텍처이다. 이는 레코드에 대한 업데이트가 발생할 때 기존 데이터를 즉시 덮어쓰는 대신, 변경 전의 과거 버전을 별도의 공간(언두 로그)에 보관함으로써 하나의 레코드에 대해 시점별로 상이한 여러 버전이 동시에 존재할 수 있게 하는 메커니즘이다.

#### 데이터 버전 관리 및 스토리지 구조의 심층 기작

InnoDB는 MVCC를 구현하기 위해 테이블의 모든 레코드에 사용자에게 보이지 않는 내부 시스템 칼럼을 추가하여 관리한다.

* **DB_TRX_ID (트랜잭션 ID)**: 해당 레코드를 마지막으로 삽입하거나 수정한 트랜잭션의 고유 식별자(6바이트)가 저장된다. 삭제 작업 역시 내부적으로는 '삭제 플래그'를 업데이트하는 수정 작업으로 간주되어 이 ID가 갱신된다.
* **DB_ROLL_PTR (롤백 포인터)**: 언두 로그(Undo Log) 영역에 저장된 이전 버전의 레코드를 가리키는 물리적 포인터(7바이트)이다. 이 포인터를 통해 현재 레코드에서 과거 버전으로 거슬러 올라가는 연결 리스트(Version Chain)가 형성된다.

레코드가 업데이트되면, InnoDB는 현재 버전을 언두 로그로 복사한 후, 실제 데이터 페이지의 레코드를 새 값으로 갱신하고 `DB_ROLL_PTR`를 언두 로그의 이전 버전을 가리키도록 연결한다. 이 과정이 반복되면 언두 로그에는 시간 역순으로 데이터의 변경 이력이 쌓이게 된다.

![](/docs/assets/img/2026-01-09-21-07-12.png)

#### 읽기 뷰(Read View)를 통한 가시성 판단 추적 (Step-by-Step Trace)

트랜잭션이 `SELECT` 쿼리를 수행할 때 어떤 버전을 읽어야 할지 결정하는 과정은 **읽기 뷰(Read View)**라는 논리적 스냅샷을 통해 이루어진다.

1.  **읽기 뷰 생성**: 트랜잭션이 시작되거나 첫 읽기 작업이 수행되는 시점에, 현재 시스템에서 활성화 상태(커밋되지 않음)인 모든 트랜잭션의 ID 목록을 캡처하여 읽기 뷰를 생성한다.
2.  **레코드 접근 및 트랜잭션 ID 검사**: 데이터 페이지에서 레코드를 읽을 때, 레코드의 `DB_TRX_ID`를 확인한다.
3.  **가시성 판단 알고리즘 적용**:
    * 레코드의 `DB_TRX_ID`가 읽기 뷰 생성 시점보다 이전에 커밋된 트랜잭션이라면 → 현재 레코드를 그대로 읽는다 (가시성 확보).
    * 레코드의 `DB_TRX_ID`가 읽기 뷰 생성 당시 활성 상태였거나, 읽기 뷰 생성 이후에 시작된 트랜잭션이라면 → 현재 레코드는 미래의 데이터이므로 볼 수 없다.
4.  **언두 로그 탐색**: 현재 레코드가 보이지 않을 경우, `DB_ROLL_PTR`를 따라 언두 로그의 이전 버전으로 이동한다. 이전 버전의 헤더에 있는 `DB_TRX_ID`에 대해 다시 3번의 가시성 판단을 수행한다. 이 과정을 볼 수 있는 버전이 나올 때까지 반복한다.

#### 언두 로그의 수명 주기와 시스템 부하

> **Q: 끊임없이 생성되는 과거 버전의 데이터(언두 로그)는 언제까지 유지되는가? 디스크 공간을 무한히 점유하는가?**
> 
> A: 그렇지 않다. 언두 로그는 해당 버전을 필요로 하는 가장 오래된 활성 트랜잭션(Read View)이 존재하는 동안에만 유지된다. 시스템 내부의 **퍼지 스레드(Purge Thread)**는 주기적으로 더 이상 어떤 트랜잭션에서도 참조하지 않는(가시성 범위 밖으로 밀려난) 오래된 언두 로그 레코드를 감지하여 물리적으로 삭제한다. 만약 장시간 커밋되지 않는 트랜잭션(Long Running Transaction)이 존재하면, 퍼지 작업이 지연되어 언두 공간이 급격히 증가하고 전체 시스템 성능이 저하될 수 있다.
{: .block-warning }

#### 기술적 트레이드오프: 동시성 확보 vs 추가 비용

MVCC는 '쓰기 작업이 읽기 작업을 블로킹하지 않고, 읽기 작업이 쓰기 작업을 블로킹하지 않는다'는 대원칙을 실현하여 동시 처리량을 극대화하지만, 이에 수반되는 명확한 비용이 존재한다.

* **저장 공간 오버헤드**: 모든 변경 이력을 언두 로그에 저장해야 하므로 추가적인 디스크 및 메모리 공간이 소모된다.
* **CPU 연산 비용**: 데이터를 읽을 때마다 단순히 현재 값을 가져오는 것이 아니라, 버전 체인을 거슬러 올라가며 가시성을 판단하는 복잡한 연산 과정이 매 레코드마다 수행되어야 한다. 이는 특히 변경이 빈번한 레코드를 조회할 때 CPU 점유율을 높이는 원인이 된다.

### 잠금 없는 일관된 읽기

**잠금 없는 일관된 읽기(Consistent Non-locking Read)**는 MVCC 메커니즘을 활용하여 읽기 작업이 수행될 때 다른 트랜잭션이 해당 레코드에 대해 점유 중인 잠금을 대기하지 않고 즉시 응답하는 기술이다. 이는 특정 시점의 데이터 스냅샷을 읽음으로써, 쓰기 작업이 진행 중인 환경에서도 읽기 성능을 저하하지 않고 데이터 정합성을 유지하는 InnoDB의 핵심 동시성 제어 전략이다.

#### 언두 로그를 활용한 가시성 확보 메커니즘

InnoDB에서 읽기 전용 쿼리(`SELECT`)는 대상 레코드에 대해 공유 잠금(S-Lock)을 획득하지 않는다. 대신, 읽기 뷰(Read View)를 기준으로 해당 트랜잭션이 보기에 유효한 가장 최신의 과거 버전을 선택한다.

1.  **배타적 잠금(X-Lock)의 무시**: 특정 레코드가 다른 트랜잭션에 의해 수정(Update) 중이어서 배타적 잠금이 걸려 있더라도, 읽기 작업은 해당 잠금의 해제를 기다리지 않는다.
2.  **버전 체인 추적**: 현재 데이터 페이지에 있는 레코드가 수정 중이거나 읽기 뷰 시점보다 미래의 데이터라면, 레코드 헤더의 `DB_ROLL_PTR`를 참조하여 언두 로그 영역에 보관된 이전 버전으로 이동한다.
3.  **일관성 보장**: 언두 로그를 통해 복구된 데이터는 쿼리 또는 트랜잭션이 시작된 시점의 상태를 완벽하게 재현하므로, 중간에 다른 트랜잭션이 커밋되더라도 읽기 작업 내에서의 데이터 일관성은 파괴되지 않는다.

#### 격리 수준에 따른 읽기 뷰의 유효성 변화 (Step-by-Step Trace)

일관된 읽기가 수행되는 구체적인 시점은 설정된 트랜잭션 격리 수준에 따라 결정적인 차이를 보인다.

1.  **READ COMMITTED**:
    * 하나의 트랜잭션 내에서도 `SELECT` 쿼리가 실행될 때마다 새로운 읽기 뷰를 생성한다.
    * 따라서 쿼리 실행 사이에 다른 트랜잭션이 커밋한 데이터가 있다면, 다음 `SELECT` 시에는 그 변경 사항이 반영된 최신 데이터를 보게 된다.
2.  **REPEATABLE READ**:
    * 트랜잭션이 시작된 후 첫 번째 읽기 작업이 수행되는 시점에 단 한 번만 읽기 뷰를 생성한다.
    * 이후 해당 트랜잭션 내에서 발생하는 모든 `SELECT` 쿼리는 동일한 읽기 뷰를 재사용하므로, 다른 트랜잭션이 수천 번 커밋하더라도 항상 트랜잭션 시작 시점의 데이터만 보게 된다.
3.  **SERIALIZABLE**:
    * 일관된 읽기가 적용되지 않는다. 모든 읽기 작업은 공유 잠금(S-Lock)을 획득해야 하며, 쓰기 작업과 상호 배타적으로 동작한다.

#### 논리적 추론 및 통찰

> **Q: 잠금을 사용하지 않는다면 데이터가 항상 최신임을 보장할 수 있는가?**
> 
> A: 보장할 수 없다. 일관된 읽기는 '최신성'보다 '일관성'에 집중하는 모델이다. 만약 다른 트랜잭션이 데이터를 수정한 직후 커밋했더라도, 현재 트랜잭션의 읽기 뷰가 그 이전 시점이라면 현재 트랜잭션은 수정 전의 데이터를 읽게 된다. 만약 반드시 가장 최신의 데이터를 읽어야 하는 'Strong Consistency'가 필요하다면, `SELECT ... FOR SHARE`나 `SELECT ... FOR UPDATE`와 같이 명시적인 잠금 읽기(Locking Read)를 사용하여 일관된 읽기 메커니즘을 의도적으로 우회해야 한다.
{: .block-warning }

#### 기술적 트레이드오프: 퍼지 지연(Purge Lag)과 시스템 부하

잠금 없는 일관된 읽기는 동시성을 극대화하는 대신, 시스템 자원의 장기 점유라는 잠재적 위험을 내포한다.

* **동시 처리량 향상**: 읽기와 쓰기가 서로를 방해하지 않으므로, 읽기 위주의 서비스(웹 서비스 등)에서 폭발적인 성능 향상을 가져온다.
* **언두 로그 누적 위험**: 특정 트랜잭션이 아주 오래 유지(Long-running Transaction)되면서 일관된 읽기를 수행할 경우, 해당 트랜잭션이 시작된 시점 이후의 모든 언두 로그는 물리적으로 삭제될 수 없다. 이는 언두 테이블스페이스의 급격한 팽창을 유발하며, 퍼지 스레드(Purge Thread)가 처리해야 할 작업량을 늘려 전체적인 시스템 응답성을 떨어뜨리는 원인이 된다.

```sql
/**
 * Consistent Read Demonstration
 * ---------------------------------------------------------
 * This SQL script shows how Read Views behave differently 
 * under different isolation levels.
 */

-- [Session A: Setting initial state]
UPDATE account SET balance = 1000 WHERE id = 1;
COMMIT;

-- [Session B: REPEATABLE READ]
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- Result: 1000 (Read View created)

-- [Session A: Concurrent Update]
UPDATE account SET balance = 2000 WHERE id = 1;
COMMIT; -- Version 1000 is moved to Undo Log

-- [Session B: Subsequent Read]
-- Still sees 1000 because it reuses the Read View created at the start.
SELECT balance FROM account WHERE id = 1; -- Result: 1000

-- [Session C: READ COMMITTED]
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
START TRANSACTION;
SELECT balance FROM account WHERE id = 1; -- Result: 2000 (New Read View)

-- [Engineer's Note]
-- REPEATABLE READ ensures that the data doesn't change during 
-- the life of the transaction, which is crucial for consistency in 
-- long-running reports.
```

### 자동 데드락 감지

**자동 데드락 감지(Automatic Deadlock Detection)**는 두 개 이상의 트랜잭션이 서로가 가진 잠금을 무한히 대기하는 교착 상태(Deadlock)를 시스템이 스스로 인지하고 해결하는 방어 메커니즘이다. InnoDB 스토리지 엔진은 내부적으로 **대기 그래프(Wait-for Graph)**를 유지 관리하며, 이를 실시간으로 탐색하여 순환 대기(Circular Wait) 구조가 발생했는지를 추적한다.

#### 대기 그래프 탐색 및 해결 프로세스

데드락 감지 스레드는 잠금 대기 요청이 발생할 때마다 그래프를 스캔하여 교착 상태 여부를 판단한다.

1.  **대기 요청 등록**: 트랜잭션이 특정 레코드에 대해 잠금을 요청했으나 다른 트랜잭션이 점유 중인 경우, 해당 트랜잭션은 잠금 대기 큐에 등록된다.
2.  **그래프 탐색**: 데드락 감지기가 활성화되어 대기 그래프를 순회한다. 이때 "트랜잭션 A가 B를 기다리고, B가 다시 A를 기다리는" 식의 순환 고리가 있는지 검사한다.
3.  **희생자(Victim) 선정**: 데드락이 확인되면 시스템은 교착 상태를 끊기 위해 참여 트랜잭션 중 하나를 강제로 종료(Rollback)한다.
4.  **최소 비용 원칙**: 희생자를 선택할 때는 **언두 로그(Undo Log)의 양**을 기준으로 삼는다. 언두 로그가 가장 적게 쌓인 트랜잭션, 즉 롤백 시 물리적 복구 비용이 가장 적게 드는 트랜잭션이 우선적으로 희생된다.

![](/docs/assets/img/2026-01-09-21-07-28.png)

#### 데드락 제어 방식의 공학적 대조

시스템은 데드락 문제를 해결하기 위해 적극적인 감지 방식과 소극적인 타임아웃 방식을 병행하거나 선택적으로 사용할 수 있다.

| 비교 속성 | 자동 데드락 감지 (Deadlock Detection) | 잠금 대기 타임아웃 (Lock Wait Timeout) |
| :--- | :--- | :--- |
| **작동 원리** | 대기 그래프를 상시 분석하여 즉각 대응 | 설정된 시간 동안 대기 후 자동 실패 처리 |
| **반응 속도** | 교착 상태 확인 즉시 해결 (매우 빠름) | 타임아웃 시간 경과 후 해결 (비교적 느림) |
| **시스템 부하** | 대기 트랜잭션이 많을수록 탐색 CPU 부하 증가 | 부하가 거의 없음 (타이머 기반) |
| **해결 정확도** | 실제 데드락만 정확히 집어내어 해결 | 데드락이 아니어도 대기가 길어지면 실패 |
| **권장 환경** | 일반적인 동시성 제어가 필요한 환경 | 초고부하 트래픽으로 인한 잠금 경합 극심 지역 |

#### 데드락 감지 메커니즘의 성능 임계점

> **Q: 데드락 감지 기능이 항상 유익한가? 왜 대규모 트래픽 환경에서 이 기능을 끄는 경우가 발생하는가?**
> 
> A: 데드락 감지기가 그래프를 탐색하는 동안에는 잠금 상태가 변하지 않도록 **잠금 목록(Lock List) 전체에 대한 뮤텍스(Mutex)**를 획득해야 하기 때문이다. 특정 레코드에 수천 개의 스레드가 동시에 잠금을 요청하는 상황(Hotspot)이 발생하면, 데드락 감지 스레드가 매번 거대한 대기 리스트를 뒤지는 과정에서 CPU를 과도하게 점유하게 된다. 이로 인해 실제 쿼리를 처리하는 스레드들까지 대기 상태에 빠지는 '성능의 역설'이 발생할 수 있다. 이 경우 감지 기능을 끄고 타임아웃(`innodb_lock_wait_timeout`) 수치를 낮게 설정하는 것이 유리할 수 있다.
{: .block-warning }

#### 데드락 발생 시나리오 추적 (Step-by-Step Trace)

다음은 두 트랜잭션이 데드락으로 치닫는 물리적 진행 단계이다.

1.  **T1**: 레코드 1에 대해 X-Lock을 획득한다.
2.  **T2**: 레코드 2에 대해 X-Lock을 획득한다.
3.  **T1**: 레코드 2에 대해 X-Lock을 요청한다. T2가 점유 중이므로 대기 상태에 진입한다.
4.  **T2**: 레코드 1에 대해 X-Lock을 요청한다. T1이 점유 중이므로 대기 상태에 진입한다.
5.  **감지기 동작**: 데드락 감지 스레드가 T1 → T2 → T1으로 이어지는 순환 고리를 발견한다.
6.  **비용 비교**: T1은 10건의 수정을 수행했고(Undo Log 10), T2는 2건의 수정만 수행했다(Undo Log 2).
7.  **강제 종료**: 언두 로그가 적은 T2를 희생자로 결정하고 롤백시킨다. T1은 잠금을 획득하고 작업을 계속한다.

```cpp
/**
 * Deadlock Detector: Cycle Search in Wait-for Graph
 * ---------------------------------------------------------
 * Simplified representation of the recursive search for deadlocks.
 */

bool find_deadlock_cycle(Transaction *current_trx, Set<Transaction*> &visited) {
    // If we encounter a transaction already visited in this search path, it's a cycle.
    if (visited.find(current_trx) != visited.end()) {
        return true; 
    }

    visited.insert(current_trx);

    // Get the transaction that current_trx is waiting for
    Transaction *wait_for_trx = current_trx->get_blocking_transaction();

    if (wait_for_trx != NULL) {
        // Recursive depth search
        if (find_deadlock_cycle(wait_for_trx, visited)) {
            return true;
        }
    }

    // Path cleared, remove from visited for other branches
    visited.erase(current_trx);
    return false;
}

void handle_deadlock_detection(Transaction *waiting_trx) {
    Set<Transaction*> search_path;
    
    if (find_deadlock_cycle(waiting_trx, search_path)) {
        // Cycle detected! Select the victim with the smallest undo log count.
        Transaction *victim = select_victim_with_least_undo(search_path);
        rollback_transaction(victim);
        log_info("Deadlock resolved by rolling back TRX_ID: %d", victim->id);
    }
}
```

### InnoDB 버퍼 풀

**InnoDB 버퍼 풀(Buffer Pool)**은 데이터베이스의 디스크 I/O 부하를 획기적으로 줄이기 위해 데이터와 인덱스 페이지를 메모리에 캐싱하고 수정 작업을 버퍼링하는 거대한 메모리 저장소이다. 이는 단순한 캐시 계층을 넘어, 쓰기 지연(Write-behind)을 통한 I/O 병합과 데이터 일관성 보장을 위한 물리적 연산의 중심지 역할을 수행한다.

#### 메모리 페이지 관리 아키텍처

버퍼 풀은 내부적으로 고정된 크기(기본 16KB)의 페이지 단위로 분할되어 관리되며, 효율적인 자원 운용을 위해 세 가지 핵심 리스트 구조를 유지한다.

* **Free List**: 실제 데이터가 채워지지 않은 비어 있는 페이지들의 목록이다. 새로운 데이터를 디스크에서 읽어올 때 이 리스트에서 페이지를 할당받는다.
* **LRU (Least Recently Used) List**: 최근에 사용된 페이지와 그렇지 않은 페이지를 구분하여 관리하는 목록이다. 메모리가 부족할 때 어떤 페이지를 제거할지 결정하는 기준이 된다.
* **Flush List**: 수정되었으나 아직 디스크로 기록되지 않은 '더티 페이지(Dirty Page)'들을 관리한다. 리두 로그의 체크포인트와 연계되어 물리적 쓰기 시점을 결정한다.

#### LRU 리스트의 중간 지점 삽입 전략 (Step-by-Step Trace)

일반적인 LRU 알고리즘과 달리, InnoDB는 풀 테이블 스캔으로 인해 자주 사용되는 데이터(Hot Page)가 메모리에서 밀려나는 것을 방지하기 위해 '미드포인트(Midpoint)' 삽입 방식을 사용한다.

1.  **신규 데이터 유입**: 디스크에서 읽어온 페이지를 리루 리스트의 최상단이 아닌, 전체의 약 3/8 지점(Old Sublist의 헤드)에 삽입한다.
2.  **검증 단계**: 해당 페이지가 즉시 다시 참조되는지 감시한다.
3.  **New Sublist 승격**: 일정 시간(innodb_old_blocks_time) 내에 다시 참조될 경우, 비로소 '자주 사용되는 데이터'로 판단하여 New Sublist의 헤드로 이동시킨다.
4.  **에이징(Aging) 및 제거**: 참조되지 않은 페이지는 점차 리스트의 꼬리(Tail) 방향으로 밀려나며, 결국 버퍼 풀에서 제거되어 Free List로 반납된다.

![](/docs/assets/img/2026-01-09-21-07-34.png)

#### 버퍼 풀의 성능 동학에 관한 통찰

> **Q: 버퍼 풀의 크기를 물리 메모리의 80% 이상으로 설정하는 것이 항상 최선인가?**
> 
> A: 위험한 접근이다. 버퍼 풀이 너무 크면 운영체제 자체가 가용 메모리 부족으로 인해 스왑(Swap) 영역을 사용하게 될 수 있으며, 이는 메모리 접근 속도를 디스크 수준으로 떨어뜨리는 결과(Swapping)를 초래한다. 또한, 버퍼 풀은 단일 뮤텍스(Mutex) 경합을 줄이기 위해 여러 개의 **버퍼 풀 인스턴스**로 분할 관리되는데, 인스턴스당 크기가 너무 작아지면 관리 오버헤드가 증가한다. 시스템의 전체 워크로드와 OS 커널이 점유할 최소 메모리를 계산하여 정밀하게 설계해야 한다.
{: .block-warning }

#### 기술적 트레이드오프: 쓰기 버퍼링과 데이터 유실 위험

버퍼 풀은 데이터 변경을 메모리에서 먼저 수행한 뒤 나중에 디스크에 기록(Flush)함으로써 성능을 얻지만, 이는 필연적으로 안정성과의 저울질을 수반한다.

* **성능 이점**: 여러 번의 수정 작업을 모아서 한 번의 디스크 쓰기로 처리(I/O Batching)함으로써 랜덤 I/O 횟수를 획기적으로 줄인다.
* **복구 복잡성**: 서버가 비정상 종료될 경우, 디스크에 반영되지 않은 버퍼 풀 내의 더티 페이지는 소실된다. 이를 보완하기 위해 모든 변경 사항은 반드시 **리두 로그(Redo Log)**에 먼저 기록(Write-Ahead Logging)되어야 하며, 시스템 재시작 시 리두 로그를 통해 버퍼 풀의 상태를 재건하는 '복구 공정'이 필수적으로 요구된다.

```cpp
/**
 * InnoDB Buffer Pool: Page Retrieval and LRU Update
 * ---------------------------------------------------------
 * Illustrates the logic of finding a page and managing its 
 * position within the sublists.
 */

Page* get_page_from_buffer_pool(PageID id) {
    // 1. Check if the page is already in the buffer pool (Hash Index)
    Page *page = buffer_pool_hash.find(id);

    if (page) {
        // Page found (Cache Hit)
        if (page->is_in_old_sublist()) {
            // Check if it's been long enough since insertion to be promoted
            if (current_time() - page->first_access_time > innodb_old_blocks_time) {
                move_to_new_sublist_head(page);
            }
        } else {
            // Already in New Sublist, just move to head (Make it young)
            move_to_new_sublist_head(page);
        }
        return page;
    }

    // 2. Cache Miss: Read from disk
    page = allocate_free_page(); // Get from Free List
    if (!page) {
        page = evict_lru_tail(); // If no free page, evict the oldest
    }
    
    read_from_disk(id, page);
    
    // 3. Midpoint Insertion
    insert_at_old_sublist_head(page);
    page->first_access_time = current_time();
    
    return page;
}
```

### Double Write Buffer

**Double Write Buffer**는 데이터 페이지를 디스크에 기록하는 과정에서 발생할 수 있는 '부분 쓰기(Partial Write)' 혹은 '페이지 파손(Torn Page)' 문제를 원천적으로 방지하기 위한 안전 장치이다. 하드웨어나 운영체제의 결함으로 인해 16KB 크기의 InnoDB 페이지가 디스크에 온전히 기록되지 못하고 일부만 기록되는 현상이 발생했을 때, 이를 복구할 수 있는 원본 데이터를 보존하는 역할을 수행한다.

#### 파손된 페이지와 복구의 한계

InnoDB의 리두 로그(Redo Log)는 데이터의 변경 이력을 저장하지만, 데이터 페이지 자체가 물리적으로 깨진 경우에는 무용지물이 된다.

* **Torn Page 현상**: InnoDB의 기본 페이지 크기는 16KB인 반면, 일반적인 OS나 HDD/SSD의 최소 기입 단위는 512Byte 또는 4KB이다. 전원 공급 중단 등의 장애가 발생하면 16KB 중 일부 블록만 디스크에 써질 수 있다.
* **체크섬 오류**: 페이지의 일부만 기록되면 페이지 내부의 체크섬 값이 불일치하게 된다. 리두 로그는 "어떤 페이지의 몇 번 오프셋을 수정하라"는 정보만 갖고 있으므로, 베이스가 되는 페이지 자체가 깨져 있으면 복구를 시도할 수 없다.

#### 데이터 기입의 이중화 프로세스 (Step-by-Step Trace)

버퍼 풀의 더티 페이지가 실제 데이터 파일로 이동하는 과정은 다음과 같은 원자적 단계를 거친다.

1.  **더티 페이지 취합**: 버퍼 풀에서 디스크로 기록해야 할 더티 페이지들을 모은다.
2.  **Double Write Buffer 기록**: 취합된 페이지들을 시스템 테이블스페이스 내의 **Double Write Buffer** 영역에 순차 I/O(Sequential I/O)로 한꺼번에 기록한다.
3.  **저장소 동기화 (fsync)**: Double Write Buffer에 데이터가 안전하게 기록되었음을 보장하기 위해 운영체제에 동기화 명령을 내린다.
4.  **실제 데이터 파일 기록**: 이제 각 페이지를 데이터 파일(.ibd) 내의 본래 위치에 무작위 I/O(Random I/O)로 기입한다.
5.  **완료 및 체크포인트**: 데이터 파일 기록이 완료되면 Double Write Buffer의 내용은 더 이상 필요하지 않게 된다.

![](/docs/assets/img/2026-01-09-21-07-45.png)

#### 안정성과 성능의 기술적 대조

Double Write Buffer의 사용 여부에 따른 시스템의 운영적 특성을 분석한다.

| 비교 속성 | Double Write Buffer 활성화 | Double Write Buffer 비활성화 |
| :--- | :--- | :--- |
| **데이터 안정성** | 비정상 종료 시 페이지 파손 완벽 복구 | 페이지 파손 시 수동 복구 또는 데이터 유실 위험 |
| **I/O 부하** | 동일 데이터를 두 번 쓰므로 I/O 총량 증가 | 한 번만 기록하므로 상대적으로 I/O 부하 적음 |
| **기록 방식** | 순차 쓰기(Buffer) + 무작위 쓰기(Data) | 오직 무작위 쓰기(Data) |
| **SSD 수명** | 쓰기 증폭(Write Amplification)으로 수명 단축 가능 | 쓰기 횟수가 적어 수명 관리에 유리함 |
| **권장 환경** | 일반적인 모든 엔터프라이즈 환경 | ZFS와 같이 파일시스템 자체적으로 Torn Page를 방지하는 경우 |

#### 논리적 추론 및 통찰

> **Q: 데이터를 두 번 쓰면 시스템 성능이 절반으로 떨어지는 것이 아닌가?**
> 
> A: 그렇지 않다. Double Write Buffer에 기록하는 과정은 '순차 I/O'로 처리되며, 여러 페이지를 묶어서 한 번의 시스템 콜로 기록하기 때문에 발생하는 부하가 생각보다 크지 않다. 실제 병목은 데이터 파일의 제자리(In-place)를 찾아가는 '무작위 I/O'에서 발생한다. 따라서 데이터의 무결성을 확보하기 위해 지불하는 비용 대비 얻는 안정성이 압도적으로 높다. 다만, 대규모 쓰기 부하가 발생하는 환경에서 SSD의 쓰기 수명을 극도로 아껴야 하거나, 파일시스템 레벨에서 원자적 기입을 보장한다면 이 기능을 끄는 선택을 고려할 수 있다.
{: .block-warning }

#### 장애 복구 시나리오 분석 (Internal Deep-Dive)

서버 재시작 시 InnoDB는 데이터 페이지를 읽으며 체크섬을 검사한다. 만약 체크섬이 맞지 않는 페이지를 발견하면 다음과 같은 복구 로직을 가동한다.

* 해당 페이지가 Double Write Buffer에 존재하는지 확인한다.
* Double Write Buffer에 있는 깨끗한 원본 페이지를 가져와서 파손된 데이터 파일의 페이지에 덮어쓴다.
* 정상으로 돌아온 페이지 위에 리두 로그를 적용(Replay)하여 최신 상태로 복구한다.

### 언두 로그

**언두 로그(Undo Log)**는 트랜잭션의 원자성(Atomicity)과 격리성(Isolation)을 물리적으로 뒷받침하는 데이터 변경 이력 저장소이다. 트랜잭션이 데이터를 변경했을 때 변경 전의 데이터를 보관하여, 장애 시 작업을 취소(Rollback)하거나 다른 트랜잭션에 특정 시점의 일관된 데이터 버전(MVCC)을 제공하는 핵심 자원으로 기능한다.

#### 언두 로그의 생성 및 연결 메커니즘 (Step-by-Step Trace)

하나의 레코드가 수정될 때 언두 로그가 생성되고 데이터와 연결되는 과정은 다음과 같은 정밀한 단계로 진행된다.

1.  **트랜잭션 시작**: 고유한 트랜잭션 ID(TRX_ID)를 할당받고 언두 세그먼트(Undo Segment) 내에 슬롯을 확보한다.
2.  **변경 전 데이터 복사**: 실제 데이터 페이지를 수정하기 직전, 기존 레코드의 값을 언두 로그 레코드로 복사한다.
3.  **데이터 페이지 갱신**: 데이터 페이지의 레코드를 새 값으로 업데이트하고, 시스템 칼럼인 `DB_TRX_ID`를 현재 트랜잭션 ID로 설정한다.
4.  **포인터 연결**: 레코드의 `DB_ROLL_PTR`가 방금 생성한 언두 로그의 위치를 가리키도록 설정한다. 이로써 현재 레코드에서 과거 버전으로 거슬러 올라가는 **버전 체인(Version Chain)**이 형성된다.
5.  **연쇄적 이력 관리**: 동일 레코드가 다시 수정되면, 새로운 언두 로그는 이전의 언두 로그를 가리키게 되어 긴 이력 사슬을 구성한다.

#### 데이터 복구와 정합성을 위한 로그 대조 분석

데이터베이스 시스템에는 변경을 기록하는 두 가지 핵심 로그가 존재한다. 언두 로그와 리두 로그의 공학적 차이를 분석한다.

| 비교 속성 | 언두 로그 (Undo Log) | 리두 로그 (Redo Log) |
| :--- | :--- | :--- |
| **핵심 목적** | 트랜잭션 롤백 및 MVCC(읽기 일관성) | 시스템 장애 시 데이터 유실 방지(영속성) |
| **기록 데이터** | 변경 전의 이전 값 (Before Image) | 변경 후의 새로운 값 (After Image) |
| **영향 범위** | 논리적 트랜잭션 단위 | 물리적 페이지/블록 단위 |
| **기록 시점** | 데이터 수정 직전 | 데이터 수정 시 (WAL 방식) |
| **수명 주기** | 참조하는 읽기 뷰가 없을 때까지 유지 | 디스크 기입 완료(체크포인트) 시 재사용 |

#### 언두 로그 관리의 논리적 임계점

> **Q: 트랜잭션이 커밋되었는데 왜 언두 로그가 즉시 삭제되지 않고 남아서 디스크를 점유하는가?**
> 
> A: 트랜잭션의 '완료'와 언두 로그의 '소멸'은 별개의 사건이기 때문이다. 커밋된 트랜잭션의 언두 로그라 할지라도, 해당 트랜잭션이 시작된 이후에 실행된 **다른 읽기 트랜잭션**이 있다면 일관된 읽기를 위해 해당 과거 버전이 여전히 필요할 수 있다. 시스템의 퍼지 스레드(Purge Thread)는 모든 읽기 뷰(Read View)가 해당 언두 로그보다 미래의 시점으로 이동했을 때에만 안전하게 이를 물리적으로 삭제한다. 따라서 긴 시간을 점유하는 트랜잭션이 하나라도 있다면 언두 로그는 기하급수적으로 쌓이게 된다.
{: .block-warning }

#### 기술적 트레이드오프: 롤백 세그먼트 확장성과 동시성

언두 로그를 관리하는 **롤백 세그먼트(Rollback Segment)**의 설계는 시스템의 동시 처리 능력에 직접적인 영향을 미친다.

* **세그먼트 분할**: InnoDB는 동시 트랜잭션 간의 경합을 줄이기 위해 언두 로그 영역을 여러 개의 롤백 세그먼트로 분할하여 관리한다. 이는 다중 CPU 코어 환경에서 언두 로그 기록 시 발생하는 뮤텍스(Mutex) 경합을 분산시킨다.
* **성능 제약**: 그러나 롤백 세그먼트의 전체 개수와 슬롯 수는 물리적으로 제한되어 있다. 극도로 많은 동시 트랜잭션이 대규모 데이터 변경을 수행할 경우 '언두 슬롯 부족' 에러가 발생할 수 있으며, 이는 시스템이 수용할 수 있는 동시 쓰기 작업의 절대적인 상한선으로 작용한다.

```cpp
/**
 * InnoDB Undo Log: Record Header and Payload
 * ---------------------------------------------------------
 * This represents how an undo log stores previous data 
 * to allow reconstruction of old versions.
 */

struct undo_log_record {
    // 1. Metadata for identification
    uint64_t undo_no;         // Sequential number of undo log in a transaction
    uint64_t trx_id;          // ID of the transaction that created this log
    uint32_t table_id;        // Target table identifier

    // 2. Navigation
    uint64_t roll_ptr_prev;   // Pointer to the previous undo log of the same row

    // 3. Payload: The "Before Image"
    // Stores only the changed columns for UPDATE, or the full row for DELETE
    byte* old_values;         
    uint32_t payload_len;

    // 4. Type of Operation
    enum undo_type {
        UNDO_INSERT,          // Only stores the PK (for simple delete on rollback)
        UNDO_UPDATE_EXISTING, // Stores old values of modified columns
        UNDO_DELETE_MARK      // Stores full values before marking for deletion
    } type;
};
```

### 리두 로그

**리두 로그(Redo Log)**는 시스템 장애 발생 시 데이터의 영속성(Durability)을 보장하기 위한 물리적 기록 장치이다. InnoDB는 성능을 위해 데이터 변경을 즉시 디스크의 데이터 파일(.ibd)에 기록하지 않고 메모리(Buffer Pool)에서 처리하는데, 이 과정에서 발생할 수 있는 데이터 유실을 방지하기 위해 변경 사항을 순차적인 로그 형태로 기록하는 **WAL(Write-Ahead Logging)** 메커니즘을 수행한다.

#### 리두 로그의 순환 구조와 LSN 관리

리두 로그는 무한히 커지는 파일이 아니라, 고정된 크기의 파일 여러 개를 연결하여 마지막 파일의 끝에 도달하면 다시 첫 번째 파일로 돌아와 덮어쓰는 **순환 구조(Circular Buffer)**로 운영된다. 이 구조를 효율적으로 관리하기 위해 시스템은 다음과 같은 핵심 지표를 사용한다.

* **LSN (Log Sequence Number)**: 리두 로그에 기록된 데이터의 양을 누적 숫자로 표현한 것이다. 로그가 기록될수록 이 숫자는 단조 증가하며, 데이터 페이지의 최신성 및 복구 지점을 결정하는 절대적인 척도가 된다.
* **체크포인트 (Checkpoint)**: 버퍼 풀의 더티 페이지가 실제 데이터 파일로 안전하게 기록되었음을 보장하는 지점이다. 체크포인트 이전의 리두 로그는 이미 데이터 파일에 반영되었으므로 덮어써도 안전한 영역(Free Space)이 된다.
* **활성 로그 (Active Redo Log)**: 마지막 체크포인트 지점부터 현재 LSN까지의 구간으로, 시스템 장애 시 반드시 재실행(Replay)되어야 하는 복구 필수 영역이다.

![](/docs/assets/img/2026-01-09-21-07-53.png)

#### 리두 로그 기록 및 복구 단계 (Step-by-Step Trace)

트랜잭션이 발생하여 리두 로그가 영속화되고 복구에 사용되는 과정은 다음과 같다.

1.  **메모리 변경**: 트랜잭션이 버퍼 풀의 특정 데이터 페이지를 수정한다.
2.  **로그 버퍼 기록**: 변경 내용에 대한 물리적 정보(Page No, Offset, New Data)를 메모리 내의 **리두 로그 버퍼**에 기록한다.
3.  **로그 플러시 (Log Flush)**: 트랜잭션이 커밋되거나 버퍼가 가득 차면, OS 커널 버퍼를 거쳐 디스크의 리두 로그 파일로 물리적인 기록(`write` 및 `fsync`)을 수행한다.
4.  **체크포인트 갱신**: 주기적으로 버퍼 풀의 더티 페이지를 데이터 파일에 기입한 후, 리두 로그 파일 헤더에 새로운 체크포인트 LSN을 기록하여 로그 공간을 확보한다.
5.  **장애 복구 (Crash Recovery)**: 시스템 재시작 시, 마지막 체크포인트 LSN과 리두 로그의 마지막 LSN을 비교한다. 두 값이 다를 경우 그 차이만큼의 로그를 읽어 데이터 페이지에 차례로 적용한다.

#### 리두 로그의 설계적 트레이드오프

> **Q: 리두 로그 파일의 크기를 아주 크게 설정하면 시스템에 어떤 영향이 있는가?**
> 
> A: 리두 로그 파일이 클수록 체크포인트 발생 주기를 늦출 수 있어 **쓰기 성능(Throughput)**은 향상된다. 더티 페이지를 디스크로 내보내는 빈도가 줄어들기 때문이다. 그러나 파일이 너무 크면 시스템 장애 발생 시 복구해야 할 로그의 양이 많아져 **복구 시간(Recovery Time)**이 비정상적으로 길어지는 트레이드오프가 발생한다. 반대로 로그가 너무 작으면 빈번한 체크포인트로 인해 버퍼 풀의 플러시 부하가 커져 전체 서비스 성능이 하락한다.
{: .block-warning }

#### 물리적 로깅(Physical)과 논리적 로깅(Logical)의 조화

InnoDB의 리두 로그는 단순히 "어떤 쿼리를 실행했다"는 논리적 기록이 아니라, "어떤 페이지의 몇 번 오프셋에 어떤 바이트를 썼다"는 **물리적 로깅**에 기반한다. 이는 복구 속도를 비약적으로 높여주는데, 인덱스 재구성이나 제약 조건 검사와 같은 복잡한 논리 연산을 생략하고 단순히 디스크 블록을 덮어쓰는 것만으로도 상태 복구가 가능하기 때문이다.

```cpp
/**
 * InnoDB Redo Log: Atomic Page Update with WAL
 * ---------------------------------------------------------
 * This pseudocode demonstrates that the redo log must be 
 * flushed to disk BEFORE the data page is allowed to be 
 * marked as clean in the buffer pool.
 */

void update_data_page(Page *page, byte *new_data) {
    // 1. Generate Redo Log Record (Physiological logging)
    RedoRecord *log = create_redo_record(page->id, page->offset, new_data);

    // 2. Write to Redo Log Buffer in Memory
    redo_log_buffer.append(log);

    // 3. Update the Page in Buffer Pool (Now it's a 'Dirty Page')
    page->apply_change(new_data);
    page->set_lsn(current_system_lsn);

    // 4. Critical Path: Commit requires Log Flush
    // The data file (.ibd) is NOT updated here.
    if (transaction_commit_requested) {
        redo_log_buffer.flush_to_disk(); // Physical I/O: fsync()
    }
}

/**
 * [Architect's Insight]
 * Note that even if the server crashes after step 4 but before 
 * the data page hits the disk, the Redo Log on disk contains 
 * everything needed to reconstruct the change. 
 * This is the essence of Atomicity and Durability.
 */
```

### 어댑티브 해시 인덱스

**어댑티브 해시 인덱스(Adaptive Hash Index)**는 B-Tree 인덱스의 물리적 탐색 한계를 극복하기 위해 InnoDB가 도입한 동적 최적화 메커니즘이다. 사용자가 명시적으로 생성하는 인덱스가 아니라, 엔진이 런타임에 인덱스 키의 탐색 빈도를 모니터링하여 "자주 참조되는 페이지"에 대해 자동으로 생성하는 메모리 기반의 해시 인덱스이다.

#### B-Tree 탐색 비용과 해시의 효율성 분석

전통적인 B-Tree 인덱스는 데이터 양이 늘어날수록 트리의 깊이(Depth)가 깊어지며, 이는 매 검색마다 여러 개의 인덱스 페이지를 거쳐야 하는 $O(\log N)$의 비용을 유발한다. 반면, 어댑티브 해시 인덱스는 특정 키 값에 대해 페이지의 메모리 주소를 즉시 반환하는 $O(1)$의 성능을 목표로 한다.

* **탐색 복잡도**: $O(\log N)$인 B-Tree와 달리 $O(1)$의 상수 시간 탐색을 제공한다.
* **저장 성격**: 디스크에 영구 저장되는 B-Tree와 달리 버퍼 풀 내 메모리 영역에만 존재하는 휘발적 성격을 띤다.
* **관리 자동화**: InnoDB 엔진이 통계 기반으로 스스로 구축 및 파괴를 반복하며, 사용자의 직접적인 관여가 필요 없다.

#### 해시 인덱스 구축 및 가용성 판단 (Step-by-Step Trace)

특정 인덱스 키가 어댑티브 해시 인덱스로 승격되는 과정은 철저히 통계적인 임계치 기반으로 작동한다.

1.  **인덱스 검색 모니터링**: B-Tree 탐색이 발생할 때마다 해당 인덱스 페이지의 접근 횟수와 검색 유형을 기록한다.
2.  **임계치 도달**: 특정 페이지 내의 레코드들이 충분히 자주(자체 알고리즘에 따른 특정 횟수 이상) 검색된다고 판단되면, 해당 페이지를 '해시 대상'으로 분류한다.
3.  **해시 엔트리 생성**: 인덱스 키 값과 해당 레코드가 위치한 **버퍼 풀 페이지의 메모리 주소**를 쌍(Pair)으로 맺어 해시 테이블에 등록한다.
4.  **다이렉트 점프**: 이후 동일한 키 값으로 검색 요청이 들어오면, B-Tree 루트부터 내려가지 않고 해시 테이블을 통해 해당 페이지로 즉시 접근한다.
5.  **무효화(Invalidation)**: 해당 데이터 페이지가 버퍼 풀에서 제거(Evict)되거나 인덱스 구조가 변경되면, 연관된 해시 엔트리도 즉시 삭제하여 데이터 정합성을 유지한다.

![](/docs/assets/img/2026-01-09-21-08-03.png)

#### 기술적 트레이드오프: 뮤텍스 경합과 메모리 자원

> **Q: 어댑티브 해시 인덱스가 항상 성능을 높여준다면, 왜 고부하 환경에서 이 기능을 주의 깊게 관리해야 하는가?**
> 
> A: 해시 테이블 자체가 메모리 상의 공유 자원이기 때문이다. 수많은 스레드가 동시에 인덱스 검색을 수행하는 환경에서, 하나의 해시 테이블에 접근하기 위해 발생하는 **세마포어(Semaphore) 및 뮤텍스 경합**이 오히려 전체 성능을 갉아먹는 병목 지점이 될 수 있다. 또한, 인덱스 키의 삭제나 변경이 빈번한 워크로드에서는 해시 테이블을 최신화하는 비용이 B-Tree 탐색을 절약하는 이득보다 커지는 역효과가 발생한다.
{: .block-warning }

#### 내부 심층 분석: 파티셔닝을 통한 경합 완화

과거의 어댑티브 해시 인덱스는 단일 뮤텍스로 관리되어 고사양 멀티코어 환경에서 심각한 경합 문제를 야기했다. 최신 아키텍처에서는 이를 해결하기 위해 해시 인덱스를 여러 개의 **파티션(Partition)**으로 분할 관리하는 기능을 제공한다. 이는 해시 값에 따라 접근 지점을 분산시켜, 서로 다른 키를 검색하는 스레드들이 각기 다른 뮤텍스를 점유하게 함으로써 동시 처리 성능을 극대화하는 설계적 진보를 보여준다.

```cpp
/**
 * InnoDB Adaptive Hash Index: Frequency Update & Promotion
 * ---------------------------------------------------------
 * This logic represents the engine's internal heuristic to decide
 * whether a B-tree search path should be hashed.
 */

void monitor_index_search(b_tree_path *path, index_key *key) {
    // 1. Update access statistics for the target page
    Page *target_page = path->leaf_page;
    target_page->access_count++;

    // 2. Check if the search pattern is consistent (e.g., same key or prefix)
    if (is_pattern_repeatable(target_page, key)) {
        target_page->pattern_match_count++;
    }

    // 3. Evaluate promotion to Adaptive Hash Index (AHI)
    // Threshold is usually based on page access vs. search pattern stability
    if (target_page->access_count > AHI_ADAPT_THRESHOLD && 
        target_page->pattern_match_count > AHI_PATTERN_THRESHOLD) {
        
        // 4. Create or update hash entry
        // Maps: Hash(Key) -> Physical memory pointer of the record/page
        hash_table_insert(ahi_global_table, key, target_page->memory_ptr);
        
        log_debug("Page %d promoted to Adaptive Hash Index.", target_page->id);
    }
}
```

### 체인지 버퍼

**체인지 버퍼(Change Buffer)**는 보조 인덱스(Secondary Index)의 변경 작업을 지연 처리하여 디스크 I/O를 최적화하는 특수 메모리 구조이다. 레코드가 삽입되거나 수정될 때 연관된 보조 인덱스 페이지가 버퍼 풀에 존재하지 않는다면, 즉시 디스크에서 페이지를 읽어와 갱신하는 대신 변경 사항을 체인지 버퍼에 기록하고 즉시 응답을 반환함으로써 랜덤 I/O 부하를 획기적으로 줄인다.

#### 보조 인덱스 한정 적용의 공학적 이유

체인지 버퍼는 프라이머리 키(PK)나 유니크 인덱스에는 적용되지 않으며, 오직 중복 허용이 가능한 보조 인덱스에만 작동한다. 이는 **유니크 제약 조건(Unique Constraint)**의 물리적 검증 특성 때문이다.

* **즉각적 중복 검사**: 유니크 인덱스는 값을 변경할 때 해당 값이 이미 존재하는지 반드시 확인해야 한다. 이를 위해서는 대상 페이지가 버퍼 풀에 없더라도 디스크에서 읽어와야만 하므로, 변경 사항을 버퍼링하여 지연 처리할 여지가 없다.
* **보조 인덱스의 비동기성**: 반면 일반 보조 인덱스는 중복을 허용하므로, 해당 페이지를 당장 읽지 않아도 논리적인 정합성 파괴가 일어나지 않는다. 이러한 특성이 지연 쓰기를 가능케 하는 핵심 근거가 된다.

#### 체인지 버퍼 머지 프로세스 (Step-by-Step Trace)

버퍼링된 변경 사항이 실제 인덱스 페이지에 반영되는 '머지(Merge)' 과정은 다음과 같은 상황에서 트리거된다.

1.  **페이지 접근**: 사용자가 해당 인덱스 페이지를 필요로 하여 디스크에서 버퍼 풀로 읽어올 때, 체인지 버퍼는 보관 중인 변경 이력이 있는지 확인하여 즉시 병합한다.
2.  **마스터 스레드 작업**: 백그라운드에서 실행되는 마스터 스레드가 주기적으로 체인지 버퍼의 내용을 병합한다.
3.  **퍼지 스레드 연동**: 더 이상 필요 없는 언두 로그를 삭제하는 퍼지(Purge) 과정에서 관련 인덱스 변경 사항을 병합한다.
4.  **공간 부족**: 체인지 버퍼가 할당된 메모리 임계치에 도달하면 강제적인 머지 작업이 발생한다.

![](/docs/assets/img/2026-01-09-21-08-10.png)

#### 체인지 버퍼의 안정성 및 영속성 보장

> **Q: 메모리에만 존재하는 체인지 버퍼의 데이터는 서버 비정상 종료 시 사라지지 않는가?**
> 
> A: 소실되지 않는다. 체인지 버퍼 역시 InnoDB 버퍼 풀의 일부로 간주되며, 체인지 버퍼에 기록되는 모든 변경 사항은 반드시 **리두 로그(Redo Log)**에 먼저 기록(WAL)된다. 따라서 서버가 갑자기 중단되더라도 재시작 시 리두 로그를 통해 체인지 버퍼의 상태가 복구되며, 이후 정상적인 머지 절차를 밟게 된다. 체인지 버퍼는 '성능을 위한 지연'일 뿐 '안정성을 포기한 증발'이 아니다.
{: .block-warning }

#### 기술적 트레이드오프: 쓰기 성능 vs 메모리 점유

체인지 버퍼는 데이터 변경이 빈번한 시스템에서 시스템 자원의 사용처를 결정짓는 핵심 변수이다.

* **인서트 처리량(Throughput) 증대**: 인덱스 페이지를 찾기 위한 디스크 탐색 시간을 제거하여 대량의 데이터 삽입 작업 속도를 비약적으로 높인다.
* **메모리 압박**: 체인지 버퍼는 버퍼 풀의 일정 비율(기본 25%, 최대 50%까지 설정 가능)을 점유한다. 인덱스 변경이 너무 많아 버퍼가 커지면 실제 데이터 페이지를 캐싱할 공간이 줄어들어, 오히려 전체적인 조회 성능이 하락하는 역효과가 발생할 수 있다.

```cpp
/**
 * InnoDB: Change Buffer Insertion Logic
 * ---------------------------------------------------------
 * Pseudocode illustrating the decision process of whether to 
 * buffer a secondary index update or perform a direct write.
 */

enum status ibuf_insert(Index *index, Tuple *entry) {
    // 1. Check if the index is a candidate for buffering
    if (index->is_primary() || index->is_unique()) {
        return IBUF_NOT_APPLICABLE; // Must read disk to check uniqueness
    }

    // 2. Check if the target page is already in the buffer pool
    Page *page = buffer_pool.get_page_if_present(index->id, entry->key);
    
    if (page != NULL) {
        // Page is already in memory, apply change directly
        apply_to_page(page, entry);
        return IBUF_DIRECT_APPLIED;
    }

    // 3. Page is on disk. Check Change Buffer capacity
    if (change_buffer.has_free_space()) {
        // Buffer the operation (Insert/Delete-mark/Purge)
        change_buffer.store(index->id, entry);
        
        // Log to Redo Log for durability (WAL)
        log_to_redo_buffer(IBUF_OP, index->id, entry);
        
        return IBUF_BUFFERED;
    }

    // 4. Fallback if buffer is full: Forced disk I/O
    return IBUF_NOT_BUFFERED_FULL;
}
```