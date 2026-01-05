### 프로세스 스케줄링

다중 프로그래밍(Multiprogramming)의 목적은 CPU 사용률을 최대화하기 위해 항상 어떤 프로세스가 실행되도록 하는 것이다. 시분할(Time Sharing)의 목적은 CPU를 프로세스들 사이에서 빈번하게 교체함으로써 사용자가 각 프로그램이 동시에 실행되는 것처럼 느끼게 하는 것이다. 이 목적을 달성하기 위해 **프로세스 스케줄러**는 실행 가능한 프로세스 중 하나를 선택해 CPU를 할당한다.

#### 스케줄링 큐 (Scheduling Queues)

프로세스는 시스템에 진입하면서 다양한 큐를 거쳐 이동한다. 운영체제는 이러한 큐를 일반적으로 **연결 리스트(Linked List)** 형태로 관리하며, 각 큐의 헤더는 리스트의 첫 번째와 마지막 PCB를 가리키는 포인터를 포함한다.



- **작업 큐 (Job Queue):** 시스템에 있는 모든 프로세스로 구성된다. (현대 OS에서는 개념적으로만 존재하거나 준비 큐와 통합되는 경우가 많다.)
- **준비 큐 (Ready Queue):** 메인 메모리에 적재되어 즉시 실행 가능한 상태(Ready)의 프로세스들이 CPU 할당을 기다리는 큐다.
- **장치 큐 (Device Queue):** 특정 입출력 장치(Disk, Network 등)를 사용하기 위해 대기하는 프로세스들의 리스트다. 각 장치마다 고유의 장치 큐가 존재한다.

**큐잉 다이어그램 (Queuing Diagram)**

프로세스 스케줄링은 큐잉 다이어그램으로 표현할 수 있다.
1. 새로운 프로세스가 **준비 큐**에 놓인다.
2. CPU를 할당받아 실행(Dispatch)된다.
3. 실행 중 다음 상황이 발생할 수 있다.
    - I/O 요청을 하여 **장치 큐**로 이동 (I/O Wait)
    - 할당 시간(Time Slice)이 만료되어 다시 **준비 큐**로 이동 (Interrupt)
    - 자식 프로세스를 생성하고 그 종료를 기다림 (**Wait** 상태)
    - 강제 종료되어 큐에서 제거됨

#### 문맥 교환 (Context Switch)

인터럽트가 발생하거나 스케줄러가 CPU를 다른 프로세스로 넘겨줄 때, 시스템은 현재 실행 중인 프로세스의 문맥(Context)을 저장하고, 새로 실행될 프로세스의 문맥을 복원해야 한다. 이 과정을 **문맥 교환**이라 한다.



- **문맥(Context):** 프로세스의 **PCB**에 표현된 상태다. CPU 레지스터 값, 프로세스 상태, 메모리 관리 정보 등을 포함한다.
- **오버헤드 (Overhead):** 문맥 교환이 일어나는 동안 CPU는 아무런 유용한 작업(User Application Code 실행)을 하지 못한다. 따라서 문맥 교환 시간은 순수한 오버헤드다.
- **속도 결정 요인:** 메모리 속도, 레지스터의 수, 특수 명령어(단일 명령으로 레지스터 전체 저장/복원) 지원 여부 등에 따라 수 마이크로초에서 수 밀리초까지 소요된다.

#### [Practice] Context Switch Benchmark: Process vs Thread

이론적으로 프로세스 간 문맥 교환이 스레드 간 문맥 교환보다 비싸다고 배웁니다. 하지만 그 차이가 얼마나 나는지 체감하기는 어렵습니다. C언어의 `pipe`와 `setjmp/longjmp` 또는 실제 스레드 라이브러리를 사용하여 두 환경의 전환 속도를 직접 측정해 봅니다.

##### 1. 검증 시나리오 (C Language)

두 개의 프로세스(또는 스레드)가 서로에게 아주 작은 데이터를 주고받으며 제어권을 넘기는 행위를 수만 번 반복하고, 전체 소요 시간을 측정합니다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>
#include <pthread.h>
#include <sys/wait.h>

#define ITERATIONS 50000

// --- [Thread Context Switch Test] ---
int pipe_t[2];
void* thread_func(void* arg) {
    char c = 't';
    for (int i = 0; i < ITERATIONS; i++) {
        write(pipe_t[1], &c, 1);
        read(pipe_t[0], &c, 1);
    }
    return NULL;
}

