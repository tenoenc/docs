### 커널 데이터 구조

운영체제 커널도 결국은 C언어로 작성된 거대한 소프트웨어다. 따라서 자원(프로세스, 메모리, 파일)을 효율적으로 관리하기 위해 우리가 알고리즘 시간에 배우는 다양한 자료구조를 내부적으로 구현하여 사용한다.

#### 리스트, 스택, 큐 (Lists, Stacks, and Queues)

가장 기본적이고 빈번하게 사용되는 자료구조다.

- **연결 리스트 (Linked List):** 커널은 배열보다는 연결 리스트를 선호한다. 크기가 동적으로 변하고 중간 삽입/삭제가 빈번하기 때문이다. 특히 리눅스 커널은 **이중 원형 연결 리스트(Circular Doubly Linked List)** 형태의 `struct list_head`를 사용하여 프로세스 목록, 인터럽트 핸들러 목록 등을 관리한다.
- **스택 (Stack):** 함수 호출 시 복귀 주소와 지역 변수를 저장하는 **커널 스택**에 사용된다.
- **큐 (Queue):** 순서대로 처리해야 하는 작업들에 사용된다. CPU를 기다리는 **준비 큐(Ready Queue)**, I/O 장치를 기다리는 **대기 큐(Wait Queue)**가 대표적이다.

##### [Practice] Intrusive (Kernel) vs Non-Intrusive (Java) Lists

리눅스 커널의 연결 리스트는 독특하다. 데이터가 노드를 감싸는 것이 아니라, **노드가 데이터 구조체 안에 심겨져(Intrusive) 있다.** 반면 Java의 `LinkedList`는 데이터 객체를 별도의 노드 객체가 가리키는 방식(Non-Intrusive)이다.

이 구조적 차이가 **캐시 적중률(Cache Hit)**에 어떤 영향을 미치는지 의사 코드로 비교해보자.

###### 구조 비교 (Pseudo-code)

커널 방식은 리스트 순회 시 데이터가 바로 옆에 붙어 있어 캐시 지역성이 좋지만, Java 방식은 노드와 데이터가 힙(Heap) 메모리 상에 흩어져 있어 **참조 비용(Pointer Chasing)**이 크다.

```java
// === 1. Linux Kernel Style (Intrusive) ===
// 데이터 구조체 '안에' 연결 고리가 포함됨
struct Task {
    int id;
    long vruntime;
    struct list_head list; // Prev, Next 포인터가 여기에 직접 존재
};

// 장점: list 포인터를 따라가면 데이터(id, vruntime)가 같은 캐시 라인에 있을 확률 높음.
// 메모리: [ Task { id | vruntime | list(prev, next) } ] -> [ Task ... ]


// === 2. Java Style (Non-Intrusive) ===
// 연결 고리(Node)가 데이터(Task)를 '가리킴'
class LinkedList<E> {
    Node<E> head;
}

class Node<E> {
    E item;      // 실제 데이터는 다른 메모리 주소에 있음 (참조)
    Node<E> next;
    Node<E> prev;
}

// 단점: Node를 읽고 -> item 주소로 점프 -> 실제 데이터 읽기 (2번 접근)
// 메모리: [ Node ] --(jump)--> [ Task Object ]
//          |
//        (next)
//          |
//          v
//       [ Node ] --(jump)--> [ Task Object ]
```

###### 해석 및 백엔드 시사점

-   **Linux Kernel:** `struct list_head`를 따라가면 바로 그 구조체(`struct Task`)의 메모리 영역에 도달한다. 즉, **한 번의 메모리 접근**으로 다음 데이터 처리가 가능하다.
-   **Java LinkedList:** `Node`를 찾은 뒤, 그 노드가 가리키는 `Data` 객체를 찾기 위해 **또 한 번의 점프**가 필요하다. (`Double Indirection`).
-   **결론:** 고성능 시스템에서는 객체를 감싸는 포장(Wrapper)조차 비용이다. Java에서 성능이 중요할 때 `LinkedList`보다 `ArrayList`(연속된 메모리)를 선호하는 이유가 바로 이 **캐시 지역성(Locality)** 때문이다.

#### 트리 (Trees)

