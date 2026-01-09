## MySQL 엔진

### 커넥션 핸들러

**커넥션 핸들러**는 데이터베이스 시스템의 물리적 관문이자, 클라이언트의 요청을 개별적인 실행 단위로 전환하는 중추적 아키텍처이다. 이는 네트워크 프로토콜 하위 계층에서의 연결 확립부터, 상위 계층의 보안 인증 및 스레드 자원 할당에 이르기까지 전 과정을 관장하며 시스템의 동시성 처리 한계를 결정짓는 제1 방어선 역할을 수행한다.

#### 연결 확립 및 세션 초기화의 공학적 단계

클라이언트가 서버에 접속하여 쿼리 실행 준비를 마치기까지의 과정은 단순한 연결을 넘어 자원 격리 및 문맥(Context) 구성의 연속이다.

1.  **네트워크 핸드쉐이크**: 커널 수준의 TCP/IP 연결이 완료되면, 커넥션 핸들러는 운영체제로부터 파일 디스크립터(File Descriptor)를 넘겨받아 통신 채널을 확보한다.
2.  **스레드 프로비저닝 (Thread Provisioning)**: 연결된 소켓을 처리할 실행 주체를 결정한다. 이때 **스레드 캐시(Thread Cache)**를 우선 조회하여 유휴 스레드를 재사용함으로써 OS 커널의 스레드 생성 오버헤드를 억제한다.
3.  **프로토콜 핸드쉐이크 및 인증**: 서버는 클라이언트에게 핸드쉐이크 패킷을 전송하여 지원 가능한 플래그와 난수(Salt)를 교환한다. 이후 사용자 자격 증명 및 호스트 기반 접근 제어(ACL) 검증을 거친다.
4.  **세션 문맥 할당**: 인증이 성공하면 해당 커넥션만을 위한 독립적 메모리 영역인 **커넥션 버퍼**를 할당하고, 사용자별 시스템 변수와 권한 정보를 캐싱하여 쿼리 실행을 위한 준비 상태로 전환한다.

#### 연결 관리 모델의 기술적 대조

동시 접속자를 처리하는 방식은 서버의 확장성과 자원 효율성에 결정적인 영향을 미친다.

| 비교 속성 | 1:1 스레드 할당 모델 (Thread-per-Connection) | 스레드 풀 모델 (Thread Pool) |
| :--- | :--- | :--- |
| **자원 할당 방식** | 커넥션마다 전용 OS 스레드 생성 | 한정된 스레드 뭉치가 다수의 커넥션을 공유 처리 |
| **CPU 효율성** | 접속자 급증 시 컨텍스트 스위칭 비용 폭증 | 활성 스레드 수를 제한하여 CPU 캐시 효율 최적화 |
| **메모리 점유** | 스택 메모리 점유가 커넥션 수에 비례하여 증가 | 스레드 수에 비례한 고정적 메모리 사용 |
| **고부하 상황 대응** | 시스템 전체의 응답 불능(OOM 등) 위험 존재 | 요청을 큐잉(Queuing)하여 시스템 안정성 유지 |
| **구현 복잡도** | 비교적 단순하고 직관적 | 고도의 스케줄링 및 우선순위 큐 관리 필요 |

#### 내부 작동 원리 심층 분석

커넥션 핸들러 내부에서 스레드가 관리되는 메커니즘은 시스템 자원의 임계치를 보호하기 위한 논리로 설계되어 있다. 가용 스레드가 부족할 경우 서버는 즉각적으로 새로운 스레드를 생성하기보다 스레드 캐시의 적중률을 먼저 계산한다. 이는 `pthread_create`와 같은 시스템 콜이 유발하는 커널 모드 전환 비용과 메모리 할당 지연을 최소화하기 위함이다. 

