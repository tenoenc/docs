### 프로세스 연산

대부분의 시스템에서 프로세스는 동시에 실행될 수 있으며, 동적으로 생성되거나 삭제될 수 있어야 한다. 이를 위해 운영체제는 프로세스 생성 및 종료를 위한 메커니즘을 제공한다.

#### 프로세스 생성 (Process Creation)

실행 중인 프로세스는 다른 프로세스를 생성할 수 있다. 이때 생성하는 프로세스를 **부모 프로세스(Parent Process)**라 하고, 생성된 프로세스를 **자식 프로세스(Child Process)**라 한다. 이 과정이 반복되며 프로세스들은 **트리(Tree)** 구조를 형성한다.

운영체제는 각 프로세스를 식별하기 위해 고유한 정수 값인 **PID(Process Identifier)**를 부여한다.



**유닉스/리눅스에서의 생성 과정**

1.  **`fork()` 시스템 콜:**
    - 부모 프로세스는 `fork()`를 호출하여 자신과 똑같은 복제본을 생성한다.
    - 자식 프로세스는 부모의 주소 공간(Code, Data, Heap, Stack) 복사본을 물려받는다.
    - **반환값:** 부모에게는 자식의 PID를, 자식에게는 `0`을 반환하여 서로를 구분하게 한다.
2.  **`exec()` 시스템 콜:**
    - `fork()` 직후 자식 프로세스는 부모와 동일한 프로그램을 실행한다.
    - 자식 프로세스가 다른 프로그램을 실행하려면 `exec()` 시스템 콜을 호출한다.
    - 이 호출은 프로세스의 메모리 공간을 새로운 프로그램의 이진 파일(Binary)로 덮어씌우고, 처음부터 실행을 시작한다.

> **`fork()` 후 `exec()` 구조의 장점:**
> 단순히 프로세스를 새로 만드는 함수 하나(`spawn`)가 아니라 두 단계로 나눈 이유는 유연성 때문이다. `fork()` 후 `exec()` 하기 전 단계에서 자식 프로세스는 파일 디스크립터를 조작하거나 입출력을 재설정(Redirection)할 수 있다.
{: .block-tip }

#### [Practice] fork()와 exec(): 프로세스의 탄생과 변신

Unix/Linux 환경에서 프로세스는 **복제(`fork`)**를 통해 태어나고, **대체(`exec`)**를 통해 다른 프로그램으로 변신한다. 이 과정을 코드로 추론해 본다.

##### 검증 코드 (C Language)

