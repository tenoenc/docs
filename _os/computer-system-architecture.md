### 컴퓨터 시스템 아키텍처

컴퓨터 시스템은 사용되는 범용 프로세서(Processor)의 개수에 따라 분류할 수 있다. 과거에는 싱글 프로세서 시스템이 주류였으나, 현재는 데스크톱과 모바일 장치를 포함한 대부분의 시스템이 멀티 프로세서 구조를 채택하고 있다.



#### 싱글 프로세서 vs 멀티 프로세서

- **싱글 프로세서 시스템:** 범용 명령어 세트를 실행할 수 있는 메인 CPU가 하나만 존재하는 시스템이다. (물론 디스크나 키보드 컨트롤러 같은 특수 목적 프로세서는 존재할 수 있다.)
- **멀티 프로세서 시스템 (병렬 시스템):** 두 개 이상의 프로세서가 버스(Bus), 클록(Clock), 메모리, 주변 장치를 공유하며 긴밀하게 연결된 시스템이다.
    - **증가된 처리량 (Increased Throughput):** N개의 프로세서를 사용하면 속도가 N배까지는 아니더라도(오버헤드 발생) 처리량이 증가한다.
    - **규모의 경제 (Economy of Scale):** 여러 개의 싱글 시스템을 구축하는 것보다, 하나의 본체에 여러 프로세서를 두는 것이 주변 장치와 전력을 공유하므로 비용 효율적이다.
    - **증가된 신뢰성 (Increased Reliability):** 하나의 프로세서가 고장 나도 시스템이 멈추지 않고 속도만 느려진 채 작동할 수 있다. 이를 **우아한 성능 저하(Graceful Degradation)** 또는 **결함 허용(Fault Tolerance)**이라 한다.

#### 대칭형 멀티프로세싱 (SMP, Symmetric Multiprocessing)

멀티 프로세서 시스템에서 CPU를 관리하는 방식 중 가장 일반적인 형태다.



- **동등한 관계 (Peer-to-Peer):** 모든 프로세서가 대등한 관계를 가진다. 주(Master) 프로세서와 종(Slave) 프로세서의 구분이 없다.
- **메모리 공유:** 모든 프로세서가 물리 메모리를 공유한다.
- **OS 수행:** 각 프로세서는 운영체제 커널의 복사본을 실행하거나, 운영체제 코드를 공유하여 스스로 스케줄링을 수행한다.
- **현대 서버의 표준:** Windows, Linux, macOS, Android 등 대부분의 현대 운영체제는 SMP를 지원한다.

> 반대 개념인 **비대칭형 멀티프로세싱(ASMP)**은 하나의 마스터 프로세서가 시스템을 제어하고 나머지 프로세서에 작업을 할당하는 방식이다. 구조는 단순하지만 마스터 프로세서가 병목 지점이 될 수 있다.
{: .block-tip }

#### 멀티코어 (Multi-core) 설계

멀티 프로세서 구조를 구현하는 최신 트렌드는 하나의 칩(Chip) 안에 여러 개의 연산 코어(Core)를 내장하는 것이다.



- **온칩(On-Chip) 통신:** 여러 개의 칩을 보드에 꽂는 방식보다, 하나의 칩 내부에서 코어끼리 통신하는 것이 훨씬 빠르고 전력 소모가 적다.
- **OS의 인식:** 운영체제 입장에서는 각 코어가 하나의 논리적인 CPU로 보인다. 예를 들어 4코어 CPU가 장착된 서버는 OS에게 4개의 프로세서가 있는 것처럼 인식된다.

##### [Practice] OS의 코어 인식과 Java 병렬 처리

운영체제는 물리적인 CPU 코어를 추상화하여 '논리 프로세서'로 인식한다. 이 실습에서는 Java 애플리케이션이 OS가 인식한 코어 정보를 어떻게 받아오고, 이를 바탕으로 어떻게 스레드를 병렬로 분배하는지 확인한다.

###### 검증 코드 (Java)

`Runtime` 클래스를 통해 OS 정보를 확인하고, `ForkJoinPool`이 관리하는 병렬 스트림이 실제로 여러 스레드를 동시에 사용하는지 추적한다.

```java
import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

public class MultiCoreCheck {
    public static void main(String[] args) {
        // 1. OS가 인식하는 논리 프로세서(Core) 개수 확인
        // 하이퍼스레딩이 적용된 4코어 CPU라면 8이 출력됨
        int availableProcessors = Runtime.getRuntime().availableProcessors();
        
        System.out.println("=== [System Info] ===");
        System.out.println("OS-Recognized Logical Processors: " + availableProcessors);
        System.out.println("Common Pool Parallelism: " + ForkJoinPool.commonPool().getParallelism());
        System.out.println("=====================\n");

        System.out.println("=== [Parallel Processing Test] ===");
        
        // 2. 코어 수보다 많은 작업(Task)을 병렬로 수행하여 스레드 분배 확인
        IntStream.rangeClosed(1, 10).parallel().forEach(i -> {
            String threadName = Thread.currentThread().getName();
            // 각 작업이 어떤 스레드에서 실행되는지 출력
            System.out.printf("[Task %2d] Executed by: %s\n", i, threadName);
            try { Thread.sleep(50); } catch (InterruptedException e) {}
        });
    }
}
```

###### 실행 결과 및 해석

코드를 실행하면 다음과 같은 결과를 확인할 수 있다.

1.  **Logical Processors:** OS가 인식한 논리 코어의 개수가 출력된다. Java의 스레드 스케줄링은 전적으로 이 숫자에 의존한다.
2.  **Worker Threads:** 로그를 보면 `worker-1`, `worker-2` 등 서로 다른 스레드가 동시에 작업을 처리함을 알 수 있다. 이는 OS가 각 스레드를 서로 다른 물리 코어(또는 논리 코어)에 매핑(Scheduling)해주었기 때문에 가능한 현상이다.

> **백엔드 관점: 스레드 풀 설정과 Scale-up**
>
> 1. **스레드 풀 사이징:** Java 애플리케이션 튜닝 시 스레드 풀(Thread Pool)의 적정 크기를 결정할 때 `Runtime.getRuntime().availableProcessors()`를 호출한다. 이때 반환되는 값이 바로 OS가 인식하는 **코어(논리 프로세서)의 개수**이다. CPU 바운드 작업이라면 `코어 수 + 1`, I/O 바운드 작업이라면 코어 수보다 넉넉하게 잡는 것이 일반적이다.
> 2. **수직적 확장 (Scale-up):** 서버의 성능을 높이기 위해 서버 대수를 늘리는 것(Scale-out)이 아니라, CPU 코어나 메모리를 증설하는 것을 Scale-up이라 한다. SMP 구조 덕분에 우리는 코어만 추가해도 별도의 소프트웨어 변경 없이(물론 동시성 튜닝은 필요하지만) 더 많은 트래픽을 처리할 수 있다.
> 3. **캐시 일관성 문제의 심화:** 멀티 코어 환경에서는 각 코어가 L1, L2 캐시를 별도로 가지므로, Java의 `volatile` 키워드나 동기화 블록 없이 변수를 공유하면 **가시성(Visibility) 문제**가 발생한다.
{: .block-tip }