또한, 할당된 스레드는 쿼리 처리가 완료된 후에도 즉시 소멸하지 않고 다시 캐시로 돌아가 다음 연결을 대기한다. 이 "생성-재활용" 루프는 고성능 데이터베이스 시스템이 초당 수천 개의 짧은 연결을 처리할 수 있게 하는 핵심 동력이다.

```c
/**
 * Connection Handler: Thread Lifecycle Management
 * ---------------------------------------------------------
 * This pseudocode illustrates the core logic of reusing or 
 * creating a thread when a new client connection arrives.
 */

void handle_new_connection(int client_fd) {
    Thread *assigned_thread = NULL;

    // Phase 1: Search for an idle thread in the Global Thread Cache
    if (!thread_cache.is_empty()) {
        assigned_thread = thread_cache.pop();
        log_debug("Reusing existing thread from cache. (Thread ID: %d)", assigned_thread->id);
    } 
    // Phase 2: Create a new thread if within maximum limits
    else if (current_thread_count < max_connections) {
        assigned_thread = create_new_system_thread();
        current_thread_count++;
        log_debug("Created new OS thread. (Total: %d)", current_thread_count);
    } 
    // Phase 3: Reject connection if resources are exhausted
    else {
        close_connection(client_fd);
        log_error("Connection rejected: max_connections reached.");
        return;
    }

    // Phase 4: Associate thread with connection and initialize context
    assigned_thread->bind(client_fd);
    assigned_thread->run_authentication_flow();
}
```

#### 논리적 추론 및 통찰

> **Q: 클라이언트의 커넥션이 물리적으로는 유지되고 있으나 아무런 쿼리도 보내지 않을 때, 커넥션 핸들러는 이를 어떻게 관리하는가?**
> 
> A: 이러한 상태를 'Sleep' 상태라고 정의한다. 커넥션 핸들러는 세션별로 마지막 활동 시간을 기록하며, `wait_timeout`으로 정의된 임계치에 도달할 때까지 쿼리가 유입되지 않으면 자원 보호를 위해 해당 연결을 강제로 단절한다. 이때 중요한 점은, 네트워크 소켓은 닫히더라도 해당 스레드는 다시 스레드 캐시로 반납되어 다른 활성 커넥션에 할당될 준비를 한다는 것이다. 즉, '연결의 단절'과 '스레드의 소멸'은 독립적으로 운영된다.
{: .block-tip }

#### 기술적 트레이드오프: Connection Limit vs System Stability

커넥션 핸들러의 최대 허용 수치(`max_connections`) 설정은 성능과 안정성 사이의 치열한 저울질 결과여야 한다.

* **높은 연결 제한**: 더 많은 동시 접속을 수용하여 서비스 가용성을 높이지만, 임계치를 넘어서는 순간 CPU 스케줄러의 부하가 기하급수적으로 증가하며 전체 쿼리 처리 속도가 하락하는 '성능의 절벽' 현상을 초래할 수 있다.
* **낮은 연결 제한**: 시스템 자원의 여유를 확보하여 개별 쿼리의 응답 속도는 보장할 수 있으나, 피크 타임 시 유효한 사용자 요청이 거부(Connection Refused)되는 서비스 저하 문제를 일으킨다.

### SQL 파서

**SQL 파서**는 사용자가 전달한 선언적 명령문인 SQL 문자열을 시스템이 처리할 수 있는 추상적 구조체로 치환하는 전처리 공정의 핵심이다. 이 컴포넌트는 단순한 문자열 검색을 넘어, 언어학적 분석 기법을 동원하여 쿼리의 문법적 정밀도를 검증하고 최적화의 토대가 되는 파스 트리(Parse Tree)를 생성한다.

#### 렉시컬 분석과 구문 분석의 논리적 전개

SQL이 내부 객체로 변환되는 과정은 두 단계의 상호 보완적인 분석 과정을 거친다.