`fork()` 시스템 콜의 반환값에 따라 부모와 자식의 실행 경로가 나뉘는 메커니즘을 확인한다.

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main() {
    pid_t pid;

    // 1. 새로운 프로세스 복제 시도
    pid = fork();

    if (pid < 0) { // 에러 처리
        fprintf(stderr, "Fork Failed");
        return 1;
    } 
    else if (pid == 0) { // [자식 프로세스 영역]
        printf("[Child] I am the child. PID: %d, Parent PID: %d\n", getpid(), getppid());
        printf("[Child] I'm going to transform into 'ls -l' command...\n");
        
        // 2. 다른 프로그램(ls)으로 실행 이미지를 덮어씌움 (exec)
        execlp("/bin/ls", "ls", "-l", NULL);
        
        // exec 성공 시 아래 코드는 절대 실행되지 않음
        printf("This should not be printed if exec succeeds.\n");
    } 
    else { // [부모 프로세스 영역]
        printf("[Parent] I am the parent. Child PID is %d\n", pid);
        
        // 자식이 종료될 때까지 대기 (동기화)
        wait(NULL);
        printf("[Parent] Child has finished its execution.\n");
    }

    return 0;
}
```

##### 실행 결과 및 해석

```text
[Parent] I am the parent. Child PID is 4312
[Child] I am the child. PID: 4312, Parent PID: 4311
[Child] I'm going to transform into 'ls -l' command...
total 20
-rwxrwxrwx 1 teno teno 16304 Jan  7 09:03 fork-exec
-rwxrwxrwx 1 teno teno  1115 Jan  7 09:03 fork-exec.c
[Parent] Child has finished its execution.
```

-   **PID 분기:** `fork()`는 한 번 호출되지만, 반환값은 두 번(부모에게는 자식의 PID, 자식에게는 0) 돌아옵니다. 이를 통해 동일한 코드 내에서 부모와 자식의 역할을 구분한다.
-   **메모리 공간의 대체:** 자식 프로세스에서 `execlp()`를 호출하는 순간, 현재 실행 중이던 C 프로그램의 메모리는 `ls` 프로그램의 이미지로 완전히 덮어씌워집니다.

#### 프로세스 종료 (Termination)

프로세스가 마지막 문장의 실행을 끝내고 `exit()` 시스템 콜을 호출하면 운영체제에 삭제를 요청한다.

-   **자원 반납:** 프로세스에 할당된 물리 메모리, 열린 파일, I/O 버퍼 등의 자원이 OS로 반환된다.
-   **상태 전달:** 종료된 프로세스는 부모 프로세스에게 상태 값(Status Value)을 반환할 수 있다. 부모는 `wait()` 시스템 콜을 통해 이 값을 받는다.

#### 좀비(Zombie)와 고아(Orphan) 프로세스

프로세스의 종료와 부모의 `wait()` 호출 시점의 불일치로 인해 특수한 상태의 프로세스가 발생할 수 있다.

-   **좀비 프로세스 (Zombie Process):**
    -   자식 프로세스가 종료(`exit()`)되었으나, 부모 프로세스가 아직 `wait()`를 호출하지 않은 상태.
    -   프로세스의 자원은 대부분 해제되었으나, 프로세스 테이블에 **PID와 종료 상태(Exit Status)** 등 최소한의 정보가 남아 있다.
    -   모든 프로세스는 종료 시 잠시 좀비 상태가 되며, 부모가 `wait()`를 호출하는 순간 제거된다. 만약 부모가 `wait()`를 안 하고 방치하면 좀비가 계속 쌓여 **PID 고갈** 문제가 발생할 수 있다.
-   **고아 프로세스 (Orphan Process):**
    -   부모 프로세스가 자식보다 먼저 종료(`exit()`)된 경우.
    -   자식 프로세스는 고아가 되며, 리눅스 시스템에서는 `init` 프로세스(또는 `systemd`, PID 1)가 새로운 부모가 된다.
    -   `init` 프로세스는 주기적으로 `wait()`를 호출하여 고아들의 종료 처리를 담당한다.

> **백엔드 관점: Copy-on-Write (COW)와 Redis**
>
> `fork()` 시스템 콜 설명 시 "부모의 메모리를 복사한다"고 했지만, 실제 물리 메모리를 무식하게 다 복사하면 프로세스 생성 시간이 너무 오래 걸릴 것이다. 현대 OS는 **Copy-on-Write(COW)**라는 최적화 기법을 사용한다.
>
> 1.  **동작 원리:**
>     - `fork()` 시 부모와 자식의 페이지 테이블은 **동일한 물리 메모리 프레임**을 가리킵니다(공유). 단, 해당 페이지들은 **읽기 전용(Read-Only)**으로 표시된다.
>     - 읽기(Read) 작업만 수행할 때는 메모리 복사가 일어나지 않는다.
>     - 부모나 자식 중 누군가 페이지에 **쓰기(Write)**를 시도하면, 그때 하드웨어가 트랩을 발생시키고 OS는 해당 페이지만 물리적으로 복사하여 수정할 수 있게 해준다.
>
> 2.  **Redis의 RDB 스냅샷 (BGSAVE):**
>     - 싱글 스레드인 Redis가 멈추지 않고(Non-blocking) 현재 메모리 상태를 디스크에 저장(Backup)할 수 있는 비결이 바로 **`fork()`와 `COW`**이다.
>     - Redis는 `fork()`를 통해 자식 프로세스를 만듭니다. 덕분에 메모리 사용량이 순간적으로 2배가 되지 않는다.
>     - 자식 프로세스가 디스크에 데이터를 쓰는 동안, 부모 프로세스는 계속 클라이언트의 쓰기 요청을 처리한다.
>     - 이때 변경된 데이터 페이지만 COW에 의해 복사되므로, 실제 추가 메모리 사용량은 **"스냅샷 도중 변경된 데이터의 양"**에 비례한다.
{: .block-tip }

#### [Practice] Zombie Process: 끝나지 않은 이야기

프로세스가 종료되었지만, 부모 프로세스가 그 종료 상태를 확인(`wait`)하지 않아 시스템의 프로세스 테이블에 정보가 남아 있는 상태를 **좀비 프로세스(Zombie Process)**라고 한다.

##### 검증 코드 (C Language)

자식 프로세스는 즉시 종료시키고, 부모 프로세스는 `wait()`를 호출하지 않은 채 긴 시간 잠들게(`sleep`) 하여 의도적으로 좀비를 생성한다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid > 0) {
        // [부모] 30초 동안 아무것도 안 하고 대기 (wait 호출 안 함)
        printf("[Parent] I'm sleeping for 30s. Check 'ps -ef | grep defunct' now!\n");
        sleep(30);
        printf("[Parent] Waking up and exiting...\n");
    } 
    else if (pid == 0) {
        // [자식] 생성되자마자 바로 종료
        printf("[Child] I'm exiting now to become a zombie...\n");
        exit(0);
    }

    return 0;
}
```

##### 실행 결과 및 해석

```text
# 프로그램을 백그라운드로 실행 후 다른 터미널에서 확인 시
$ ./zombie_test &
[Parent] I'm sleeping for 30s. Check 'ps -ef | grep defunct' now!
[Child] I'm exiting now to become a zombie...

$ ps -el | grep Z
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
1 Z  1000    4471    4470  0  80   0 -     0 -      pts/0    00:00:00 zombie-process <defunct>
```

-   **실태 확인:** 자식 프로세스는 `exit(0)`을 호출했음에도 불구하고, `ps` 명령어에서 `<defunct>`(죽은 상태) 혹은 `Z` 상태로 표시된다.
-   **시사점:** 자원은 반납했지만 '이름(Process Table Entry)'은 지워지지 않은 상태이다. 부모가 `wait()`를 통해 종료 코드를 읽어가는 순간 좀비는 사라진다. 백엔드 서버에서 좀비 프로세스가 누적되면 새로운 프로세스를 생성할 수 없는 고갈 상태에 빠질 수 있다.