검색 성능이 중요한 곳에는 트리가 사용된다. 단순한 이진 검색 트리(BST)는 데이터 입력 순서에 따라 편향(Skewed)되어 최악의 경우 $O(n)$의 성능을 보일 수 있으므로, 커널은 스스로 균형을 맞추는 트리를 주로 사용한다.

##### 이진 검색 트리와 RB-Tree (Red-Black Tree)

- **RB-Tree:** 자가 균형 이진 탐색 트리(Self-Balancing BST)의 일종이다. 데이터의 삽입/삭제 시 트리의 높이(Depth)를 자동으로 조절하여 최악의 경우에도 검색, 삽입, 삭제 시간을 $O(\log n)$으로 보장한다.
- **사용 사례:**
    - **CPU 스케줄러:** 리눅스의 기본 스케줄러인 **CFS(Completely Fair Scheduler)**는 실행 대기 중인 프로세스들을 `vruntime`(가상 실행 시간)을 키(Key)로 하여 RB-Tree에 저장한다. 가장 왼쪽 노드(가장 `vruntime`이 작은 프로세스)를 $O(1)$에 가깝게 찾아내어 CPU를 할당한다.
    - **가상 메모리 관리:** 프로세스 내의 가상 메모리 영역(VMA)을 관리할 때도 빠른 검색을 위해 사용된다.

##### [Practice] CFS Scheduler & Java TreeMap

리눅스 커널의 **CFS(Completely Fair Scheduler)**는 프로세스들을 공평하게 실행하기 위해 `vruntime`을 기준으로 정렬된 상태를 유지한다. 이때 사용하는 자료구조가 **Red-Black Tree**이다. 놀랍게도 이 알고리즘은 우리가 자주 사용하는 Java의 `TreeMap`과 동일하다.

###### 검증 코드 (Java)

Java의 `TreeMap`을 사용하여 무작위 순서로 데이터를 넣어도 내부적으로 정렬이 유지됨을 확인한다. 이는 커널이 프로세스를 `vruntime` 순으로 항상 정렬해두는 원리와 같다.

```java
import java.util.TreeMap;

public class RBTreeExample {
    public static void main(String[] args) {
        // TreeMap은 내부적으로 Red-Black Tree로 구현되어 있음
        // Key: vruntime (가상 실행 시간), Value: Process Name
        TreeMap<Integer, String> processQueue = new TreeMap<>();

        // 1. 무작위 순서로 프로세스(Task) 삽입
        // 리눅스 커널이 프로세스 상태가 변할 때마다 큐에 넣는 것과 유사
        processQueue.put(100, "Task-A");
        processQueue.put(50,  "Task-B (High Priority)");
        processQueue.put(150, "Task-C");

        System.out.println("=== Scheduler Queue (Sorted by vruntime) ===");
        
        // 2. 순회 출력 (항상 Key 기준으로 정렬되어 있음)
        // 커널은 가장 왼쪽 노드(가장 작은 vruntime)를 $O(1)$ ~ $O(\log n)$에 꺼내어 실행함
        processQueue.forEach((vruntime, task) -> {
            System.out.printf("vruntime: %3d | Process: %s\n", vruntime, task);
        });
        
        // Output:
        // vruntime:  50 | Process: Task-B (High Priority)
        // vruntime: 100 | Process: Task-A
        // vruntime: 150 | Process: Task-C
    }
}
```

###### 실행 결과 및 해석

-   **자동 정렬:** 데이터(`100`, `50`, `150`)를 무작위로 넣었지만, 출력 결과는 `50` -> `100` -> `150` 순서이다.
-   **시간 복잡도:** 커널 스케줄러가 다음에 실행할 프로세스(가장 왼쪽 노드)를 찾는 비용, Java 맵에서 특정 키를 찾는 비용 모두 **$O(\log n)$**으로 매우 효율적이다.

#### 해시 맵 (Hash Maps)

데이터의 키(Key)를 해시 함수에 통과시켜 값(Value)이 저장된 위치를 즉시 찾아내는 구조다.

- **사용 사례:** **PID(Process ID) 해시 테이블**. 시스템에 존재하는 수많은 프로세스 중 특정 PID를 가진 프로세스의 제어 블록(PCB)을 빠르게 찾아야 할 때 사용한다. 리스트로 관리한다면 PID를 찾기 위해 전체를 순회해야 하므로 매우 비효율적일 것이다.