1.  **어휘 분석 (Lexical Analysis)**: 입력 스트림을 의미 있는 최소 단위인 '토큰(Token)'으로 쪼개는 과정이다. `SELECT`, `*`, `FROM`, `table_name`과 같이 쿼리를 구성하는 원자적 요소들을 식별하고 분류한다.
2.  **구문 분석 (Syntactic Analysis)**: 추출된 토큰들을 SQL 문법 규례(Grammar Rules)에 따라 배열하고, 각 토큰 간의 관계를 정의한다. 이 과정에서 토큰들은 위계 구조를 가진 트리 형태로 재구성되며, 문법적 오류가 발견될 경우 즉시 실행을 중단하고 클라이언트에게 에러를 반환한다.

#### 분석 단계별 책임 및 특성 대조

| 구분 | 어휘 분석 (Lexer) | 구문 분석 (Parser) |
| :--- | :--- | :--- |
| **주요 입력** | 순수 문자열 (Raw String) | 토큰 시퀀스 (Token Stream) |
| **핵심 행위** | 패턴 매칭 및 토큰화 | 문법 규칙 적용 및 관계 설정 |
| **검출 오류** | 잘못된 문자, 정의되지 않은 키워드 | 순서 위반, 필수 절(Clause) 누락 |
| **결과물** | 단어 뭉치 (Tokens) | 파스 트리 (Parse Tree) |
| **비용 특성** | 선형 스캔 위주의 저비용 연산 | 재귀적 탐색 및 규칙 검증의 고비용 연산 |

#### 쿼리 구조화 단계별 추적 (Step-by-Step Trace)

단순한 조회 쿼리가 파서 내부에서 구조적 객체로 변모하는 과정은 다음과 같다.

1.  **입력 수신**: `SELECT name FROM users WHERE id = 10;`
2.  **어휘 분리**: `[SELECT]`, `[name]`, `[FROM]`, `[users]`, `[WHERE]`, `[id]`, `[=]`, `[10]` 토큰 생성.
3.  **루트 노드 할당**: 쿼리의 시작점인 `SELECT` 노드를 생성하고 하위 분기 준비.
4.  **프로젝션 분석**: 조회 대상인 `name` 칼럼 정보를 `Select List` 자식 노드에 할당.
5.  **데이터 소스 매핑**: 대상 테이블인 `users`를 `From Clause` 자식 노드에 연결.
6.  **필터 조건 구조화**: `WHERE` 절 하위의 `id = 10`을 비교 연산 트리로 구성.

![](/docs/assets/img/2026-01-09-20-33-38.png)

#### 구문 검증의 논리적 한계와 임계점

> **Q: SQL 파서가 '성공'을 반환했다면, 해당 쿼리는 반드시 실행 가능한 상태인가?**
> 
> A: 그렇지 않다. SQL 파서의 유일한 관심사는 "문장이 문법적으로 올바른가"에 국한된다. 존재하지 않는 테이블을 참조하거나, 숫자형 칼럼에 문자열을 비교하는 등의 '실체적 오류'는 파서가 아닌 그다음 단계인 **프리프로세서(Preprocessor)**의 영역이다. 파서는 설계도의 오타를 수정할 뿐, 실제 현장에 해당 자재가 있는지는 확인하지 않는다.
{: .block-warning }

#### 기술적 트레이드오프: Parsing Overhead vs Query Complexity

파싱 공정은 쿼리의 복잡도가 증가함에 따라 기하급수적으로 자원을 소모한다.

* **복잡한 서브쿼리 및 조인**: 트리의 깊이와 너비가 확장되면서 노드 생성 및 메모리 할당 비용이 증가한다. 이는 특히 동시성 높은 환경에서 CPU 점유율을 상승시키는 요인이 된다.
* **쿼리 캐싱 전략**: 동일한 구조의 쿼리가 반복될 경우 파싱 과정을 생략하려는 시도가 있으나, 미세한 공백이나 대소문자 차이로 인해 캐시 미스가 발생하면 파싱 오버헤드는 고스란히 시스템의 지연 시간(Latency)으로 전이된다.

