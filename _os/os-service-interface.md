### 운영체제 서비스와 인터페이스

운영체제는 사용자 프로그램과 하드웨어 사이의 중재자로서, 프로그램 실행을 위한 환경을 제공한다. 이를 위해 OS는 사용자에게 인터페이스를 제공하고, 프로그램이 OS의 기능을 사용할 수 있도록 통로를 열어준다.

#### 사용자 인터페이스 (User Interface)

사용자가 운영체제와 상호작용하는 방식은 크게 두 가지다.

- **CLI (Command-Line Interface):** 텍스트 명령어를 입력하여 시스템을 제어한다 (예: `sh`, `bash`). 서버 환경에서는 주로 CLI를 사용하므로 백엔드 개발자에게 익숙해야 한다.
- **GUI (Graphical User Interface):** 아이콘, 윈도우, 마우스 등을 사용하는 그래픽 환경이다.

#### 시스템 호출 (System Calls)

운영체제가 제공하는 서비스에 대한 **프로그래밍 인터페이스**다. 사용자 모드(User Mode)에서 실행되는 프로그램이 커널 모드(Kernel Mode)의 기능을 요청할 때 사용한다.



- **동작 원리:** 프로그램이 시스템 콜을 호출하면 **트랩(Trap)**이라는 소프트웨어 인터럽트가 발생한다. 제어권이 커널로 넘어가고, 커널은 요청을 처리한 뒤 결과를 반환하며 다시 사용자 모드로 돌아온다.
- **API와 시스템 콜의 관계:** 개발자가 어셈블리어로 직접 시스템 콜을 호출하는 것은 매우 복잡하고 이식성이 낮다. 따라서 대부분의 언어는 시스템 콜을 캡슐화한 **API(Application Programming Interface)**를 제공한다.

**Java에서의 실행 흐름 예시**

Java 프로그램이 디스크의 파일을 읽을 때의 흐름은 다음과 같다.

1. **Java Code:** `new FileInputStream("data.txt")` 호출 (Java API)
2. **JVM (Native Method):** 내부적으로 `open()`과 관련된 C 라이브러리 함수 호출
3. **JNI (Java Native Interface):** 운영체제의 시스템 호출 인터페이스(System Call Interface)에 접근
4. **Kernel:** 실제 디스크 컨트롤러를 제어하여 파일 읽기 수행

#### [Practice] 추상화의 가면을 벗기다: strace를 통한 시스템 콜 추적

우리는 Java 코드를 짤 때 `File.read()`나 `println()`이 당연하게 작동한다고 생각한다. 하지만 JVM 밑바닥에서는 운영체제 커널에게 자원을 요청하는 치열한 **시스템 콜(System Call)**이 발생하고 있다. 리눅스의 `strace` 도구를 사용해 이 추상화의 실체를 확인해 본다.

##### 검증 시나리오 (Shell Script)

간단한 문자열을 출력하는 Java 프로그램과 파일을 읽는 `cat` 명령어를 대상으로, 실행 시점에 발생하는 시스템 콜(`openat`, `read`, `write` 등)을 필터링하여 추적한다.

```bash
# 1. 테스트용 Java 코드 작성 및 컴파일
echo 'public class HelloOS {
    public static void main(String[] args) {
        System.out.println("Hello, OS Kernel!");
    }
}' > HelloOS.java
javac HelloOS.java

# 2. strace를 사용하여 Java 실행 시 발생하는 시스템 콜 추적
# -e: 특정 시스템 콜만 필터링 (write만 추적해보기)
# -f: 모든 스레드(자식 스레드 포함)의 시스템 콜 확인
echo "=== Tracing Java System Calls (write) ==="
strace -e write -f java HelloOS

# 3. cat 명령어로 파일 읽기 시 발생하는 전체 흐름 확인 (open, read, write)
echo "This is a test file." > test.txt
echo -e "\n=== Tracing cat command (openat, read, write) ==="
strace -e openat,read,write cat test.txt
```

##### 실행 결과 및 해석

