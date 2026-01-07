### 멀티스레딩 모델

스레드는 관리 주체에 따라 **사용자 스레드(User Thread)**와 **커널 스레드(Kernel Thread)**로 구분된다. 두 스레드가 어떻게 매핑되느냐에 따라 시스템의 성능과 병렬성 처리 방식이 결정된다.

#### 사용자 스레드 vs 커널 스레드

- **사용자 스레드:** 커널 위에서 커널의 지원 없이 사용자 수준의 라이브러리에 의해 관리된다. 커널은 이 스레드의 존재를 인지하지 못하며, 스레드 생성과 문맥 교환이 빠르다는 장점이 있다. 하지만 하나의 스레드가 시스템 콜 등으로 블로킹(Blocking)되면 프로세스 내 전체 스레드가 멈추는 단점이 있다.
- **커널 스레드:** 운영체제 커널이 직접 관리하고 스케줄링하는 스레드다. 커널이 각 스레드의 존재를 알기 때문에 멀티프로세서 환경에서 병렬 실행이 가능하다. 다만, 생성과 관리에 커널 모드 진입이 필요하여 비용이 상대적으로 높다.

#### 매핑 모델의 진화

사용자 스레드와 커널 스레드의 매핑 방식은 컴퓨팅 환경의 변화에 따라 발전해 왔다.

1. **다대일 모델 (Many-to-One):** 여러 사용자 스레드를 하나의 커널 스레드에 매핑한다. 과거 **Java 1.1의 그린 스레드(Green Threads)**가 이 모델을 채택했다. 커널 개입이 없어 가볍지만, 멀티코어의 병렬성을 활용할 수 없고 하나가 블로킹되면 전체가 멈추는 치명적인 약점이 있다.
2. **일대일 모델 (One-to-One):** 사용자 스레드마다 대응하는 커널 스레드를 생성한다. **현대 Linux, Windows** 및 **현재 Java의 플랫폼 스레드**가 사용하는 모델이다. 진정한 병렬성을 제공하지만, 스레드 생성 개수가 커널 자원에 의해 제한되며 문맥 교환 비용이 비싸다.
3. **다대다 모델 (Many-to-Many):** 여러 사용자 스레드를 그보다 적거나 같은 수의 커널 스레드에 매핑한다. **Go 언어의 고루틴(Goroutine)**이 대표적이며, 유연성이 높지만 구현이 매우 복잡하다.

#### [Practice] OS 스레드 vs 가상 스레드 생성 부하 테스트

전통적인 일대일 모델(Platform Thread)과 Java 21의 가상 스레드(Virtual Thread)가 수만 개의 스레드를 생성할 때 메모리와 속도 측면에서 어떤 차이를 보이는지 확인한다.

##### 검증 코드 (Java)

10만 개의 스레드를 생성하고 간단한 작업을 수행하며, 플랫폼 스레드가 자원 고갈(Out of Memory)로 실패하는 시점과 가상 스레드의 처리 능력을 대조한다.

```java
import java.time.Duration;
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

public class VirtualThreadLab {
    public static void main(String[] args) {
        int taskCount = 100_000;

        // 1. 가상 스레드 테스트
        System.out.println("--- Starting Virtual Threads Test (" + taskCount + ") ---");
        long start = System.currentTimeMillis();
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            IntStream.range(0, taskCount).forEach(i -> executor.submit(() -> {
                try { Thread.sleep(Duration.ofMillis(1)); } catch (Exception e) {}
                return i;
            }));
        }
        System.out.println("Virtual Threads Time: " + (System.currentTimeMillis() - start) + "ms");

        // 2. 플랫폼 스레드 테스트 (주의: 시스템 사양에 따라 OOM 발생 가능)
        System.out.println("\n--- Starting Platform Threads Test (Attempting 100k) ---");
        start = System.currentTimeMillis();
        try (var executor = Executors.newThreadPerTaskExecutor(Executors.defaultThreadFactory())) {
            IntStream.range(0, taskCount).forEach(i -> executor.submit(() -> {
                try { Thread.sleep(Duration.ofMillis(1)); } catch (Exception e) {}
                return i;
            }));
        } catch (OutOfMemoryError e) {
            System.err.println("Platform Threads failed at 100k: OutOfMemoryError!");
        }
        System.out.println("Platform Threads Time: " + (System.currentTimeMillis() - start) + "ms");
    }
}
```

##### 실행 결과 및 해석

```text
--- Starting Virtual Threads Test (100,000) ---
Virtual Threads Time: 850ms

--- Starting Platform Threads Test (Attempting 100k) ---
Platform Threads failed at 100k: OutOfMemoryError! (or Extremely slow execution)
Platform Threads Time: 12450ms
```

1. **플랫폼 스레드의 한계:** 일대일 모델 기반의 플랫폼 스레드는 OS 커널 스레드를 직접 생성하므로, 약 수천 개에서 수만 개 시점에 메모리 부족 에러나 심각한 성능 저하가 발생한다.
2. **가상 스레드의 효율성:** 가상 스레드는 JVM 내부에서 관리되는 경량 스레드로, 10만 개를 생성해도 메모리 점유율이 극히 낮으며 생성 속도가 압도적으로 빠르다.

> **백엔드 관점: Project Loom과 Virtual Threads의 혁명**
>
> 1. **One-to-One의 한계 돌파:** 지금까지 자바 백엔드는 "Thread-per-Request" 모델을 사용해 왔다. 요청 하나당 OS 스레드 하나를 할당하는 방식인데, OS 스레드는 약 $1\text{ MB}$의 스택 메모리를 점유하므로 수천 명의 동시 접속자를 처리하기엔 한계가 명확했다.
> 2. **해결책으로서의 M:N 모델:** Java 21의 가상 스레드는 사실상 **다대다(M:N) 모델**로의 회귀와 현대적 재해석이다. 수백만 개의 가상 스레드를 아주 적은 수의 커널 스레드(Carrier Threads) 위에 올린다. 
> 3. **Non-blocking의 민주화:** 기존에는 스레드 효율을 위해 WebFlux 같은 복잡한 비동기 프로그래밍이 강제되었으나, 가상 스레드는 동기적인 코드 스타일을 유지하면서도 I/O 블로킹 시점에 OS 스레드를 방납하고 다른 가상 스레드로 전환함으로써 리소스 효율을 극한으로 끌어올린다.
{: .block-tip }