### SQL 옵티마이저

**SQL 옵티마이저**는 데이터베이스 시스템의 지능적 중추로서, 요청된 쿼리를 최단 시간 내에 처리할 수 있는 물리적 실행 경로를 설계한다. 옵티마이저는 단순히 구문을 해석하는 수준을 넘어, 스토리지 엔진으로부터 수집된 통계 정보를 정밀 분석하고 수만 가지의 가용한 경로 중 비용(Cost)이 가장 낮은 최적의 **실행 계획(Execution Plan)**을 산출하는 공학적 의사결정을 수행한다.

#### 비용 모델과 물리적 자원 산출의 심층 메커니즘

최신 데이터베이스 아키텍처는 **비용 기반 최적화(CBO, Cost-Based Optimization)**를 표준으로 채택한다. 여기서 '비용'이란 단순히 처리 시간을 의미하는 것이 아니라, 쿼리 수행을 위해 소모되는 하드웨어 자원의 합산 수치이다. 옵티마이저는 다음과 같은 물리적 지표를 기반으로 각 실행 단계의 비용을 계산한다.

* **I/O 비용**: 디스크로부터 데이터 페이지를 읽어오는 과정에서 발생하는 부하이다. 순차 I/O와 랜덤 I/O의 비용을 차등 계산하며, 버퍼 풀에 데이터가 이미 캐싱되어 있을 확률까지 고려한다.
* **CPU 비용**: 읽어온 레코드를 메모리상에서 비교, 연산, 정렬하는 과정에서의 연산 부하이다. 복잡한 조건식이나 대규모 데이터 정렬이 포함될수록 이 수치는 가중된다.
* **메모리 비용**: 해시 조인이나 임시 테이블 생성을 위해 점유해야 하는 메모리 영역의 크기와 할당/해제 오버헤드를 포함한다.

옵티마이저는 시스템 변수에 정의된 단위 비용(예: `memory_block_read_cost`, `io_block_read_cost`)을 가중치로 사용하여 전체 쿼리의 총합 비용을 산출하며, 이 값이 가장 낮은 경로를 최종 선택한다.

#### 조인 순서 결정 및 검색 공간 최적화 (Step-by-Step Trace)

다중 조인 쿼리에서 테이블을 어떤 순서로 결합하느냐는 성능을 결정짓는 핵심 변수이다. 테이블이 N개일 때 가능한 조인 순서는 $N!$개에 달하며, 옵티마이저는 이를 효율적으로 처리하기 위해 다음과 같은 단계를 밟는다.

1.  **카디널리티 분석**: 각 테이블의 인덱스 통계를 기반으로 조건을 만족하는 예상 레코드 수(Cardinality)를 계산한다.
2.  **드라이빙 테이블 선정**: 결과 집합이 가장 작을 것으로 예측되는 테이블을 조인의 시발점(Driving Table) 후보로 둔다.
3.  **탐색 공간 가지치기 (Pruning)**: 모든 조인 순열을 검토하는 대신, 현재 계산 중인 부분 비용이 이미 발견된 최저 비용을 초과할 경우 해당 경로는 즉시 폐기한다.
4.  **조인 알고리즘 매핑**: 각 단계마다 네스티드 루프 조인(Nested Loop Join)이나 해시 조인(Hash Join) 중 물리적 비용이 낮은 알고리즘을 개별적으로 적용한다.
5.  **실행 계획 직렬화**: 확정된 논리 구조를 쿼리 실행기가 이해할 수 있는 바이너리 형태의 실행 계획으로 변환한다.

#### 통계 정보의 불확실성과 판단 오류

