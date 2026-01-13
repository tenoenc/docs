## 컴퓨터 시스템의 기초

### 하드웨어 구조
#### CPU
##### ALU
##### 제어 장치 (Control Unit)
##### 레지스터 (Register)
#### 메모리
##### 휘발성 (Volatility)
##### 주소 지정 (Addressing)
##### 메모리 정렬 (Alignment)
#### 저장장치
##### HDD
##### SSD
#### I/O 컨트롤러
##### 로컬 버퍼
##### 제어 레지스터
##### 상태 레지스터

### 시스템 버스
#### 버스 분류
##### 데이터 버스
##### 주소 버스
##### 제어 버스
#### 버스 제어
##### 버스 중재 (Arbitration)
##### 동기식 전송
##### 비동기식 전송
#### 현대적 인터페이스
##### PCIe (Point-to-Point)

### 명령어 집합 구조 (ISA)
#### 명령어 구성
##### Opcode
##### Operand
#### 주소 지정 방식
##### 직접 주소 지정
##### 간접 주소 지정
##### 레지스터 주소 지정
##### 변위 주소 지정
#### 아키텍처 설계
##### CISC
##### RISC
##### 폰 노이만 구조
##### 하버드 구조

### 운영체제 실행 구조
#### 이중 모드 (Dual Mode)
##### 사용자 모드
##### 커널 모드
##### 모드 비트 (Mode Bit)
#### 실행 보호
##### 특권 명령 (Privileged Instruction)
##### 메모리 보호
##### I/O 보호
#### 시스템 초기화
##### BIOS/UEFI
##### POST (Power-On Self-Test)
##### 부트로더 (Bootloader)

### 커널 아키텍처
#### 모놀리식 커널 (Monolithic Kernel)
#### 마이크로커널 (Microkernel)
#### 계층형 구조 (Layered Structure)
#### 혼합 커널 (Hybrid Kernel)

### 시스템 인터페이스
#### 시스템 콜 (System Call)
##### 시스템 콜 테이블
##### 매개변수 전달 방식
#### 소프트웨어 인터럽트
##### 트랩 (Trap)
##### 인터럽트 벡터 (Interrupt Vector)
#### 가상화 및 추상화
##### 가상 머신 (Virtual Machine)
##### 컨테이너 (Container)

## 프로세서와 실행 제어

### 프로세스 (Process)
#### 프로세스 구조
##### PCB (Process Control Block)
##### 코드 영역 (Text)
##### 데이터 영역 (Data)
##### 힙 영역 (Heap)
##### 스택 영역 (Stack)
#### 프로세스 상태
##### New (생성)
##### Ready (준비)
##### Running (실행)
##### Waiting (대기)
##### Terminated (종료)
#### 프로세스 관리
##### 복제 (fork)
##### 교체 (exec)
##### 대기 (wait)
##### 종료 (exit)
##### 좀비 프로세스 (Zombie)
##### 고아 프로세스 (Orphan)

### 스레드 (Thread)
#### 스레드 추상화
##### TCB (Thread Control Block)
##### 스레드 로컬 저장소 (TLS)
#### 멀티스레딩 구현
##### 사용자 수준 스레드
##### 커널 수준 스레드
#### 멀티스레딩 모델
##### Many-to-One
##### One-to-One
##### Many-to-Many
#### 스레드 풀 (Thread Pool)

### 실행 제어 메커니즘
#### 문맥 교환 (Context Switch)
##### 트리거 이벤트
##### 상태 보존 및 복구
##### 오버헤드 (Overhead)
#### 인터럽트 (Interrupt)
##### 하드웨어 인터럽트
##### 소프트웨어 인터럽트 (Trap)
##### 인터럽트 벡터 (Vector)
##### ISR (Service Routine)
#### 모드 전환 (Mode Switch)
##### 유저-커널 전환
##### 문맥 교환과의 차이

### 프로세스 간 통신 (IPC)
#### 통신 모델
##### 공유 메모리 (Shared Memory)
##### 메시지 전달 (Message Passing)
#### 메시지 전달 상세
##### 직접 통신
##### 간접 통신 (Mailbox)
#### 데이터 통로
##### 익명 파이프 (Pipe)
##### 명명된 파이프 (FIFO)
##### 메시지 큐 (Message Queue)
#### 원격 통신
##### 소켓 (Socket)
##### RPC (Remote Procedure Call)
#### 동기화 신호
##### 시그널 (Signal)

