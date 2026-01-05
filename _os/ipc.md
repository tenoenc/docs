### 프로세스 간 통신

시스템 내의 프로세스들은 **독립적(Independent)**이거나 **협력적(Cooperating)**이다. 독립적인 프로세스는 서로 영향을 주지 않지만, 협력적인 프로세스는 데이터 공유, 연산 가속화, 모듈화 편의성을 위해 서로 데이터를 주고받아야 한다. 이 메커니즘을 **IPC**라고 한다.

IPC에는 근본적으로 다른 두 가지 모델이 존재한다.



#### 공유 메모리 (Shared Memory)

두 개 이상의 프로세스가 특정 메모리 영역을 구축하고, 이 영역을 통해 데이터를 읽고 쓰는 방식이다.

- **동작 원리:**
    1. 프로세스 A가 시스템 콜(`shmget` 등)을 통해 공유 메모리 세그먼트를 생성한다.
    2. 프로세스 B가 이 세그먼트를 자신의 주소 공간에 첨부(Attach)한다.
    3. 이후 커널의 개입 없이 일반적인 메모리 접근(Memory Access)만으로 데이터를 교환한다.
- **장점:** 커널을 거치지 않으므로 **가장 빠르다**. 대량의 데이터 전송에 유리하다.
- **단점:** 프로세스들이 동시에 같은 메모리 주소에 쓰려고 할 때 데이터 불일치가 발생할 수 있다. 따라서 개발자가 직접 **동기화(Synchronization)** 문제를 해결해야 한다.

#### [Practice] Shared Memory (Low Overhead)

공유 메모리 방식의 가장 큰 특징은 **"커널의 개입을 최소화"**한다는 점입니다. 초기 설정(`setup`) 단계에서만 시스템 콜을 사용하고, 이후 데이터 접근은 일반 메모리 변수를 다루듯 **포인터 연산**만으로 수행됩니다.

##### 1. 검증 코드 (C Language)

C언어의 `mmap()` 함수를 사용하여 파일을 프로세스의 메모리 주소 공간에 직접 매핑합니다. 이후 `write()` 시스템 콜 없이 오직 `memcpy()`(메모리 복사)만으로 데이터를 저장하는 과정을 확인합니다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/mman.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    const char *filepath = "shared_data.bin";
    const char *message = "Hello, Shared Memory! (Direct Access via Pointer)";
    
    // 파일 열기 (공유 메모리 객체 역할)
    int fd = open(filepath, O_RDWR | O_CREAT, 0666);
    if (fd == -1) { perror("open"); return 1; }
    
    // 파일 크기 확보 (메모리 공간 확보)
    ftruncate(fd, 1024);

    // [핵심] mmap: 파일을 프로세스의 메모리 주소에 매핑 (System Call)
    char *shared_memory = mmap(NULL, 1024, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    close(fd); // 매핑 후에는 파일 디스크립터가 필요 없음

    printf("Memory Mapped at address: %p\n", shared_memory);

    // [핵심] Data Write: 시스템 콜(write) 없이, 순수 메모리 연산으로 데이터 씀
    // 커널 모드로 전환되지 않음 -> 고성능
    memcpy(shared_memory, message, strlen(message) + 1);
    
    printf("Data written to memory: %s\n", shared_memory);

    munmap(shared_memory, 1024); // 메모리 해제
    return 0;
}
```

##### 2. 실행 결과 및 해석

```text
Memory Mapped at address: 0x7f94b2800000
Data written to memory: Hello, Shared Memory! (Direct Access via Pointer)
```

-   **핵심:** `mmap()` 호출이 성공한 뒤에는 커널을 거치지 않고(Zero System Call) 데이터를 씁니다.
-   **의의:** 마치 내 프로그램의 변수에 값을 대입하는 것과 똑같은 속도로 IPC 통신이 가능함을 의미합니다. 대용량 데이터 처리에 압도적으로 유리합니다.

#### 메시지 전달 (Message Passing)

운영체제가 제공하는 통신 수단을 통해 메시지 단위로 데이터를 주고받는 방식이다. 프로세스들이 메모리를 공유하지 않으므로 별도의 동기화 로직 없이도 충돌을 피할 수 있다.

- **동작 원리:** 운영체제는 `send(message)`와 `receive(message)`라는 두 가지 연산을 제공한다. 프로세스는 커널에게 메시지를 전달하고, 커널이 이를 수신 프로세스에게 전달한다.
- **구현 방식:**
    - **파이프 (Pipe):** 데이터가 한쪽 방향으로만 흐르는 통신 채널. 보통 부모-자식 프로세스 간 통신에 사용된다.
    - **소켓 (Socket):** 네트워크 인터페이스를 통한 통신. 서로 다른 머신에 있는 프로세스 간 통신(분산 시스템)의 표준이다.
- **장점:** 구현이 쉽고 안전하다. 분산 시스템 환경에서 사용 가능하다.
- **단점:** 데이터를 보낼 때마다 **시스템 콜**이 필요하며, 커널 메모리를 거쳐서 복사가 일어나므로 공유 메모리보다 느리다.

#### [Practice] Message Passing (Kernel Mediation)

반면, 파이프(Pipe)나 소켓(Socket)을 이용한 메시지 전달 방식은 데이터를 주고받을 때마다 **운영체제 커널이 중재자**로서 개입해야 합니다.

##### 1. 검증 코드 (C Language)

`pipe()` 시스템 콜로 커널 버퍼를 생성하고, `fork()`로 생성된 자식 프로세스가 `write()` 시스템 콜을 통해 부모에게 데이터를 보냅니다.

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main() {
    int pipe_fd[2]; // [0]: Read, [1]: Write
    pid_t pid;
    char buffer[100];

    // [핵심] 파이프 생성 (커널 내부 버퍼 생성)
    if (pipe(pipe_fd) == -1) { perror("pipe"); return 1; }

    pid = fork(); // 프로세스 복제

    if (pid > 0) { // 부모 프로세스 (Reader)
        close(pipe_fd[1]); // 쓰기 채널 닫기
        
        // [핵심] Data Read: 시스템 콜 발생 (Kernel Buffer -> User Buffer)
        read(pipe_fd[0], buffer, sizeof(buffer));
        printf("Parent received: %s\n", buffer);
        
        close(pipe_fd[0]);
        wait(NULL); // 자식 종료 대기
    } else { // 자식 프로세스 (Writer)
        close(pipe_fd[0]); // 읽기 채널 닫기
        
        const char *msg = "Hello from Child via Pipe!";
        // [핵심] Data Write: 시스템 콜 발생 (User Buffer -> Kernel Buffer)
        // 문맥 교환 발생
        write(pipe_fd[1], msg, strlen(msg) + 1);
        
        close(pipe_fd[1]);
    }
    return 0;
}
```

