### 컴퓨터 시스템의 구성

현대 운영체제는 **인터럽트 구동식(Interrupt driven)**이다. 마우스 클릭, 키보드 입력, 디스크 완료 신호 등 이벤트가 발생하면 CPU는 하던 일을 멈추고 해당 이벤트를 처리한다.

#### 인터럽트 (Interrupts)

인터럽트는 하드웨어 또는 소프트웨어로부터 발생하는 이벤트 신호다. CPU는 인터럽트를 감지하면 즉시 현재 수행 중인 작업을 중단하고, 해당 인터럽트를 처리하기 위한 **서비스 루틴(ISR)**으로 제어권을 옮긴다.

##### 인터럽트의 정의와 발생 원리

인터럽트는 발생 주체에 따라 크게 두 가지로 나뉜다.

- **하드웨어 인터럽트:** 키보드, 마우스, 디스크 컨트롤러, 프린터 등 외부 하드웨어 장치가 CPU에 보내는 신호다. 하드웨어는 시스템 버스를 통해 CPU에 신호를 보내 인터럽트를 발생시킨다.
- **소프트웨어 인터럽트:** 실행 중인 프로그램이 특정 상황(오류 또는 요청)을 알리기 위해 발생시킨다.

##### 트랩(Trap) 및 예외(Exception)

소프트웨어 인터럽트는 주로 **트랩(Trap)** 또는 **예외(Exception)**라고 불린다. 트랩이 발생하는 이유는 크게 두 가지다.

1. **오류(Error):** 0으로 나누기(Division by zero), 잘못된 메모리 접근(Segmentation fault) 등 프로그램 실행 중 발생한 치명적인 오류.
2. **사용자 요청(User Request):** 사용자 프로그램이 운영체제 서비스를 받기 위해 실행하는 **시스템 호출(System Call)**.

> 운영체제는 이중 모드(Dual Mode)로 보호되므로, 사용자 프로그램이 직접 I/O 장치를 건드릴 수 없다. 따라서 시스템 콜이라는 트랩을 발생시켜 커널 모드로 진입한 뒤 OS에게 작업을 요청해야 한다.
{: .block-tip }

##### 인터럽트 벡터 (Interrupt Vector)

인터럽트가 발생했을 때 CPU는 "어떤 코드를 실행해야 하는지" 알아야 한다. 각 인터럽트 종류마다 처리해야 할 코드(ISR)의 시작 주소가 다르기 때문이다.

운영체제는 메모리의 하위 공간에 **인터럽트 벡터**라는 테이블을 유지한다. 이 테이블은 다양한 인터럽트 서비스 루틴들의 주소(포인터)를 배열 형태로 저장하고 있다. 인터럽트 요청이 오면 CPU는 해당 인터럽트 번호를 인덱스로 사용하여 벡터 테이블에서 주소를 찾아 점프한다.

##### 인터럽트 처리 과정

1. **인터럽트 감지 및 요청:** 디스크 읽기 완료 등 이벤트가 발생하여 CPU에 신호를 보낸다.
2. **현재 상태 저장:** CPU는 현재 실행 중인 프로세스의 상태(Program Counter, 레지스터 값 등)를 PCB 또는 스택에 저장한다. 원래 하던 일로 복귀하기 위함이다.
3. **ISR 실행:** 인터럽트 벡터를 통해 해당 인터럽트를 처리할 서비스 루틴(ISR)으로 이동하여 코드를 실행한다.
4. **상태 복구 및 재개:** ISR 처리가 끝나면 `return from interrupt` 명령을 수행하고, 저장해 두었던 이전 프로세스의 상태를 복구하여 중단된 지점부터 다시 실행한다.

**폴링(Polling) vs 인터럽트**

초기 컴퓨터나 아주 단순한 시스템에서는 CPU가 입출력 장치의 상태를 주기적으로 계속 확인하는 **폴링(Polling)** 방식을 사용했다. 하지만 이는 CPU 사이클을 심각하게 낭비한다. 반면, 인터럽트 방식은 CPU가 다른 일을 하다가 이벤트가 발생했을 때만 통보받으므로 처리 효율이 압도적으로 높다.

