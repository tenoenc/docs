### 운영체제 설계 및 구현

운영체제는 매우 크고 복잡한 시스템이기 때문에, 설계 단계에서부터 **변경에 유연하게 대처할 수 있는 구조**를 잡는 것이 중요하다. 그중 가장 중요한 원칙이 바로 **정책과 메커니즘의 분리**다.

#### 정책(Policy)과 메커니즘(Mechanism)의 분리

이 원칙은 시스템을 설계할 때 **'무엇을(What)'** 할 것인지와 **'어떻게(How)'** 할 것인지를 명확히 구분하는 것이다.



##### 메커니즘 (Mechanism): 어떻게 할 것인가? (How)

어떤 기능을 수행하기 위한 **도구**나 **구현 그 자체**를 말한다. 하드웨어와 직접적으로 상호작용하거나 저수준의 연산을 수행하는 부분이다. 메커니즘은 자주 변하지 않는다.

- **예시**:
    - **문맥 교환(Context Switch):** CPU 레지스터 값을 저장하고 복구하는 기능 자체.
    - **타이머 인터럽트:** 일정 시간마다 신호를 보내는 하드웨어적 기능.
    - **메모리 보호:** 기준 레지스터와 한계 레지스터를 비교하여 접근을 차단하는 기능.

##### 정책 (Policy): 무엇을 할 것인가? (What)

주어진 메커니즘을 사용하여 **어떤 결정을 내릴지 정하는 규칙**이다. 사용자 요구사항이나 시스템 부하 상황에 따라 자주 변경될 수 있다.

- **예시**:
    - **CPU 스케줄링:** "문맥 교환(메커니즘)"을 이용해, "처리량이 많은 프로세스를 먼저 실행할 것인가, 응답 시간이 중요한 프로세스를 먼저 실행할 것인가?"를 결정.
    - **페이지 교체:** "페이지 교체(메커니즘)" 기능을 이용해, "LRU(가장 오래된 것)를 버릴 것인가, FIFO(먼저 들어온 것)를 버릴 것인가?"를 결정.

**분리의 장점**

정책과 메커니즘이 분리되어 있다면, 메커니즘(커널 코드)을 다시 작성하지 않고도 정책(파라미터 설정이나 모듈 교체)만 변경하여 시스템을 튜닝할 수 있다. 반대로 두 가지가 결합되어 있다면 정책을 바꿀 때마다 커널 전체를 다시 컴파일해야 하는 비효율이 발생한다.

> **백엔드 관점: 좋은 아키텍처의 황금률**
>
> 이 챕터는 백엔드 개발자가 **객체지향 설계(OOD)**를 할 때 가장 중요한 원칙과 맞닿아 있다.
>
> 1. **전략 패턴 (Strategy Pattern):** 변하지 않는 전체 흐름(Context/Mechanism)에서 알고리즘(Strategy/Policy)만 인터페이스로 분리하여 갈아 끼우는 패턴이다.
> 2. **의존성 주입 (DI):** Spring Framework에서 `@Service` 로직(Policy)은 `@Repository` 구현체(Mechanism)가 무엇이든(JPA든 JDBC든) 상관없이 동작해야 한다.
{: .block-tip }

#### [Practice] 전략 패턴을 통한 정책과 메커니즘의 분리 구현

운영체제가 커널 수정 없이 스케줄링 알고리즘을 바꿀 수 있는 이유는 설계 단계에서 정책과 메커니즘을 분리했기 때문이다. 이를 Java의 **전략 패턴(Strategy Pattern)**으로 구현하여, "실행(Mechanism)"은 유지한 채 "규칙(Policy)"만 동적으로 교체하는 유연함을 확인해보자.

##### 1. 검증 코드 (Java)

작업을 실행하는 환경(`TaskExecutor`)은 **메커니즘**에 해당하며, 작업의 순서를 결정하는 `SchedulingPolicy`는 **정책**에 해당한다. 

```java
import java.util.*;

// 1. Policy Interface (무엇을 할 것인가?)
interface SchedulingPolicy {
    void sort(List<Task> tasks);
}

// 2. Concrete Policies (다양한 정책들)
class FifoPolicy implements SchedulingPolicy {
    public void sort(List<Task> tasks) { /* 그대로 둠 */ }
}

class PriorityPolicy implements SchedulingPolicy {
    public void sort(List<Task> tasks) {
        tasks.sort(Comparator.comparingInt(t -> t.priority));
    }
}

// 3. Mechanism (어떻게 실행할 것인가? - 변하지 않는 부분)
class TaskExecutor {
    private SchedulingPolicy policy;

    public void setPolicy(SchedulingPolicy policy) {
        this.policy = policy;
    }

    public void runTasks(List<Task> tasks) {
        System.out.println("\n[System] Applying Policy: " + policy.getClass().getSimpleName());
        policy.sort(tasks); // 정책에 따라 순서 결정
        
        for (Task t : tasks) {
            System.out.println("Executing: " + t.name);
        }
    }
}

class Task {
    String name;
    int priority;
    Task(String name, int priority) { this.name = name; this.priority = priority; }
}

public class PolicyMechanismLab {
    public static void main(String[] args) {
        List<Task> tasks = new ArrayList<>(Arrays.asList(
            new Task("Low-Priority-Task", 10),
            new Task("High-Priority-Task", 1)
        ));

        TaskExecutor executor = new TaskExecutor();

        // 정책 1 적용 (FIFO)
        executor.setPolicy(new FifoPolicy());
        executor.runTasks(tasks);

        // 정책 2 적용 (Priority) - 메커니즘 코드는 수정 없이 정책만 교체
        executor.setPolicy(new PriorityPolicy());
        executor.runTasks(tasks);
    }
}
```

##### 2. 실행 결과 및 해석

코드를 실행하여 정책 교체에 따른 시스템 동작 변화를 확인한다.

```text
[System] Applying Policy: FifoPolicy
Executing: Low-Priority-Task
Executing: High-Priority-Task

[System] Applying Policy: PriorityPolicy
Executing: High-Priority-Task
Executing: Low-Priority-Task
```

1.  **메커니즘의 고정:** `TaskExecutor` 클래스의 `runTasks()` 메서드(작업을 꺼내어 실행하는 로직)는 단 한 줄도 수정되지 않는다.
2.  **정책의 유연성:** `FifoPolicy`를 `PriorityPolicy`로 갈아 끼우는 것만으로 시스템 전체의 작업 처리 순서가 바뀐다.
3.  **결론:** OS가 인터럽트와 문맥 교환이라는 강력한 **메커니즘**을 갖춰놓고, 그 위에 다양한 스케줄링 **정책**을 얹는 것과 동일한 원리이다. 이 분리 덕분에 시스템은 요구사항 변화에 압도적으로 유연하게 대처할 수 있다.