## CPU 스케줄링 및 병렬성

### 스케줄링 기초
#### 성능 지표
##### 이용률 (Utilization)
##### 처리량 (Throughput)
##### 소요 시간 (Turnaround Time)
##### 대기 시간 (Waiting Time)
##### 응답 시간 (Response Time)
#### 스케줄링 원칙
##### 선점 (Preemptive)
##### 비선점 (Non-preemptive)
#### 디스패처 (Dispatcher)
##### 디스패치 지연 (Latency)

### 스케줄링 알고리즘
#### FCFS (First-Come, First-Served)
#### SJF (Shortest Job First)
##### SRTF (Shortest Remaining Time First)
#### 라운드 로빈 (Round Robin)
##### 타임 슬라이스 (Time Slice)
#### 우선순위 스케줄링
##### 기아 현상 (Starvation)
##### 에이징 (Aging)
#### 다단계 큐 (Multilevel Queue)
#### 다단계 피드백 큐 (Multilevel Feedback Queue)

### 다중 처리기 스케줄링
#### 처리기 구성
##### 비대칭 다중 처리 (Asymmetric)
##### 대칭 다중 처리 (SMP)
#### 친화도 및 균형
##### 프로세서 친화도 (Affinity)
##### 부하 균형 (Load Balancing)
##### Push vs Pull Migration
#### 하드웨어 스레딩
##### 멀티코어 프로세서
##### 하드웨어 멀티스레딩 (SMT)
##### 메모리 스톨 (Memory Stall)

### 실시간 스케줄링 (Real-Time)
#### 성능 제약
##### 인터럽트 지연 (Latency)
##### 디스패치 지연 (Latency)
#### 실시간 알고리즘
##### Rate Monotonic (RM)
##### Earliest Deadline First (EDF)
#### 우선순위 결함
##### 우선순위 역전 (Inversion)
##### 우선순위 상속 (Inheritance)

### 스케줄링 분석 및 평가
#### 분석 모델
##### 결정론적 모델링
##### 큐잉 모델 (Little's Law)
#### 검증 기법
##### 시뮬레이션
##### 구현 및 측정

## 프로세스 동기화 및 상호 배제

### 동기화 기초
#### 경쟁 조건 (Race Condition)
#### 임계 구역 (Critical Section)
##### 상호 배제 (Mutual Exclusion)
##### 진행 (Progress)
##### 한정된 대기 (Bounded Waiting)
#### Peterson의 해결안

### 하드웨어 동기화
#### 원자적 명령 (Atomicity)
#### Test-and-Set
#### Compare-and-Swap (CAS)
#### 메모리 장벽 (Memory Barrier)

### 동기화 도구
#### 뮤텍스 (Mutex Lock)
##### 바쁜 대기 (Busy Waiting)
##### 스핀락 (Spinlock)
#### 세마포어 (Semaphore)
##### 카운팅 세마포어
##### 이진 세마포어
##### Wait (P) / Signal (V)
#### 모니터 (Monitor)
##### 조건 변수 (Condition Variable)
##### Entry Queue
#### 라이브니스 (Liveness)
##### 교착 상태 (Deadlock)
##### 우선순위 역전 (Priority Inversion)

### 고전적 동기화 문제
#### 유한 버퍼 문제 (Bounded-Buffer)
#### 독자-저자 문제 (Readers-Writers)
#### 식사하는 철학자 문제 (Dining-Philosophers)

### 교착 상태 (Deadlock)
#### 발생 조건
##### 상호 배제
##### 점유 및 대기
##### 비선점
##### 순환 대기
#### 자원 할당 그래프 (RAG)
#### 처리 전략
##### 예방 (Prevention)
##### 회피 (Avoidance)
##### 탐지 (Detection)
##### 복구 (Recovery)
##### 무시 (Ostrich Algorithm)
#### 회피 알고리즘
##### 안전 상태 (Safe State)
##### 은행원 알고리즘 (Banker's Algorithm)
#### 복구 매커니즘
##### 자원 선점
##### 프로세스 종료
##### 희생자 선정 (Victim Selection)