#### 비트맵 (Bitmaps)

비트 배열(Bit Array)이라고도 하며, $n$개의 항목에 대한 상태(사용 중/비사용 중)를 $n$개의 비트로 표현하는 매우 공간 효율적인 자료구조다.

- **사용 사례:**
    - **디스크 블록 관리:** 파일 시스템에서 특정 디스크 블록이 데이터로 차 있는지(1), 비어 있는지(0)를 표시한다.
    - **PID 관리:** 어떤 PID 번호가 현재 사용 중인지 빠르게 체크하고 할당하기 위해 사용한다.

##### [Practice] Bitmap의 압도적인 공간 효율성

운영체제는 디스크 블록의 사용 여부나 PID 할당 여부 등 수만 개의 상태(True/False)를 관리해야 한다. 이때 `boolean` 배열이나 `int` 배열을 쓰지 않고 **비트맵**을 쓰는 이유는 명확하다.

###### 검증 코드 (Java)

Java의 `BitSet`과 비트 연산자(`<<`, `|`, `&`)를 사용하여, 단 하나의 `int` 변수(32bit)로 32개의 상태 플래그를 관리하는 방법을 보여준다.

```java
import java.util.BitSet;

public class BitmapExample {
    public static void main(String[] args) {
        // Case 1: Java BitSet 활용 (간편한 API)
        BitSet resourceMap = new BitSet(8); // 8비트 공간
        
        resourceMap.set(0); // 0번 블록 사용 중
        resourceMap.set(5); // 5번 블록 사용 중
        
        System.out.println("=== [1] BitSet Usage ===");
        System.out.println("Is Index 0 used? " + resourceMap.get(0));
        System.out.println("Is Index 1 used? " + resourceMap.get(1));
        System.out.println("Internal representation: " + resourceMap.toString());

        // Case 2: 원시 비트 연산 (OS 커널 스타일)
        // int 변수 하나(32bit)로 32개 플래그 관리
        int flags = 0; 

        // Set Flag (OR 연산 |)
        flags |= (1 << 2); // 2번 비트 켜기 (0000 0100)
        flags |= (1 << 7); // 7번 비트 켜기 (1000 0000)

        System.out.println("\n=== [2] Bitwise Operation ===");
        System.out.println("Flags Value (Decimal): " + flags);
        System.out.println("Flags Value (Binary) : " + Integer.toBinaryString(flags));

        // Check Flag (AND 연산 &)
        boolean isBit2Set = (flags & (1 << 2)) != 0;
        System.out.println("Is 2nd bit set? " + isBit2Set);
    }
}
```

###### 효율성 분석

-   **boolean 배열:** Java에서 `boolean`은 보통 1바이트(8비트)를 차지한다. 32개의 상태를 저장하려면 **32바이트**가 필요하다.
-   **BitSet / 비트마스크:** `int` 하나는 4바이트(32비트)이다. 즉, **4바이트**만으로 32개의 상태를 완벽하게 관리할 수 있다.
-   **결론:** 데이터가 수백만 개로 늘어날수록 비트맵 방식은 메모리 사용량을 **1/8 (약 87% 절약)**로 줄여준다. 이것이 OS가 비트맵을 사랑하는 이유이다.

> **백엔드 관점: 자료구조의 실전 활용**
>
> 이 챕터는 백엔드 개발자가 **"왜 자료구조와 알고리즘을 공부해야 하는가?"**에 대한 근본적인 대답을 준다.
> 1. **DB 인덱스:** 우리가 사용하는 MySQL의 인덱스(B+Tree)와 커널의 RB-Tree는 모두 "디스크/메모리 접근 비용을 줄이기 위한 트리 구조"라는 공통점이 있다.
> 2. **Java Collection:** Java의 `TreeMap`은 내부적으로 Red-Black Tree로 구현되어 있다. 즉, 리눅스 커널 스케줄러가 사용하는 알고리즘과 우리가 Java에서 정렬된 맵을 쓸 때 사용하는 알고리즘이 같다.
> 3. **성능 최적화:** 운영체제가 특정 상황에서 왜 느려지는지(예: 해시 충돌, 트리 리밸런싱 비용)를 이해하면, 대용량 트래픽을 처리하는 서버 애플리케이션의 성능 병목도 예측할 수 있다.
{: .block-tip }