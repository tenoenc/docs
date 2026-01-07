### 기본 개념

운영체제는 다중 프로그래밍(Multiprogramming)을 통해 CPU 이용률을 극대화한다. 이를 위해 프로세스들 사이에서 CPU를 적절히 교체하며 실행하는 스케줄링이 필수적이다.

#### CPU-I/O Burst Cycle

프로세스의 실행은 CPU 연산을 수행하는 **CPU Burst**와 I/O를 기다리는 **I/O Burst**의 순환으로 이루어진다. 프로세스는 이 두 상태를 번갈아 수행하다가, 마지막 CPU Burst가 끝나는 시점에 시스템 실행 종료를 요청한다.

- **Burst 분포의 특징:** 수많은 프로세스를 측정해 보면 짧은 CPU Burst가 굉장히 많이 발생하며, 긴 CPU Burst는 상대적으로 적게 발생한다.
- **스케줄링의 전략적 근거:** I/O-bound 프로세스는 짧은 CPU Burst를 자주 가진다. 이들에게 우선적으로 CPU를 할당하여 빨리 I/O 작업을 하러 떠나게 만드는 것이 전체적인 자원 효율성 면에서 유리하다.

#### 스케줄러 (CPU Scheduler)와 디스패처 (Dispatcher)

스케줄링은 '누구를 고를 것인가'와 '어떻게 실행시킬 것인가'로 나뉜다.

- **CPU 스케줄러:** 준비 큐(Ready Queue)에 있는 프로세스들 중 다음 CPU를 할당받을 프로세스를 선택한다. (단기 스케줄러의 역할)
- **디스패처:** 스케줄러가 선택한 프로세스에게 실제 CPU 제어권을 넘겨주는 모듈이다.
    - **Context Switch:** 현재 실행 중인 프로세스의 상태(PCB)를 저장하고 새 프로세스의 상태를 복구한다.
    - **Mode Switching:** 커널 모드에서 사용자 모드로 전환한다.
    - **Jump:** 사용자 프로그램의 중단되었던 지점으로 이동하여 실행을 재개한다.

디스패처가 하나의 프로세스를 정지시키고 다른 프로세스를 실행하는 데 걸리는 시간을 **디스패치 지연(Dispatch Latency)**이라고 하며, 이는 시스템 오버헤드의 주요 요인이다.


#### 선점형(Preemptive) vs 비선점형(Non-preemptive)

CPU 스케줄링 결정은 프로세스의 상태 변화가 일어나는 특정 시점에 발생한다.

1. **Running → Waiting:** I/O 요청 등 자발적 대기.
2. **Running → Ready:** 타이머 인터럽트 등 강제 중단.
3. **Waiting → Ready:** I/O 완료 후 다시 실행 대기.
4. **Terminated:** 프로세스 종료.

- **비선점형 스케줄링:** 1번과 4번 상황에서만 스케줄링이 발생한다. 한 번 CPU를 잡으면 스스로 내려놓을 때까지 독점한다.
- **선점형 스케줄링:** 2번과 3번 상황을 포함하여, OS가 필요 시 실행 중인 프로세스로부터 CPU를 강제로 뺏어올 수 있다. 현대 OS는 빠른 응답성을 위해 대부분 선점형 방식을 채택한다.


#### [Practice] Thread 실행 순서와 선점 확인

Java 스레드를 통해 여러 작업이 CPU를 나누어 쓰는 병행 실행 과정을 확인하고, 시스템이 어떻게 작업 제어권을 전환하는지 관찰한다.

##### 검증 코드 (Java)

동일한 작업을 수행하는 두 개의 스레드가 번갈아 실행되며 콘솔을 점유하는 과정을 통해 선점형 스케줄링의 동작을 시뮬레이션한다.

```java
public class SchedulingBasicDemo {
    public static void main(String[] args) {
        Runnable task = () -> {
            for (int i = 1; i <= 5; i++) {
                System.out.println(Thread.currentThread().getName() + " - Count: " + i);
                // 연산 부하를 주어 스케줄링이 일어날 틈을 제공
                try { Thread.sleep(1); } catch (InterruptedException e) {}
            }
        };

        Thread threadA = new Thread(task, "Thread-A");
        Thread threadB = new Thread(task, "Thread-B");

        System.out.println("--- Scheduling Start ---");
        threadA.start();
        threadB.start();
    }
}
```

##### 실행 결과 및 해석

```text
--- Scheduling Start ---
Thread-A - Count: 1
Thread-B - Count: 1
Thread-A - Count: 2
Thread-B - Count: 2
Thread-B - Count: 3
Thread-A - Count: 3
...
```

1. **시분할 동작 확인:** 한 스레드가 모든 작업을 끝내고 다음 스레드가 시작되는 것이 아니라, 출력문이 뒤섞여 나온다. 이는 OS가 선점형 스케줄링을 통해 CPU 시간을 쪼개어 배분하고 있음을 의미한다.
2. **비결정론적 실행:** 매 실행 시마다 출력 순서가 달라질 수 있는데, 이는 OS 스케줄러의 동적인 결정에 의존하기 때문이다.

> **백엔드 관점: 선점형 스케줄링의 부작용**
>
> 선점형 시스템은 응답성이 좋지만, 공유 데이터에 접근하는 도중 CPU를 뺏길 경우 데이터 불일치(Inconsistency)가 발생할 수 있다. 예를 들어, 자바에서 `ArrayList`와 같은 비동기화 컬렉션에 여러 스레드가 동시에 쓰기 작업을 할 때 선점이 발생하면 데이터가 유실될 수 있다. 따라서 선점형 OS 환경에서 돌아가는 백엔드 애플리케이션은 항상 임계 구역(Critical Section) 보호를 염두에 두어야 한다.
{: .block-tip }