> **Q: 왜 충분한 인덱스가 있음에도 옵티마이저는 풀 테이블 스캔을 선택하는가?**
> 
> A: 이는 **'선택도(Selectivity)'**와 **'랜덤 I/O 비용'**의 상관관계 때문이다. 옵티마이저는 인덱스를 통해 읽어야 할 레코드가 전체의 약 20~25%를 초과한다고 판단하면, 인덱스를 통한 랜덤 I/O보다 한 번에 대량의 페이지를 읽어오는 풀 테이블 스캔(Sequential I/O)이 더 저렴하다고 결론 내린다. 또한, 스토리지 엔진이 제공하는 통계 정보가 실제 데이터 분포와 괴리가 클 경우, 옵티마이저는 잘못된 비용 산출로 인해 최악의 경로를 선택할 수 있다.
{: .block-warning }

#### 기술적 트레이드오프: 최적화 정밀도 vs 탐색 시간

옵티마이저는 최고의 계획을 찾는 것과 그 계획을 찾는 데 걸리는 시간 사이에서 끊임없는 트레이드오프를 수행한다.

* **Greedy Search 기법**: 테이블 수가 방대할 경우 모든 조합을 검토하는 것은 불가능하다. 따라서 매 단계에서 부분적인 최적해를 선택하며 나아가는 탐색 기법을 사용하여 최적화 시간을 단축한다.
* **성능의 임계치**: 쿼리 실행 시간이 수 밀리초(ms)에 불과한 단순 조회 쿼리에서 수 초 동안 최적의 경로를 찾는 것은 자원 낭비이다. 따라서 옵티마이저는 일정 수준 이상의 유효한 계획이 발견되면 탐색을 조기 종료하는 전략을 취한다.

```cpp
/**
 * Optimizer: Cost Estimation Snippet (Conceptual)
 * ---------------------------------------------------------
 * This logic represents how the optimizer weights different 
 * physical operations to derive a total cost for a plan.
 */

double estimate_execution_cost(PlanNode *node) {
    double total_cost = 0.0;

    // 1. Calculate I/O Cost: Based on page reads and cache hit ratio
    double io_cost = node->expected_page_reads * SYSTEM_VAR_IO_BLOCK_COST;
    
    // 2. Calculate CPU Cost: Based on record comparisons and function evals
    double cpu_cost = node->expected_records * SYSTEM_VAR_ROW_EVALUATE_COST;

    // 3. Apply Penalties for Specific Operations (e.g., Sorting, Disk Temp Tables)
    if (node->requires_external_sort) {
        cpu_cost += (node->expected_records * log2(node->expected_records)) * SORT_WEIGHT;
    }

    total_cost = io_cost + cpu_cost;
    
    // Add cumulative cost of child nodes (Recursive cost aggregation)
    for (auto child : node->children) {
        total_cost += estimate_execution_cost(child);
    }

    return total_cost;
}
```

### 쿼리 실행기

**쿼리 실행기(Query Executor)**는 옵티마이저가 설계한 이론적 실행 계획을 실체적인 데이터 연산으로 전환하는 물리적 수행 계층이다. 옵티마이저가 '최적의 경로를 결정하는 두뇌'라면, 쿼리 실행기는 그 결정을 바탕으로 스토리지 엔진에 레코드 읽기/쓰기를 요청하고, 반환된 데이터를 가공(조인, 정렬, 그룹화 등)하여 최종 결과셋을 구성하는 '실행의 손' 역할을 담당한다.

#### 핸들러 API를 통한 스토리지 엔진과의 상호작용

쿼리 실행기의 가장 핵심적인 특징은 특정 스토리지 엔진의 내부 구조(B-Tree, 로그 구조 등)에 직접 관여하지 않는다는 점이다. 대신, **핸들러 API(Handler API)**라는 추상화된 인터페이스를 통해 데이터를 주고받는다.