##### [Practice] Blocking vs Non-blocking I/O

초기 컴퓨터는 CPU가 입출력 장치를 계속 감시하는 **폴링(Polling)** 방식을 썼지만, 현대 시스템은 **인터럽트**를 통해 비동기 처리를 수행한다. 이 차이가 애플리케이션 레벨(Java)에서 스레드 효율성에 어떤 차이를 만드는지 확인해보자.

###### 검증 코드 (Java)

전통적인 `Blocking I/O` 방식(작업이 끝날 때까지 대기)과 `CompletableFuture`를 이용한 `Non-blocking` 방식(요청 후 다른 작업 수행)을 비교한다.

```java
import java.util.concurrent.CompletableFuture;

public class IoComparison {
    public static void main(String[] args) throws Exception {
        System.out.println("=== [1] Blocking I/O Simulation ===");
        blockingIo();

        System.out.println("\n=== [2] Non-blocking I/O Simulation ===");
        nonBlockingIo();
        
        // 메인 스레드 종료 방지 (비동기 결과 확인용)
        Thread.sleep(2000); 
    }

    // 전통적인 블로킹 방식: 작업이 끝날 때까지 스레드가 멈춤
    private static void blockingIo() {
        System.out.println("Requesting I/O...");
        long start = System.currentTimeMillis();
        
        // I/O 작업 시뮬레이션 (1초 대기)
        try { Thread.sleep(1000); } catch (InterruptedException e) {}
        
        System.out.println("I/O Finished. Duration: " + (System.currentTimeMillis() - start) + "ms");
        System.out.println("Executing next logic..."); // 1초 뒤에 실행됨
    }

    // 비동기 논블로킹 방식: 요청 즉시 리턴, 완료 시 콜백 실행 (인터럽트 개념)
    private static void nonBlockingIo() {
        System.out.println("Requesting Async I/O...");
        long start = System.currentTimeMillis();

        CompletableFuture.runAsync(() -> {
            // 별도의 스레드에서 I/O 수행 (실제로는 커널에 요청 후 대기하는 시간)
            try { Thread.sleep(1000); } catch (InterruptedException e) {}
        }).thenRun(() -> {
            // I/O 완료 후 실행될 콜백 (ISR과 유사)
            System.out.println(">> [Callback] Async I/O Finished. Duration: " + (System.currentTimeMillis() - start) + "ms");
        });

        System.out.println("Doing other work immediately..."); // 즉시 실행됨
    }
}
```

###### 실행 결과 및 해석

- **Blocking 방식:** I/O 작업(Simulated by `sleep`)이 진행되는 동안 메인 스레드는 아무것도 하지 못하고 멈춰 있다. (CPU 자원 낭비)
- **Non-blocking 방식:** I/O 작업을 요청하자마자 스레드는 즉시 반환되어 "다른 작업 수행 중..." 메시지를 출력합니다. 이후 I/O가 완료되면(인터럽트 발생 시점과 유사) 콜백이 실행되어 결과를 처리한다.
- **결론:** 인터럽트 기반의 비동기 처리는 I/O 대기 시간 동안 CPU가 다른 유용한 작업을 할 수 있게 하여 시스템의 처리량(Throughput)을 비약적으로 높여준다.

> **백엔드 관점: 비동기 I/O와 Event-Driven**
>
> 인터럽트 메커니즘은 백엔드 개발에서 **Spring WebFlux**나 **Node.js**, **Netty**가 추구하는 **'비동기 논블로킹(Asynchronous Non-blocking) I/O'**의 하드웨어적 근간이 된다.
> 기존의 블로킹(Blocking) 방식은 I/O 작업(DB 조회 등)이 끝날 때까지 스레드가 아무것도 안 하고 대기(Polling과 유사한 낭비)한다. 반면, 논블로킹 방식은 I/O 요청만 보내놓고 스레드는 다른 작업을 처리한다. 이후 I/O가 완료되면 **인터럽트**가 발생하고, 이를 이벤트(Event)로 감지하여 콜백 함수를 실행한다. 즉, CPU와 스레드 자원을 낭비 없이 극한으로 활용할 수 있게 해주는 핵심 원리가 바로 인터럽트이다.
{: .block-tip }

