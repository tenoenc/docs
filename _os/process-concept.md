### 프로세스 개념

운영체제에서 **프로세스(Process)**는 실행 중인 프로그램(Program in execution)을 의미한다. 프로그램은 디스크에 저장된 수동적인 존재(Passive Entity)인 반면, 프로세스는 메모리에 적재되어 PC(Program Counter)를 가지는 능동적인 존재(Active Entity)다.

#### 프로세스 상태 (Process State)

프로세스는 실행되면서 그 상태가 변한다. 프로세스의 상태는 현재 프로세스가 어떤 활동을 하고 있는지를 정의한다.



- **New (생성):** 프로세스가 막 생성되고 있는 단계. 아직 메모리에 완전히 올라오지 않았거나 PCB가 초기화되는 중이다.
- **Running (실행):** CPU를 점유하여 명령어들이 실제로 실행되고 있는 상태. 싱글 코어 시스템에서는 한 순간에 오직 하나의 프로세스만이 Running 상태에 있을 수 있다.
- **Waiting (대기):** 프로세스가 어떤 이벤트(입출력 완료, 신호 수신 등)가 일어나기를 기다리는 상태. 이 상태의 프로세스는 CPU를 주어도 실행할 수 없다.
- **Ready (준비):** CPU만 할당받으면 즉시 실행될 수 있는 상태. 메모리에 적재되어 있으며, 준비 큐(Ready Queue)에서 차례를 기다린다.
- **Terminated (종료):** 프로세스가 실행을 끝내고 자원을 반납하는 단계.

> **상태 전이 (State Transition):**
> - **Scheduler Dispatch:** Ready -> Running (스케줄러가 CPU를 할당)
> - **Interrupt:** Running -> Ready (할당 시간 만료 등으로 인해 강제로 뺏김)
> - **I/O or Event Wait:** Running -> Waiting (스스로 입출력을 요청하고 대기)
> - **I/O or Event Completion:** Waiting -> Ready (요청 완료 후 다시 줄 서기)
{: .block-tip }

> **백엔드 관점: 서버 부하의 정체**
>
> 리눅스 서버에서 `top` 명령어를 쳤을 때 보이는 **Load Average**는 단순히 CPU 사용률이 아니다. 이는 **'Running 상태이거나 Ready 상태에 있는 프로세스의 수'** + **'I/O를 기다리는(Waiting) 프로세스의 수'**를 의미한다.
> - CPU가 바빠서 줄 서 있는 프로세스가 많다면? -> **CPU Bound** (스케일 업/아웃 필요)
> - Disk I/O를 기다리느라 멈춰있는 프로세스가 많다면? -> **I/O Bound** (DB 쿼리 튜닝, 캐시 도입 필요)
{: .block-tip }

#### 프로세스 제어 블록 (PCB, Process Control Block)

운영체제는 각 프로세스를 관리하기 위해 **PCB**라는 자료구조를 커널 메모리 영역에 유지한다. PCB는 특정 프로세스에 대한 모든 정보를 담고 있는 저장소다. (리눅스에서는 `task_struct`라는 이름으로 구현되어 있다.)



**PCB에 저장되는 주요 정보:**

1. **프로세스 상태 (Process State):** new, ready, running 등 현재 상태.
2. **프로그램 카운터 (Program Counter):** 이 프로세스가 다음에 실행할 명령어의 주소. (문맥 교환 시 가장 중요하게 저장해야 할 값)
3. **CPU 레지스터 (CPU Registers):** 누산기, 인덱스 레지스터, 스택 포인터 등. 인터럽트 발생 시 현재 계산 값을 잃지 않기 위해 저장한다.
4. **CPU 스케줄링 정보:** 프로세스 우선순위, 스케줄링 큐에 대한 포인터 등.
5. **메모리 관리 정보:** 기준(Base) 및 한계(Limit) 레지스터 값, 페이지 테이블 정보.
6. **계정 정보 (Accounting Info):** CPU 사용 시간, 시간 제한, 프로세스 번호(PID).
7. **입출력 상태 정보:** 이 프로세스에 할당된 입출력 장치 목록, 열린 파일 목록(File Descriptors).