* **추상화된 데이터 요청**: 실행기는 "인덱스의 다음 레코드를 가져와라" 또는 "PK가 10인 레코드를 읽어라"와 같은 표준화된 명령을 핸들러 인터페이스에 전달한다.
* **엔진 독립성**: 이러한 구조 덕분에 MySQL 엔진은 하위 스토리지 엔진이 InnoDB이든 MyISAM이든 관계없이 동일한 로직으로 쿼리를 실행할 수 있다.
* **반복적 호출 (Volcano Model)**: 대부분의 실행 과정은 'Open - Next - Close' 단계로 이루어진 반복자(Iterator) 패턴을 따르며, 조건에 맞는 레코드가 더 이상 없을 때까지 엔진으로부터 레코드를 한 건씩 끌어올린다(Pull).

#### 엔진과 실행기의 책임 분담 구조

데이터 처리의 효율성을 극대화하기 위해 실행 계층과 저장 계층은 명확한 역할 분담을 가진다.

| 책임 영역 | 쿼리 실행기 (MySQL 엔진) | 스토리지 엔진 (InnoDB 등) |
| :--- | :--- | :--- |
| **데이터 접근** | 핸들러 API 호출 (명령 하달) | 물리적 디스크/버퍼 풀 I/O 수행 |
| **필터링 및 연산** | 복잡한 수식 계산, 사용자 정의 함수 실행 | 인덱스를 이용한 범위 제한 및 단순 비교 |
| **중간 결과 관리** | 임시 테이블 생성, 정렬, 그룹화 작업 | 인덱스 정렬 순서 유지 및 잠금 관리 |
| **데이터 일관성** | 격리 수준에 따른 읽기 요청 | MVCC 관리 및 언두/리두 로그 처리 |

#### 실행 프로세스의 단계별 추적 (Step-by-Step Trace)

단순 조인 쿼리가 실행기에서 처리되는 물리적 흐름은 다음과 같다.

1.  **핸들러 초기화**: 실행 계획에 명시된 테이블들에 대해 핸들러 객체를 생성하고 필요한 잠금을 획득한다.
2.  **드라이빙 레코드 추출**: 첫 번째 테이블(Driving Table)에서 조건을 만족하는 첫 레코드를 읽어오도록 엔진에 요청한다.
3.  **조인 버퍼 적재 및 결합**: 읽어온 레코드를 기반으로 두 번째 테이블(Driven Table)의 매칭되는 레코드를 찾는다. 이때 옵티마이저가 결정한 조인 알고리즘(예: NLJ)에 따라 반복적인 API 호출이 발생한다.
4.  **결과 가공 (Post-processing)**: 모든 조인이 완료된 데이터에 대해 `WHERE` 절의 추가 필터링을 수행하거나, 메모리/디스크 상에서 정렬(Filesort)을 수행한다.
5.  **결과셋 반환**: 가공이 완료된 레코드를 커넥션 핸들러로 전달하여 클라이언트에게 전송한다.

![](/assets/img/2026-01-09-20-35-04.png)

#### 논리적 심층 분석 및 통찰

> **Q: 엔진에서 가져온 데이터를 실행기가 다시 필터링하는 이유는 무엇인가?**
> 
> A: 모든 조건이 인덱스를 통해 처리될 수 없기 때문이다. 인덱스에 포함된 칼럼 조건은 스토리지 엔진 수준에서 필터링되어 실행기로 올라오는 데이터 양을 줄여주지만(Index Condition Pushdown), 인덱스에 없는 칼럼이나 복잡한 연산이 포함된 조건은 엔진이 판단할 수 없다. 따라서 실행기는 엔진이 건네준 '잠정적 후보' 레코드들을 최종적으로 검사하여 불필요한 데이터를 솎아내는 최종 검수원 역할을 수행해야 한다.
{: .block-warning }

#### 기술적 트레이드오프: Row-based vs Batch Processing

쿼리 실행기의 처리 방식은 시스템 전체의 지연 시간과 처리량(Throughput) 사이에서 상충 관계를 형성한다.