```text
=== Tracing Java System Calls (write) ===
... (JVM 초기화 과정의 write 생략) ...
write(1, "Hello, OS Kernel!\n", 18)     = 18
Hello, OS Kernel!
+++ exited with 0 +++

=== Tracing cat command (openat, read, write) ===
openat(AT_FDCWD, "test.txt", O_RDONLY)  = 3
read(3, "This is a test file.\n", 131072) = 21
write(1, "This is a test file.\n", 21)  = 21
This is a test file.
read(3, "", 131072)                     = 0
+++ exited with 0 +++
```

-   **`write(1, "Hello...", ...)`**: Java의 `System.out.println`은 결국 파일 디스크립터 1번(표준 출력)에 데이터를 쓰는 `write` 시스템 콜로 변환된다.
-   **`openat(..., "test.txt", ...)`**: 파일을 읽기 전, 커널에게 파일 주소(Path)를 넘겨주고 접근 권한을 확인받는 과정이다.
-   **의의**: 우리가 사용하는 모든 라이브러리와 프레임워크는 결국 OS가 제공하는 이 유한한 시스템 콜 인터페이스 위에서 춤추고 있다. 시스템 콜의 빈도와 효율을 이해하는 것이 곧 **고성능 백엔드 엔지니어링**의 시작이다.

> **백엔드 관점: I/O 성능 최적화의 원리**
>
> **Buffered I/O**가 왜 빠를까? 매번 `write` 시스템 콜을 호출하는 대신, 유저 영역의 버퍼에 데이터를 모았다가 한 번에 시스템 콜을 날리기 때문이다. 시스템 콜은 유저 모드에서 커널 모드로의 **문맥 교환(Context Switch)**이라는 비싼 비용을 치르기 때문에, 호출 횟수를 줄이는 것이 성능 최적화의 핵심이다.
{: .block-tip }

#### 시스템 호출의 파라미터 전달 방식

시스템 콜을 할 때, 운영체제에 정보(예: 읽으려는 파일 이름, 버퍼의 크기 등)를 넘겨줘야 한다. 단순한 레지스터만으로는 부족할 수 있어 다음과 같은 3가지 방식을 사용한다.



1. **레지스터 이용:** 파라미터가 소수일 때, CPU 레지스터에 직접 값을 담아 전달한다. 가장 빠르다.
2. **블록/테이블 메모리 주소 전달:** 파라미터가 많거나 데이터가 큰 경우(배열 등), 메모리 블록에 데이터를 저장하고 **그 블록의 시작 주소**만 레지스터에 담아 전달한다. (리눅스 등 대부분의 OS가 선호하는 방식)
3. **스택(Stack) 활용:** 프로그램이 파라미터를 스택에 `push`하고, OS가 스택에서 `pop`하여 가져간다.

#### 주요 시스템 호출 유형

- **프로세스 제어 (Process Control):** `fork()`(생성), `exit()`(종료), `wait()`(대기).
- **파일 조작 (File Manipulation):** `open()`, `read()`, `write()`, `close()`.
- **장치 관리 (Device Management):** 장치 요청/해제, 읽기/쓰기.
- **정보 유지 (Information Maintenance):** `time()`(시간 확인), 시스템 데이터 설정.
- **통신 (Communication):** `pipe()`(파이프), `shmget()`(공유 메모리), `socket()`(소켓 생성).

> **백엔드 관점: 시스템 콜 비용과 버퍼링(Buffering)**
>
> 백엔드 성능 튜닝에서 "시스템 콜을 줄여라"라는 말이 나오는 기술적 근거가 바로 이 시스템 콜 메커니즘에 있다.
>
> 1. **비용의 원인:** 시스템 콜은 사용자 모드와 커널 모드를 오가는 **문맥 교환(Context Switching)** 비용이 발생하므로 단순 연산보다 훨씬 비싼 작업이다.
> 2. **버퍼링의 이유:** Java에서 `FileOutputStream` 대신 `BufferedOutputStream`을 사용하는 이유가 여기에 있다.
>     - `write()`를 호출할 때마다 매번 시스템 콜(디스크 I/O 요청)을 날리면 성능이 끔찍하게 저하된다.
>     - 대신, 사용자 영역의 버퍼(메모리)에 데이터를 모아두었다가 버퍼가 찼을 때 **한 번의 시스템 콜**로 뭉텅이 데이터를 보내는 것이 핵심 최적화 전략이다.
{: .block-tip }