> **문맥 교환(Context Switch)의 실체:**
> 문맥 교환이란 결국 **"실행 중인 프로세스의 PCB에 현재 상태를 저장하고, 다음에 실행할 프로세스의 PCB에서 상태를 읽어와 CPU 레지스터에 복원하는 과정"**을 말한다. PCB는 커널 메모리에 있으므로 빈번한 접근은 오버헤드가 된다.
{: .block-tip }

#### 프로세스 메모리 배치 (Process Memory Layout)

프로세스가 메모리에 로드될 때, 효율적인 관리를 위해 영역을 나누어 사용한다.



- **Text (Code) 영역:**
    - 실행할 프로그램의 **코드(기계어)**가 저장되는 영역.
    - 프로세스가 실행되는 동안 크기가 변하지 않으며(Fixed), 내용을 바꿀 수 없도록 **읽기 전용(Read-Only)**으로 설정된다.
- **Data 영역:**
    - **전역 변수(Global Variable)**와 **정적 변수(Static Variable)**가 저장되는 영역.
    - 프로그램 시작 시 할당되고 종료 시 소멸한다.
- **Heap 영역:**
    - 프로그램 실행 중(Runtime)에 **동적으로 할당**되는 메모리 영역.
    - C언어의 `malloc()`, Java의 `new` 연산자가 사용하는 공간이다.
    - 메모리의 낮은 주소에서 높은 주소 방향으로 커진다 (위로 자란다).
- **Stack 영역:**
    - 함수 호출 시 임시 데이터(**매개변수, 반환 주소, 지역 변수**)를 저장하는 영역.
    - 함수가 호출되면 할당되고 리턴되면 자동으로 해제된다.
    - 메모리의 높은 주소에서 낮은 주소 방향으로 커진다 (아래로 자란다).

> **백엔드 관점: JVM 메모리 구조와의 매핑**
>
> 백엔드 개발자에게 가장 익숙한 **JVM의 Runtime Data Area**는 OS 프로세스 메모리 구조를 소프트웨어적으로 흉내 낸 것이다.
>
> - **OS Text** ≒ **JVM Method Area**: 클래스 정보와 바이트코드가 저장됨.
> - **OS Heap** ≒ **JVM Heap**: `new`로 생성된 객체(Object)가 저장되며, GC(Garbage Collection)의 대상이 됨.
> - **OS Stack** ≒ **JVM Stack**: 각 스레드마다 존재하며, 메소드 호출 시 프레임(Frame)이 쌓임.
>     - 무한 루프나 깊은 재귀 호출 시 발생하는 `StackOverflowError`는 이 영역이 꽉 찼다는 뜻이다.
>     - 객체를 너무 많이 생성하면 발생하는 `OutOfMemoryError`는 Heap 영역이 꽉 찼다는 뜻이다.
>
> **주의:** JVM 자체도 하나의 OS 프로세스이므로, OS 입장에서 보면 JVM이 쓰는 모든 메모리(Heap + Stack + Method Area)는 통째로 OS 프로세스의 **Heap** 영역(혹은 Data) 어딘가에 위치할 것이다.
{: .block-tip }

#### [Practice] 프로세스 메모리 구조의 실체: 주소 공간 탐계

우리는 이론적으로 프로세스의 메모리가 Code, Data, Heap, Stack으로 나뉜다고 배운다. 하지만 이것이 실제로 어떤 주소 번지에 놓여 있는지, 그리고 "Stack은 아래로, Heap은 위로 자란다"는 말이 무엇인지 직접 눈으로 확인해보자.

##### 1. 검증 코드 (C Language)