// --- [Process Context Switch Test] ---
void process_test() {
    int pipe_p[2];
    pipe(pipe_p);
    pid_t pid = fork();

    if (pid == 0) { // 자식
        char c = 'p';
        for (int i = 0; i < ITERATIONS; i++) {
            write(pipe_p[1], &c, 1);
            read(pipe_p[0], &c, 1);
        }
        exit(0);
    } else { // 부모
        char c = 'p';
        clock_t start = clock();
        for (int i = 0; i < ITERATIONS; i++) {
            read(pipe_p[0], &c, 1);
            write(pipe_p[1], &c, 1);
        }
        wait(NULL);
        clock_t end = clock();
        printf("[Process] Avg Context Switch: %.3f us\n", (double)(end - start) / ITERATIONS);
    }
}

int main() {
    // 1. Thread Test
    pipe(pipe_t);
    pthread_t t;
    pthread_create(&t, NULL, thread_func, NULL);
    
    char c = 't';
    clock_t start = clock();
    for (int i = 0; i < ITERATIONS; i++) {
        read(pipe_t[0], &c, 1);
        write(pipe_t[1], &c, 1);
    }
    pthread_join(t, NULL);
    clock_t end = clock();
    printf("[Thread ] Avg Context Switch: %.3f us\n", (double)(end - start) / ITERATIONS);

    // 2. Process Test
    process_test();

    return 0;
}
```

##### 2. 실행 결과 및 해석

```text
[Thread ] Avg Context Switch: 1.240 us
[Process] Avg Context Switch: 4.850 us
```

-   **수치적 차이:** 일반적으로 스레드 간 문맥 교환이 프로세스 간 문맥 교환보다 **2배에서 수십 배까지** 빠릅니다.
-   **캐시와 TLB:** 프로세스가 바뀔 때는 하드웨어의 메모리 관리 장치(MMU)가 가리키는 페이지 테이블을 아예 갈아치워야 합니다. 이때 CPU 캐시와 TLB(Translation Lookaside Buffer)가 무효화(Flush)되는 비용이 매우 큽니다.
-   **스레드의 이점:** 스레드 전환 시에는 스택과 레지스터 정보만 교체하면 되므로, 캐시 데이터가 유지될 확률이 높아 훨씬 가볍습니다.



> **백엔드 관점: C10K 문제와 가상 스레드(Virtual Threads)**
>
> 과거에는 접속자 1만 명을 처리하기 위해 1만 개의 프로세스를 띄우는 것이 불가능했습니다(C10K 문제). 그래서 우리는 **멀티스레딩**을 선택했습니다. 하지만 스레드조차 OS 커널이 관리하는 '무거운 자원'이기 때문에 수십만 개의 연결을 처리하기엔 역부족이었습니다.
> Java 21에서 도입된 **Virtual Threads**는 이 문맥 교환 비용을 **커널 모드가 아닌 유저 모드**에서 처리하여, OS 스레드 전환보다 훨씬 더 가벼운 '나노초 단위'의 전환을 가능하게 합니다. 우리가 OS의 문맥 교환 비용을 공부하는 이유는 바로 이러한 고차원적인 최적화 기술을 이해하기 위해서입니다.
{: .block-tip }

> **백엔드 관점: 문맥 교환의 진짜 비용 (Cache Pollution & TLB)**
>
> 단순히 "레지스터를 저장하고 복구하는 시간"만이 비용의 전부가 아닙니다. 백엔드 성능 관점에서 더 치명적인 비용은 **"캐시가 차갑게 식는 것(Cold Cache)"**입니다.
>
> 1. **캐시 오염 (Cache Pollution):**
>     - 프로세스 A가 실행되는 동안 CPU 캐시(L1, L2, L3)는 A가 사용하는 데이터로 채워져 있습니다(Warmed up).
>     - 문맥 교환이 발생해 프로세스 B가 실행되면, 캐시에 있는 A의 데이터는 B에게 쓸모가 없습니다(Garbage).
>     - B는 필요한 데이터를 메모리(RAM)에서 다시 캐시로 가져와야 하므로, 실행 초기에 심각한 메모리 지연(Memory Latency)이 발생합니다.
>
> 2. **TLB 플러시 (TLB Flush):**
>     - 프로세스마다 고유한 가상 메모리 주소 공간을 가집니다. 따라서 프로세스가 바뀌면 가상 주소를 물리 주소로 변환하는 캐시인 **TLB(Translation Lookaside Buffer)**를 비워야(Flush) 합니다.
>     - TLB가 비워지면 이후 메모리 접근 시마다 페이지 테이블을 다시 뒤져야 하므로 성능이 저하됩니다.
>
> **빌드업: 왜 스레드(Thread)인가?**
> 스레드는 프로세스와 달리 **메모리 주소 공간(Code, Data, Heap)을 공유**합니다. 따라서 같은 프로세스 내에서 스레드끼리 문맥 교환을 할 때는 **TLB를 비울 필요가 없고**, 캐시 데이터의 상당 부분을 재활용할 수 있습니다. 이것이 스레드를 **'경량 프로세스(LWP)'**라고 부르는 핵심 이유입니다.
{: .block-tip }