#### 저장장치 구조와 계층 (Storage Structure)

CPU는 메모리에서 명령을 가져와 실행한다. 하지만 메인 메모리는 영구적이지 않고 비용이 비싸다. 따라서 컴퓨터 시스템은 다양한 저장 장치를 계층적으로 구성하여 **속도, 비용, 비휘발성** 사이의 균형을 맞춘다.

##### 저장장치 계층 구조 (Storage-Device Hierarchy)

저장 장치는 접근 속도와 비용에 따라 피라미드 형태의 계층 구조를 이룬다.



- **최상위 (Registers, Cache):** CPU 내부에 있거나 아주 가까이 있어 접근 속도가 매우 빠르지만, 용량이 작고 비싸다. 휘발성이다.
- **중간 (Main Memory):** CPU가 직접 접근할 수 있는 실행 영역이다. DRAM을 주로 사용하며 휘발성이다.
- **하위 (Non-volatile Memory, Magnetic Disk):** SSD(Solid State Disk)나 HDD(Hard Disk Drive)를 포함한다. 용량이 크고 저렴하며 전원이 꺼져도 데이터가 유지되지만, 접근 속도가 메모리에 비해 수십만 배 느리다.

##### 캐싱 (Caching)

캐싱은 정보가 사용될 때 더 빠른 저장 장치(캐시)에 복사해 두는 것을 말한다. 저장장치 계층 구조의 핵심 원리는 **"데이터를 하위 계층에서 상위 계층으로 캐싱하는 것"**이다.

**지역성(Locality) 원리**

캐싱이 효율적인 이유는 프로그램이 **참조의 지역성**을 가지기 때문이다.
- **시간적 지역성:** 한 번 참조된 데이터는 곧 다시 참조될 가능성이 높다 (예: 반복문의 변수).
- **공간적 지역성:** 어떤 데이터가 참조되면 그 주변 데이터도 곧 참조될 가능성이 높다 (예: 배열 순회).

**캐시 일관성(Coherency) 문제**

멀티태스킹이나 멀티프로세서 환경에서는 여러 곳(레지스터, 캐시, 메모리)에 동일한 데이터의 복사본이 존재할 수 있다. 한쪽에서 데이터를 변경했을 때, 다른 쪽의 복사본도 즉시 업데이트되어야 하는데 이를 **캐시 일관성** 문제라고 한다. 분산 시스템에서는 여러 컴퓨터 간의 캐시 일관성을 유지하는 것이 매우 복잡하고 중요하다.

> **백엔드 관점: 성능 최적화의 핵심**
>
> 이 계층 구조는 왜 우리가 **Redis** 같은 In-Memory DB를 메인 DB 앞단에 두는지, 왜 DB 튜닝에서 **Disk I/O**를 줄이는 것이 지상 과제인지 설명해준다.
> 디스크 접근은 메모리 접근보다 약 10만 배 이상 느리다. 따라서 백엔드 성능 개선의 1순위는 언제나 "느린 저장소(Disk/Network)로 가는 횟수를 줄이고, 빠른 저장소(Memory/Cache)에서 처리하는 것"이다. DB의 버퍼 풀(Buffer Pool)이나 인덱스도 결국 디스크 접근을 줄이기 위한 캐싱 전략의 일환이다.
{: .block-tip }

#### 입출력 구조 (I/O Structure)

초기의 입출력 방식은 CPU가 바이트 전송마다 개입해야 했다. 하지만 대량의 데이터를 전송할 때 이러한 방식은 CPU 자원을 심각하게 낭비한다.