## 메모리 관리 전략

### 메모리 관리 기초
#### 주소 공간
##### 논리적 주소 (Logical Address)
##### 물리적 주소 (Physical Address)
##### MMU (Memory Management Unit)
#### 주소 바인딩 (Binding)
##### 컴파일 시간 바인딩
##### 로드 시간 바인딩
##### 실행 시간 바인딩
#### 메모리 보호
##### 재배치 레지스터 (Relocation)
##### 한계 레지스터 (Limit)

### 연속 메모리 할당
#### 할당 방식
##### 고정 분할 (Fixed Partitioning)
##### 가변 분할 (Variable Partitioning)
#### 메모리 단편화 (Fragmentation)
##### 내부 단편화
##### 외부 단편화
##### 압축 (Compaction)
#### 동적 할당 알고리즘
##### 최초 적합 (First-fit)
##### 최적 적합 (Best-fit)
##### 최악 적합 (Worst-fit)

### 페이징 (Paging)
#### 페이징 메커니즘
##### 프레임 (Frame)
##### 페이지 (Page)
##### 페이지 테이블 (Page Table)
#### 하드웨어 지원
##### PTBR (Page Table Base Register)
##### TLB (Translation Look-aside Buffer)
##### 히트 비율 (Hit Ratio)
#### 페이징 보호
##### 유효-무효 비트 (Valid-Invalid Bit)
##### 보호 비트 (Protection Bit)

### 페이지 테이블 구조
#### 계층적 페이징 (Hierarchical Paging)
#### 해시 페이지 테이블 (Hashed Page Table)
#### 역 페이지 테이블 (Inverted Page Table)

### 세그멘테이션 (Segmentation)
#### 세그먼트 테이블 (Segment Table)
#### 세그먼트 보호 및 공유
#### 세그멘테이션-페이징 혼합 (Hybrid)

### 가상 메모리 관리
#### 요구 페이징 (Demand Paging)
##### 페이지 폴트 (Page Fault)
##### 유효-무효 비트 (Valid-Invalid Bit)
##### 참조 국부성 (Locality)
#### 페이지 교체 알고리즘
##### FIFO 교체
##### 최적 교체 (Optimal)
##### LRU 교체
##### LFU/MFU 교체
#### 프레임 관리
##### 프레임 할당 (Allocation)
##### 스래싱 (Thrashing)
##### 작업 집합 (Working Set)
##### 페이지 폴트 빈도 (PFF)

## 캐시 메모리와 일관성

### 캐시 아키텍처
#### 캐시 계층 구조
##### L1 캐시
##### L2 캐시
##### L3 캐시 (LLC)
#### 캐시 사상 (Mapping)
##### 직접 사상 (Direct Mapping)
##### 완전 연관 사상 (Fully Associative)
##### 집합 연관 사상 (Set Associative)

### 지역성과 성능
#### 지역성 (Locality)
##### 시간 지역성 (Temporal)
##### 공간 지역성 (Spatial)
#### 캐시 상태
##### 캐시 적중 (Cache Hit)
##### 캐시 미스 (Cache Miss)
#### 캐시 미스 분류
##### 강제 미스 (Compulsory/Cold)
##### 용량 미스 (Capacity)
##### 충돌 미스 (Conflict)

### 캐시 일관성 (Coherence)
#### 데이터 불일치 원인
#### 일관성 프로토콜
##### 스누핑 (Snooping)
##### 디렉토리 기반 (Directory-based)
#### MESI 프로토콜
##### Modified (수정)
##### Exclusive (독점)
##### Shared (공유)
##### Invalid (무효)

### 쓰기 정책 (Write Policy)
#### 업데이트 시점
##### Write-through
##### Write-back
#### 쓰기 미스 처리
##### Write-allocate
##### No-write-allocate

## 저장장치 및 파일 시스템

### 보조 기억 장치 관리
#### 디스크 구조
##### 물리적 구조 (Platter, Arm, Head)
##### 논리적 블록 주소 (LBA)
#### 디스크 스케줄링
##### FCFS 스케줄링
##### SSTF 스케줄링
##### SCAN / C-SCAN
##### LOOK / C-LOOK
#### 플래시 메모리 (SSD) 관리
##### 가비지 컬렉션 (Garbage Collection)
##### 웨어 레벨링 (Wear Leveling)
##### TRIM 명령어

