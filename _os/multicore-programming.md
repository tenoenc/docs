### 멀티코어 프로그래밍

현대 컴퓨팅 시스템은 단일 칩에 여러 연산 코어를 포함하는 멀티코어 설계를 지향한다. 이에 따라 개발자는 소프트웨어가 여러 코어를 효율적으로 활용할 수 있도록 설계해야 한다.

#### 병행성(Concurrency) vs 병렬성(Parallelism)

멀티스레드 환경에서 자주 혼용되는 두 개념은 실행 방식에 있어 명확한 차이가 존재한다.

- **병행성 (Concurrency):** 단일 프로세서 시스템에서 여러 작업이 **교대로 실행**되어 동시에 실행되는 것처럼 보이는 상태를 의미한다. CPU 스케줄러가 시분할(Time-slicing)을 통해 빠르게 문맥 교환을 수행함으로써 구현된다.
- **병렬성 (Parallelism):** 멀티 프로세서 시스템에서 여러 작업이 **물리적으로 동일한 시간에 실행**되는 상태를 의미한다. 각 작업은 서로 다른 코어에 할당되어 실제로 동시에 연산을 수행한다.



#### 암달의 법칙 (Amdahl's Law)

코어의 개수를 무한히 늘린다고 해서 프로그램의 성능이 그에 비례하여 무한히 좋아지지는 않는다. 암달의 법칙은 프로그램 내부에서 병렬화가 불가능한 **순차적인 부분(Sequential portion)** 때문에 전체 성능 향상에 한계가 발생함을 수학적으로 증명한다.

$$S = \frac{1}{(1-P) + \frac{P}{N}}$$

- $S$: 전체 시스템의 속도 향상(Speedup)
- $P$: 전체 작업 중 병렬화가 가능한 부분의 비율 ($0 \le P \le 1$)
- $1-P$: 병렬화가 불가능한 순차 실행 영역의 비율
- $N$: 프로세서(코어)의 개수

암달의 법칙에 따르면, 병렬화 가능한 부분이 95%이더라도 순차 실행 영역(5%)이 존재한다면 코어를 아무리 늘려도 최대 속도 향상은 20배를 넘지 못한다. 이는 멀티코어 최적화에서 단순히 코어 수를 늘리는 것보다 순차 실행 영역을 최소화하는 설계가 더 중요함을 시사한다.



#### [Practice] Parallel Stream을 이용한 병렬 처리 성능 측정

Java의 병렬 스트림(Parallel Stream)을 사용하여 동일한 연산을 수행할 때 코어 활용도에 따른 실제 성능 차이를 측정하고, 암달의 법칙이 시사하는 '순차 영역의 존재'를 체감한다.

##### 검증 코드 (Java)

대량의 숫자 리스트에 대해 복잡한 연산을 수행하며, 순차 스트림(`stream()`)과 병렬 스트림(`parallelStream()`)의 소요 시간을 비교한다.

```java
import java.util.List;
import java.util.stream.LongStream;
import java.util.stream.Collectors;

public class MulticoreBenchmark {
    public static void main(String[] args) {
        long n = 10_000_000;
        List<Long> numbers = LongStream.rangeClosed(1, n)
                                       .boxed()
                                       .collect(Collectors.toList());

        // 1. 순차 처리 (Sequential Stream)
        long start = System.currentTimeMillis();
        long sum1 = numbers.stream()
                           .mapToLong(x -> x * x) // 무거운 연산 시뮬레이션
                           .sum();
        long end = System.currentTimeMillis();
        System.out.println("Sequential Time: " + (end - start) + "ms");

        // 2. 병렬 처리 (Parallel Stream)
        start = System.currentTimeMillis();
        long sum2 = numbers.parallelStream()
                           .mapToLong(x -> x * x)
                           .sum();
        end = System.currentTimeMillis();
        System.out.println("Parallel Time:   " + (end - start) + "ms");
        
        System.out.println("Available Processors: " + Runtime.getRuntime().availableProcessors());
    }
}
```

##### 실행 결과 및 해석

```text
Sequential Time: 450ms
Parallel Time:   110ms
Available Processors: 8
```

1. **병렬 성능 향상:** 멀티코어 환경에서 병렬 스트림은 `ForkJoinPool`을 사용하여 작업을 분할하므로 순차 처리보다 빠른 속도를 보여준다.
2. **한계점 발견:** 코어 수만큼 정확히 N배 빨라지지 않는다. 이는 작업을 분할하고 합치는 과정에서 발생하는 오버헤드와 병렬화할 수 없는 관리 로직(순차 영역)이 존재하기 때문이다.

> **백엔드 관점: 무조건적인 병렬화의 위험성**
>
> 1. **오버헤드 고려:** 병렬 처리는 데이터를 쪼개고 다시 합치는 과정(Divide and Conquer)에서 추가적인 CPU 비용이 발생한다. 데이터의 크기가 작거나 연산이 단순하다면 오히려 순차 처리가 더 빠를 수 있다.
> 2. **공유 자원 경합:** 병렬로 실행되는 스레드들이 동기화 블록(`synchronized`)이나 공유 데이터에 접근한다면, 이는 암달의 법칙에서 말하는 '순차 실행 영역'이 늘어나는 것과 같다. 병렬 효율을 극대화하려면 **불변 객체(Immutable Object)**를 사용하거나 상태 공유를 최소화해야 한다.
{: .block-tip }