* **개별 레코드 처리 (Row-at-a-time)**: 레코드가 생성되는 즉시 다음 단계로 넘겨 응답 속도(Time to first row)를 높이지만, 수백만 건의 데이터를 처리할 때는 빈번한 API 호출 오버헤드가 누적된다.
* **배치 처리 (Batch Execution)**: 특정 단계의 데이터를 뭉쳐서 한 번에 처리(예: 인덱스 프리페치)하여 전체 I/O 효율을 높이지만, 첫 번째 결과가 나올 때까지의 지연 시간이 길어질 수 있다.

```cpp
/**
 * Executor: Core Execution Loop for Simple Table Scan
 * ---------------------------------------------------------
 * This code demonstrates how the Executor interacts with the 
 * Storage Engine through the 'handler' interface.
 */

void execute_table_scan(handler *file, List<Item> &conditions) {
    int error;
    uchar *record = file->table->record[0];

    // 1. Initialize the scan (Storage Engine prepares its cursors)
    if ((error = file->ha_rnd_init(true))) {
        handle_error(error);
        return;
    }

    // 2. Iterate through records one by one (The Volcano Model)
    while (!(error = file->ha_rnd_next(record))) {
        // 3. Evaluation: MySQL Engine-level filtering
        // Even if the engine returns a row, the Executor must verify it.
        if (evaluate_where_conditions(record, conditions)) {
            // 4. Send the qualified record to the client/connection handler
            send_record_to_client(record);
        }
        
        // Check for kill signal or timeout
        if (current_thd->killed) break;
    }

    // 5. Clean up the scan context
    file->ha_rnd_end();
}
```

### 스토리지 엔진 인터페이스

**스토리지 엔진 인터페이스**는 상위의 SQL 처리 계층과 하위의 데이터 저장 계층을 격리하는 추상화 레이어이다. 이는 객체 지향 설계의 어댑터 패턴(Adapter Pattern)을 데이터베이스 아키텍처 수준에서 구현한 것으로, MySQL 엔진이 특정 저장 기술의 물리적 세부 사항에 종속되지 않고 표준화된 방식으로 데이터를 조작할 수 있게 하는 통로 역할을 한다.

#### 핸들러 아키텍처와 추상화 기작

이 인터페이스의 실체는 **핸들러(Handler)**라고 불리는 추상 클래스이다. 모든 스토리지 엔진은 이 핸들러 클래스를 상속받아 자신만의 물리적 저장 로직을 구현해야 하며, MySQL 엔진은 오직 이 클래스에 정의된 가상 함수(Virtual Function)만을 호출하여 데이터에 접근한다.

이러한 구조는 시스템에 다음과 같은 공학적 특성을 부여한다.

* **플러거블 아키텍처(Pluggable Architecture)**: 사용자는 서버를 재컴파일하지 않고도 실행 중에 스토리지 엔진을 교체하거나 추가할 수 있다.
* **일관된 제어 흐름**: 데이터가 B-Tree에 저장되든, 로그 파일에 저장되든, 혹은 원격 서버에 있든 관계없이 상위 계층은 동일한 API(`ha_index_read`, `ha_write_row` 등)를 통해 데이터를 제어한다.
* **최적화 영역의 분리**: MySQL 엔진은 조인 및 쿼리 변환과 같은 논리적 최적화에 집중하고, 스토리지 엔진은 디스크 I/O 최적화 및 동시성 제어와 같은 물리적 무결성에 집중할 수 있는 분리된 책임 구조를 가진다.

#### 인터페이스 기반 데이터 조작의 단계별 추적

레코드가 검색되거나 삽입되는 과정에서 인터페이스가 어떻게 기능하는지, 그 구체적인 호출 흐름을 추적한다.