### 파일 시스템 인터페이스
#### 파일 개념
##### 파일 속성 (Metadata)
##### 파일 연산 (Create, Open, Write, Read)
##### 파일 접근 제어 (Access Control)
#### 디렉토리 구조
##### 1단계 디렉토리
##### 2단계 디렉토리
##### 트리 구조 디렉토리
##### 비순환 그래프 디렉토리 (DAG)
#### 파일 시스템 마운팅 (Mounting)

### 파일 시스템 구현
#### 파일 할당 방식
##### 연속 할당 (Contiguous Allocation)
##### 연결 할당 (Linked Allocation)
##### 색인 할당 (Indexed Allocation)
##### i-node 구조 (Index Node)
#### 가용 공간 관리
##### 비트 벡터 (Bit Vector)
##### 연결 리스트 (Linked List)
##### 그룹화 (Grouping)
#### 디렉토리 구현
##### 선형 리스트 (Linear List)
##### 해시 테이블 (Hash Table)

### 파일 시스템 최적화 및 신뢰성
#### 구조적 추상화
##### 가상 파일 시스템 (VFS)
#### 일관성 제어
##### 저널링 (Journaling)
##### 일관성 체크 (fsck)
#### 성능 및 무결성 기법
##### 로그 구조 파일 시스템 (LFS)
##### 쓰기 시 복사 (Copy-on-Write)

### RAID 구조
#### 기본 RAID 모델
##### RAID 0 (Striping)
##### RAID 1 (Mirroring)
#### 고가용성 RAID 모델
##### RAID 5 (Distributed Parity)
##### RAID 6 (Double Parity)
##### RAID 10 (Striping + Mirroring)

## 입출력 시스템 및 버스

### I/O 하드웨어 원리
#### 하드웨어 구성 요소
##### 포트 (Port)
##### 버스 (Bus)
##### 컨트롤러 (Controller)
#### 통신 매커니즘
##### 폴링 (Polling)
##### 인터럽트 (Interrupt)
##### DMA (Direct Memory Access)
##### 입출력 채널 (I/O Channel)

### 커널 I/O 서브시스템
#### 입출력 제어 기법
##### I/O 스케줄링
##### 버퍼링 (Buffering)
##### 캐싱 (Caching)
##### 스풀링 (Spooling)
#### 성능 및 안정성
##### 장치 예약
##### 에러 처리
##### I/O 보호

### 사용자 입출력 인터페이스
#### 장치 드라이버 아키텍처
##### 블록 장치 (Block Device)
##### 문자 장치 (Character Device)
##### 네트워크 장치
#### 입출력 수행 방식
##### 동기식 I/O (Blocking)
##### 비동기식 I/O (Non-blocking)
##### vectored I/O (Scatter-Gather)

## 보호 및 보안

### 시스템 보호 (Protection)
#### 보호 원칙
##### 최소 권한 원칙
##### 보호 영역 (Domain)
#### 접근 제어 매커니즘
##### 접근 행렬 (Access Matrix)
##### ACL (Access Control List)
##### 권한 리스트 (Capability List)
#### 권한 관리
##### 역할 기반 접근 제어 (RBAC)
##### 권한 철회 (Revocation)

### 시스템 보안 (Security)
#### 위협 모델
##### 프로그램 위협 (바이러스, 웜)
##### 시스템 위협 (DOS, 침입)
#### 보안 매커니즘
##### 사용자 인증 (Authentication)
##### 암호화 (Encryption)
##### 디지털 서명 (Digital Signature)
#### 탐지 및 방어
##### 침입 탐지 시스템 (IDS)
##### 방화벽 (Firewall)

### 가상화 아키텍처
#### 가상화 기술
##### 하이퍼바이저 (Hypervisor)
##### 전가상화 vs 반가상화
#### 컨테이너 기술
##### 네임스페이스 (Namespace)
##### Cgroups (Control Groups)
#### 하드웨어 보안 지원
##### 신뢰 실행 환경 (TEE)
##### Intel SGX / ARM TrustZone