##### DMA (Direct Memory Access)

DMA는 CPU의 개입 없이 I/O 장치(디스크 컨트롤러 등)가 직접 메모리에 데이터를 전송하는 기술이다.



1. CPU는 DMA 컨트롤러에게 "A 파일의 데이터를 메모리 B 주소로 옮겨라"라고 명령만 내리고 다른 작업을 수행한다.
2. DMA 컨트롤러는 CPU를 거치지 않고 직접 버스를 통해 디스크에서 메모리로 데이터 블록을 전송한다.
3. 전송이 모두 완료되면 DMA 컨트롤러는 CPU에 **인터럽트**를 한 번만 보내 완료를 알린다.

결과적으로 CPU는 바이트 단위가 아니라 **블록 단위**로 인터럽트를 처리하게 되어 효율성이 극대화된다.

##### [Practice] Zero-Copy (DMA) Implementation

DMA 기술을 활용하면 CPU가 데이터 복사에 관여하지 않아도 되므로 성능이 비약적으로 향상된다. 이를 Java 코드에서 전통적인 방식과 비교해보자.

###### 검증 코드 (Java)

Java NIO의 `transferTo()` 메서드는 리눅스의 `sendfile()` 시스템 콜을 사용하여 커널 영역에서 데이터를 직접 전송하는 **Zero-Copy**를 구현한다.

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.nio.channels.FileChannel;

public class ZeroCopyTest {
    public static void main(String[] args) throws Exception {
        String source = "large-file.dat";
        String dest = "copy.dat";

        // 1. Traditional Copy (User Space Copy)
        // Read -> Kernel Buffer -> User Buffer -> Kernel Buffer -> Write
        try (FileInputStream fis = new FileInputStream(source);
             FileOutputStream fos = new FileOutputStream(dest)) {
            byte[] buffer = new byte[8192];
            int len;
            while ((len = fis.read(buffer)) != -1) {
                fos.write(buffer, 0, len);
            }
        }

        // 2. Zero-Copy (DMA / sendfile)
        // Kernel Buffer -> Kernel Buffer (Direct Transfer)
        try (FileChannel srcChannel = new FileInputStream(source).getChannel();
             FileChannel destChannel = new FileOutputStream(dest).getChannel()) {
            
            // transferTo uses sendfile() system call on Linux
            srcChannel.transferTo(0, srcChannel.size(), destChannel);
        }
    }
}
```

###### 실행 결과 및 해석

- **전통적인 Stream Copy:** 데이터를 커널 버퍼에서 유저 영역의 버퍼로 읽어오고(`read`), 다시 유저 영역에서 커널 버퍼로 쓰는(`write`) 과정에서 불필요한 **CPU 복사 연산과 컨텍스트 스위칭**이 발생한다.
- **Zero-Copy (transferTo):** 데이터가 유저 영역을 거치지 않고 커널 영역(디스크 캐시 -> 소켓/파일 버퍼) 내에서만 이동한다. DMA 컨트롤러가 하드웨어적으로 전송을 담당하므로 **CPU 점유율(Load)**이 현저히 낮아지고 전송 속도는 빨라진다.

> **백엔드 관점: Zero-Copy와 고성능 처리**
>
> 대용량 파일 업로드/다운로드나 스트리밍 서버를 구축할 때 CPU 사용량이 치솟는다면 DMA와 관련된 **Zero-Copy** 기술을 활용하지 못하고 있을 가능성이 크다.
> Java NIO의 `transferTo()` 메서드나 Netty 같은 프레임워크는 내부적으로 리눅스의 `sendfile` 시스템 콜을 활용한다. 이는 데이터가 커널 영역(Disk Read)에서 유저 영역(Application)으로 복사되지 않고, 커널 영역 내에서 바로 소켓 버퍼로 복사되도록 하여 CPU의 복사 연산(Context Switching) 부하를 제거하는 기술이다. 이 배경에는 DMA가 있다.
{: .block-tip }