1.  **핸들러 인스턴스화**: 쿼리가 특정 테이블을 참조하면, MySQL 엔진은 해당 테이블의 엔진 타입에 맞는 핸들러 객체를 메모리에 생성한다.
2.  **스캔 모드 설정**: 옵티마이저의 결정에 따라 핸들러에게 '풀 테이블 스캔' 혹은 '특정 인덱스 범위 스캔' 모드로 초기화(Init) 명령을 내린다.
3.  **레코드 페칭(Fetching)**: 쿼리 실행기는 핸들러의 `index_read` 또는 `rnd_next` 메서드를 호출한다. 이때 인터페이스는 물리적 레코드를 'MySQL 내부 레코드 포맷'으로 변환하여 상위 계층으로 전달한다.
4.  **트랜잭션 생명주기 관리**: `commit` 또는 `rollback` 명령 역시 이 인터페이스를 통해 전달되며, 하위 엔진은 이를 수신하여 자신의 리두(Redo) 로그와 언두(Undo) 로그를 최종 확정하거나 폐기한다.



#### 논리적 심층 분석 및 통찰

> **Q: 인터페이스라는 추상화 계층이 존재함으로써 발생하는 성능 오버헤드는 없는가?**
> 
> A: 명백히 존재한다. 상위 엔진과 하위 엔진 사이에서 매 레코드마다 가상 함수 호출(Virtual Function Call)이 발생하며, 데이터 포맷을 상호 변환하는 과정에서 CPU 사이클이 소모된다. 특히 대량의 레코드를 스캔할 때 이 오버헤드는 누적되어 전체 성능에 영향을 줄 수 있다. 이를 극복하기 위해 최신 아키텍처에서는 인터페이스를 한 번 호출할 때 여러 레코드를 묶어서 반환하는 **배치 읽기(Batch Read)**나, 필터링 조건을 엔진 내부로 밀어넣는 **조건 푸시다운(Condition Pushdown)** 기능을 도입하여 인터페이스 경계를 넘나드는 횟수를 최소화하고 있다.
{: .block-warning }

#### 기술적 트레이드오프: 범용성 vs 엔진 전용 최적화

스토리지 엔진 인터페이스는 모든 엔진을 수용해야 하므로 '최대 공약수'적인 기능만을 표준화한다.

* **범용성 확보**: 다양한 저장 기술(Memory, Disk, Cloud Storage)을 동일한 SQL 인터페이스로 다룰 수 있게 하여 개발자의 학습 비용을 낮추고 시스템 확장성을 극대화한다.
* **특수 기능의 제한**: 특정 엔진만이 가진 고유한 물리적 강점(예: 하드웨어 가속 정렬)이 있다 하더라도, 인터페이스 표준에 해당 정의가 없다면 상위 엔진은 이를 활용하지 못하고 표준적인 방식으로만 데이터를 요청하게 된다. 이는 시스템이 '상향 평준화'되는 동시에 특정 엔진의 '최고 성능'을 100% 이끌어내기 어렵게 만드는 제약이 되기도 한다.

```cpp
/**
 * Storage Engine Interface: The 'handler' Class
 * ---------------------------------------------------------
 * This represents the abstract contract that every storage 
 * engine must implement to be integrated into MySQL.
 */

class handler : public Sql_alloc {
public:
    // 1. Storage Engine Metadata & Capability flags
    Table *table;                  // Pointer to the table object
    Table_share *table_share;      // Shared metadata across threads

    // 2. Data Retrieval Methods (Standardized Access)
    virtual int index_read_map(uchar *buf, const uchar *key, 
                               key_part_map keypart_map, 
                               enum ha_rkey_function find_flag) = 0;
    
    virtual int rnd_next(uchar *buf) = 0; // Sequential scan to the next row

    // 3. Data Modification Methods
    virtual int ha_write_row(uchar *buf) = 0;  // Insert a new record
    virtual int ha_update_row(const uchar *old_data, uchar *new_data) = 0;
    virtual int ha_delete_row(const uchar *buf) = 0;

    // 4. Transaction Management
    virtual int external_lock(THD *thd, int lock_type) = 0;
};
```