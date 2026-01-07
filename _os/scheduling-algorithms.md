### 스케줄링 알고리즘

CPU 스케줄러는 준비 큐에 있는 프로세스들을 효율적으로 관리하기 위해 다양한 전략을 사용한다. 각 알고리즘은 최적화하고자 하는 목표(대기 시간 최소화, 응답성 향상 등)에 따라 설계 원리가 다르다.

#### FCFS와 SJF: 기초와 최적성

가장 직관적인 방식과 이론적으로 가장 효율적인 방식의 대비다.

- **FCFS (First-Come, First-Served):** 먼저 도착한 프로세스가 먼저 CPU를 점유한다. 간단하지만, 긴 프로세스가 앞에 있으면 짧은 프로세스들이 하염없이 기다리는 **호위 효과(Convoy Effect)**가 발생한다.
- **SJF (Shortest Job First):** CPU Burst가 가장 짧은 프로세스에게 먼저 CPU를 준다. 
    - **최적성(Optimality):** 주어진 프로세스 세트에 대해 **평균 대기 시간(Average Waiting Time)을 최소화**함이 증명되어 있다.
    - **예측 불가능성:** 실제 환경에서는 다음 CPU Burst의 길이를 정확히 알 수 없다. 따라서 과거의 이력을 바탕으로 **지수 평균(Exponential Averaging)**을 내어 미래를 추측할 뿐이다.

#### 라운드 로빈 (Round Robin, RR): 시분할의 핵심

현대적인 선점형 스케줄링의 근간이다. 각 프로세스는 **시간 할당량(Time Quantum, $q$)**만큼 CPU를 쓰고 다음 프로세스에게 양보한다.

- **트레이드오프:** 성능은 $q$의 크기에 절대적으로 의존한다.
    - **$q$가 매우 크면:** FCFS와 다를 바 없어진다.
    - **$q$가 매우 작으면:** 잦은 문맥 교환으로 인한 오버헤드가 커져 배보다 배꼽이 더 커진다. (일반적으로 문맥 교환 시간보다 $q$가 훨씬 커야 한다.)

#### 우선순위 스케줄링 (Priority Scheduling)

각 프로세스에 우선순위를 부여하고, 높은 순위부터 실행한다.

- **기아 상태 (Starvation):** 낮은 우선순위의 프로세스가 무한히 CPU를 잡지 못하는 현상이 발생할 수 있다.
- **에이징 (Aging):** 이를 해결하기 위해 시간이 지날수록 대기 중인 프로세스의 우선순위를 점진적으로 높여주는 기법을 사용한다.


#### 멀티레벨 큐 (Multilevel Queue, MLQ)와 피드백 큐 (MLFQ)

현실적인 운영체제는 프로세스의 성격(포그라운드, 백엔드 등)에 따라 큐를 여러 개 운영한다.

- **MLQ:** 프로세스는 태생에 따라 특정 큐에 고정된다.
- **MLFQ (Multilevel Feedback Queue):** 프로세스가 CPU Burst 특성에 따라 **큐 사이를 이동**할 수 있다. I/O 중심 프로세스는 상위 큐로, CPU 중심 프로세스는 하위 큐로 강등시키며 응답성과 처리량의 균형을 동적으로 조절한다.


#### [Practice] Scheduling Metrics 시뮬레이션

간단한 프로세스 세트를 사용하여 FCFS와 SJF의 평균 대기 시간을 계산하고, 알고리즘 선택이 성능 지표에 미치는 영향을 확인한다.

##### 검증 코드 (Java)

프로세스의 도착 시간과 Burst Time을 입력받아 알고리즘별 평균 대기 시간을 계산하는 시뮬레이터를 구현한다.

```java
import java.util.*;

class Process {
    int id;
    int burstTime;
    Process(int id, int burstTime) {
        this.id = id;
        this.burstTime = burstTime;
    }
}

public class SchedulingMetrics {
    public static void main(String[] args) {
        List<Process> processes = Arrays.asList(
            new Process(1, 24),
            new Process(2, 3),
            new Process(3, 3)
        );

        System.out.println("--- Scheduling Metrics Analysis ---");
        calculateFCFS(new ArrayList<>(processes));
        calculateSJF(new ArrayList<>(processes));
    }

    static void calculateFCFS(List<Process> ps) {
        int waitingTime = 0;
        int totalWaiting = 0;
        System.out.print("FCFS Order: ");
        for (int i = 0; i < ps.size(); i++) {
            System.out.print("P" + ps.get(i).id + " ");
            if (i > 0) {
                waitingTime += ps.get(i-1).burstTime;
                totalWaiting += waitingTime;
            }
        }
        System.out.println("\nAvg Waiting Time: " + (double)totalWaiting / ps.size());
    }

    static void calculateSJF(List<Process> ps) {
        ps.sort(Comparator.comparingInt(p -> p.burstTime)); // Burst Time 순 정렬
        int waitingTime = 0;
        int totalWaiting = 0;
        System.out.print("SJF Order: ");
        for (int i = 0; i < ps.size(); i++) {
            System.out.print("P" + ps.get(i).id + " ");
            if (i > 0) {
                waitingTime += ps.get(i-1).burstTime;
                totalWaiting += waitingTime;
            }
        }
        System.out.println("\nAvg Waiting Time: " + (double)totalWaiting / ps.size());
    }
}
```

##### 실행 결과 및 해석

```text
--- Scheduling Metrics Analysis ---
FCFS Order: P1 P2 P3 
Avg Waiting Time: 17.0
SJF Order: P2 P3 P1 
Avg Waiting Time: 3.0
```

1. **SJF의 우위:** 동일한 작업 부하에서도 SJF가 FCFS보다 월등히 짧은 평균 대기 시간을 보여준다.
2. **알고리즘의 선택:** 하지만 현실에서 SJF를 완벽히 구현하기 어려운 이유(예측 비용 등)를 인지해야 한다.

> **백엔드 관점: 웹 서버의 스케줄링 관점 (Throughput vs Response Time)**
>
> 백엔드 개발자는 시스템의 목적에 따라 최적화 지표를 선택해야 한다.
> 1. **Nginx (Event-driven):** 적은 자원으로 수많은 연결을 처리해야 하므로 **처리량(Throughput)** 극대화에 초점이 맞춰져 있다. 
> 2. **Tomcat (Thread-per-request):** 각 요청에 대한 빠른 피드백이 중요하므로 **응답 시간(Response Time)** 관리가 핵심이다. 
> CPU 스케줄링의 RR이나 MLFQ 원리는 우리가 작성한 API가 스레드 풀에서 어떻게 실행되고, 타임아웃 처리가 어떻게 일어나는지를 이해하는 밑바탕이 된다.
{: .block-tip }