C언어에서 각 영역에 속하는 변수들의 메모리 주소(&)를 직접 출력하여 상대적인 위치를 비교한다.

```c
#include <stdio.h>
#include <stdlib.h>

// 1. Data 영역 (초기화된 전역 변수)
int global_var = 10;

// Code 영역을 확인하기 위한 함수
void test_function() {}

int main() {
    // 2. Stack 영역 (지역 변수)
    int stack_var1 = 100;
    int stack_var2 = 200;

    // 3. Heap 영역 (동적 할당 변수)
    int *heap_var1 = (int *)malloc(sizeof(int));
    int *heap_var2 = (int *)malloc(sizeof(int));

    printf("=== Process Memory Layout ===\n");
    
    // Code 영역 (Text)
    printf("[Code ]  Function Addr : %p\n", (void *)test_function);
    
    // Data 영역
    printf("[Data ]  Global Var Addr: %p\n", (void *)&global_var);
    
    // Heap 영역 (성장 방향 확인)
    printf("[Heap ]  Heap Var 1 Addr: %p\n", (void *)heap_var1);
    printf("[Heap ]  Heap Var 2 Addr: %p (Upward growth)\n", (void *)heap_var2);
    
    // Stack 영역 (성장 방향 확인)
    printf("[Stack]  Stack Var 1 Addr: %p\n", (void *)&stack_var1);
    printf("[Stack]  Stack Var 2 Addr: %p (Downward growth)\n", (void *)&stack_var2);

    free(heap_var1);
    free(heap_var2);

    return 0;
}
```

##### 2. 실행 결과 및 해석

```text
=== Process Memory Layout ===
[Code ]  Function Addr : 0000000000401550
[Data ]  Global Var Addr: 0000000000403010
[Heap ]  Heap Var 1 Addr: 0000000000712470
[Heap ]  Heap Var 2 Addr: 0000000000712490 (Upward growth)
[Stack]  Stack Var 1 Addr: 000000000061FE0C
[Stack]  Stack Var 2 Addr: 000000000061FE08 (Downward growth)
```

- **주소값의 배치 (Logical vs. Actual):**
  - 전통적인 논리 메모리 모델에서는 `Code < Data < Heap <<<<< Stack` 순으로 주소가 높아진다고 설명한다.
  - 하지만 실제 현대 운영체제(Windows 등)에서는 **ASLR(Address Space Layout Randomization)** 기술과 OS의 메모리 할당 정책에 따라 Stack이 Heap보다 낮은 주소 번지에 할당되기도 한다. 따라서 주소의 절대적인 수치보다는 영역 간의 상대적인 위치와 성장 방향에 주목하는 것이 타당하다.
- **성장 방향 (Heap):** 연속적인 `malloc` 호출을 통해 할당된 주소를 비교하면, 나중에 할당된 주소가 더 큰 값을 가진다. 이는 Heap이 낮은 주소에서 높은 주소 방향으로 **상향 성장(Upward growth)**함을 의미한다.
- **성장 방향 (Stack):** 지역 변수가 선언된 순서나 함수 호출에 따른 주소 변화를 비교하면, 나중에 할당된 변수의 주소가 더 작다. 이는 Stack이 높은 주소에서 낮은 주소 방향으로 **하향 성장(Downward growth)**함을 의미한다.

> **백엔드 관점: 메모리 오버플로우의 본질**
>
> **Stack Overflow**가 왜 발생할까? 무한 재귀 호출로 인해 Stack 영역이 아래로 계속 자라다가 결국 Heap 영역이나 다른 데이터 영역을 침범하려고 할 때 OS가 이를 차단하며 발생하는 에러이다. 반대로 **Heap Overflow**는 동적 할당을 너무 많이 해서 Heap이 위로 자라다가 메모리 한계를 넘어서는 현상이다. 우리가 짜는 코드 한 줄이 이 거대한 메모리 지도 위의 한 점을 찍고 있다는 사실을 기억하자.
{: .block-tip }