##### 2. 실행 결과 및 해석

```text
Parent received: Hello from Child via Pipe!
```

-   **핵심:** 데이터를 보낼 때 `write()`, 받을 때 `read()`라는 **시스템 콜이 매번 발생**합니다.
-   **오버헤드:** 데이터가 **[유저 영역 → 커널 버퍼 → 유저 영역]**으로 두 번 복사되며, 이 과정에서 문맥 교환(Context Switch) 비용이 필연적으로 발생합니다.

> **백엔드 관점: 고성능 아키텍처의 선택**
>
> 1.  **Shared Memory:** **Redis**나 **Kafka**(Zero-Copy), 고성능 DB들이 극한의 성능을 낼 때 이 방식을 사용합니다. 다만 동기화 문제(Locking)를 개발자가 직접 해결해야 하므로 구현 난이도가 높습니다.
> 2.  **Message Passing:** **gRPC**, **REST API**, 그리고 대부분의 **MSA 통신**이 이 모델에 해당합니다. 성능은 조금 떨어지지만, 커널(또는 네트워크 스택)이 동기화와 전달을 책임지므로 구현이 안전하고 쉽습니다.
{: .block-tip }

> **백엔드 관점: MSA 아키텍처로의 확장**
>
> 이 로컬 IPC 개념은 현대적인 **MSA(Microservices Architecture)** 환경의 통신 방식과 정확히 매핑됩니다.
>
> 1. **Shared Memory 모델 → Redis 캐시 공유 / Shared DB**
>     - 여러 마이크로서비스(Process)가 **Redis**나 **DB**라는 외부 저장소(Shared Memory)를 통해 상태를 공유합니다.
>     - **장점:** 서비스 A가 데이터를 넣으면 서비스 B가 즉시 볼 수 있어 속도가 빠릅니다.
>     - **Pain Point:** 동시성 문제가 그대로 발생합니다. 두 서비스가 동시에 재고를 차감하려 할 때 **분산 락(Distributed Lock)**이 필수적인 이유는 OS의 공유 메모리 동기화 문제와 같습니다.
>
> 2. **Message Passing 모델 → REST API, gRPC, Message Queue**
>     - **REST/gRPC:** 소켓(Socket) 통신을 기반으로 하며, 명시적으로 요청과 응답을 주고받습니다.
>     - **Message Queue (RabbitMQ, Kafka):** OS의 메일박스(Mailbox) 역할을 하는 큐를 통해 비동기로 메시지를 주고받습니다.
>     - **특징:** 메모리를 공유하지 않으므로 서비스 간 **결합도(Coupling)**가 낮아지고, 서로의 내부 상태를 모르기 때문에 더 안전한 아키텍처가 됩니다.
>
> **결론:** "MSA에서 왜 DB를 공유하지 말고 API로 통신하라고 할까?"
> 이는 **"공유 메모리는 빠르지만 위험하고(결합도 높음), 메시지 패싱은 느리지만 안전하다(결합도 낮음)"**는 OS의 오랜 교훈을 분산 시스템에 적용한 것입니다. (Database per Service 패턴)
{: .block-tip }