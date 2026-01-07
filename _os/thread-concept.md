### 스레드 개념

하나의 프로세스 내에서 실행되는 여러 흐름의 단위를 스레드라고 한다. 현대의 운영체제는 대부분 멀티스레딩(Multithreading)을 기본으로 지원하며, 애플리케이션 개발에 있어 프로세스보다 더 세밀한 실행 단위로 활용된다.

#### 프로세스 vs 스레드

프로세스가 운영체제로부터 자원을 할당받는 작업의 단위라면, 스레드는 프로세스가 할당받은 자원을 이용하여 실행되는 흐름의 단위다. 두 개념의 가장 큰 차이는 **자원의 공유 여부**에 있다.

- **자원 공유 (Shared):** 같은 프로세스에 속한 스레드들은 **코드(Code), 데이터(Data), 힙(Heap)** 영역을 공유한다. 이를 통해 스레드 간의 통신이 별도의 IPC 없이도 매우 빠르고 효율적으로 이루어진다.
- **독립 공간 (Independent):** 각 스레드는 독립적인 실행 흐름을 유지하기 위해 **스택(Stack)**과 **레지스터(Register)** 세트를 개별적으로 가진다. 스택이 독립적이라는 것은 각 스레드가 자신만의 함수 호출 이력과 지역 변수를 가진다는 의미다.

#### 스레드의 장점

멀티스레드 설계는 다음과 같은 네 가지 주요 이점을 제공한다.

1. **응답성 (Responsiveness):** 프로그램의 일부분이 중단되거나 긴 작업을 수행하더라도 다른 스레드가 실행 중이라면 사용자와의 상호작용을 계속 유지할 수 있다.
2. **자원 공유 (Resource Sharing):** 프로세스 간 통신(IPC)과 달리 공유 메모리나 메시지 전달을 설정할 필요 없이 데이터와 자원을 직접 공유한다.
3. **경제성 (Economy):** 프로세스를 생성하는 것보다 스레드를 생성하는 것이 시간과 메모리 측면에서 훨씬 저렴하다. 또한, 문맥 교환(Context Switch) 오버헤드 역시 스레드 간 전환이 프로세스 간 전환보다 월등히 낮다.
4. **규모 적응성 (Scalability):** 멀티프로세서 아키텍처에서 각 스레드가 서로 다른 프로세서에서 병렬로 실행될 수 있어 성능 향상을 극대화한다.

#### [Practice] 스레드의 자원 공유와 스택의 독립성 확인

동일한 프로세스 내에서 생성된 여러 스레드가 힙(Heap) 영역의 객체는 공유하지만, 각자의 메서드 호출 내에서 정의된 로컬 변수(Stack)는 침범할 수 없음을 Java 코드로 증명한다.

##### 검증 코드 (Java)

전역 변수(Static)를 사용하여 공유 데이터에 접근하는 상황과, 로컬 변수가 각 스레드 내에서 어떻게 독립적으로 관리되는지 확인한다.

```java
public class ThreadConceptLab {
    // 1. 공유 자원 (Data/Heap 영역에 위치)
    private static int sharedCounter = 0;

    public static void main(String[] args) throws InterruptedException {
        Runnable task = () -> {
            // 2. 개별 자원 (각 스레드의 Stack 영역에 위치)
            int localValue = 0;
            
            for (int i = 0; i < 5000; i++) {
                sharedCounter++; // 공유 자원 수정
                localValue++;    // 독립 자원 수정
            }
            
            System.out.println(Thread.currentThread().getName() + 
                " -> Shared Counter (In progress): " + sharedCounter + 
                ", Local Value (Stack): " + localValue);
        };

        Thread thread1 = new Thread(task, "Thread-1");
        Thread thread2 = new Thread(task, "Thread-2");

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();

        System.out.println("\n[Final Result] Shared Counter: " + sharedCounter);
    }
}
```

##### 실행 결과 및 해석

```text
Thread-1 -> Shared Counter (In progress): 6421, Local Value (Stack): 5000
Thread-2 -> Shared Counter (In progress): 9832, Local Value (Stack): 5000

[Final Result] Shared Counter: 9832 (데이터 손실 발생 가능성 확인)
```

1. **힙 공유:** 모든 스레드가 `sharedCounter` 값을 증가시키며 최종 결과에 영향을 미친다. 이는 데이터 영역과 힙 영역이 공유됨을 보여준다.
2. **스택 독립성:** 각 스레드 내부에서 선언된 `localValue`는 다른 스레드의 영향을 받지 않고 고유한 값을 유지한다. 이는 각 스레드가 자신만의 스택 프레임을 가진다는 증거다.

> **백엔드 관점: 멀티스레드 환경의 양날의 검**
>
> 1. **Thread-Safety:** 스레드의 '자원 공유'라는 장점은 백엔드 개발자에게 **동시성 제어**라는 숙제를 안겨준다. 여러 스레드가 동시에 힙 영역의 데이터를 수정하려고 할 때 발생하는 **경합 상태(Race Condition)**를 막기 위해 `synchronized`, `Atomic`, `volatile` 등의 키워드를 이해해야 한다.
> 2. **Stateless 설계:** Spring의 빈(Bean)이 기본적으로 싱글톤(Singleton)으로 관리되는 환경에서, 멤버 변수에 상태를 저장하는 행위는 매우 위험하다. 모든 요청은 개별 스레드에서 처리되므로, 가능한 한 '로컬 변수(Stack)'를 활용하여 스레드 안전성을 확보해야 한다.
{: .block-tip }