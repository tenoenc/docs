## 커널 아키텍처

### 커널의 역할과 위치
#### 하드웨어 추상화 계층 (HAL)
#### 시스템 리소스 중재
#### 유저 공간과의 격리

### 커널 실행 모델
#### 모놀리식 커널 구조
#### 로드 가능 커널 모듈 (LKM)
##### 모듈 의존성 관리
##### 동적 로딩 및 해제

### 시스템 호출 인터페이스
#### 시스템 호출 진입 메커니즘
#### 매개변수 전달 및 검증
#### vDSO 및 시스템 콜 최적화

### 커널 예외 처리
#### 하드웨어 인터럽트 핸들링
#### 소프트웨어 인터럽트 (Softirqs)
#### 태스크렛 (Tasklets)
#### 워크 큐 (Work Queues)

### 커널 주소 공간
#### 가상 메모리 레이아웃
#### 커널 스택 구조
#### 페이지 테이블 격리 (KPTI)

### 커널 부팅 메커니즘
#### 부트 파라미터 전달
#### 초기 램 디스크 (initrd/initramfs)
#### 커널 심볼 테이블 (kallsyms)

### 커널 서브시스템 제어
#### sysctl 매개변수 튜닝
#### 커널 명령행 옵션 (cmdline)

### 커널 상태 인터페이스
#### procfs (프로세스 및 커널 정보)
#### sysfs (장치 및 커널 객체)
#### debugfs (커널 디버깅 인터페이스)

### 커널 동기화 및 락킹
#### 원자적 연산 (Atomic Operations)
#### 커널 스핀락 (Spinlocks)
#### 수면 가능한 락 (Mutex/Semaphore)
#### RCU (Read-Copy-Update)
#### 메모리 장벽 (Memory Barriers)

## 쉘 및 환경 설정

### 쉘의 역할과 분류
#### 명령 인터프리터 아키텍처
#### 로그인 쉘 vs 비로그인 쉘
#### 대화형 쉘 vs 비대화형 쉘

### 쉘 초기화 메커니즘
#### 전역 설정 파일 (/etc/profile, /etc/bashrc)
#### 사용자 설정 파일 (~/.bash_profile, ~/.bashrc)
#### 실행 시점별 로딩 시퀀스

### 환경 변수 및 변수 범위
#### 쉘 변수와 환경 변수의 차이
#### 상속 매커니즘 (export)
#### 핵심 시스템 변수 (PATH, HOME, LANG)
#### 공유 라이브러리 경로 (LD_LIBRARY_PATH)

### 쉘 확장 및 해석 프로세스
#### 명령어 대치 (Command Substitution)
#### 산술 및 중괄호 확장
#### 경로 패턴 확장 (Globbing)
#### 인용 부호 처리 (Quoting Rules)

### 표준 입출력 및 파이프라인
#### 파일 디스크립터 (0, 1, 2)
#### 입출력 리다이렉션 (Redirection)
##### 출력 덮어쓰기 및 추가 (>, >>)
##### 표준 오류 통합 처리 (2>&1)
#### 파이프라인 (Pipe) 연산 원리

### 작업 제어 (Job Control)
#### 포그라운드 및 백그라운드 실행
#### 작업 상태 제어 (jobs, fg, bg)
#### 세션 종료와 프로세스 유지 (nohup, disown)

### 쉘 스크립트 실행 구조
#### 시뱅(Shebang, #!)의 역할
#### 실행 권한과 PATH 탐색 메커니즘
#### Sourcing(. / source) vs 실행(Execution)의 차이

### 별칭 및 함수
#### 별칭(Alias) 설정 및 해제
#### 쉘 함수(Shell Function) 정의
#### 해석 우선순위 (Alias vs Function vs Built-in vs Binary)

### 쉘 사용자 편의 기능
#### 히스토리(History) 관리
#### 명령 자동 완성 (Tab Completion) 매커니즘

### 터미널 인터페이스
#### TTY (Teletypewriter) 개념
#### PTY (Pseudo-Terminal) 가상화
#### 터미널 라인 규정 (Line Discipline)
#### 제어 터미널과 세션 관리

## 파일 시스템

### 가상 파일 시스템 (VFS)
#### VFS 추상화 계층
#### 슈퍼블록 (Superblock)
#### 아이노드 (Inode) 구조
#### 디렉토리 엔트리 (Dentry) 캐시
#### 파일 객체 (File Object)

### 리눅스 디렉토리 구조 (FHS)
#### 루트 디렉토리 (/)
#### 실행 바이너리 (/bin, /sbin, /usr)
#### 시스템 설정 (/etc)
#### 가변 데이터 및 로그 (/var)
#### 임시 및 런타임 데이터 (/tmp, /run)

### 파일 유형 및 메타데이터
#### 파일 종류 식별 (File Types)
#### 특수 파일 (Socket, FIFO, Device)
#### 아이노드 메타데이터 구조
#### 파일 시간 정보 (atime, mtime, ctime)

### 링크 메커니즘
#### 하드 링크 (Hard Link)
#### 심볼릭 링크 (Symbolic Link)
##### 아이노드 참조 및 링크 카운트
##### 대상을 잃은 링크 (Dangling Link)

### 마운트 메커니즘
#### 마운트 포인트 및 트리 통합
#### 파일 시스템 테이블 (/etc/fstab)
#### 바인드 마운트 (Bind Mount)

### 리눅스 파일 시스템 종류
#### 로컬 파일 시스템 (Ext4, XFS)
#### 메모리 기반 파일 시스템 (tmpfs)
#### 네트워크 파일 시스템 (NFS)

### 가용 공간 및 데이터 관리
#### 데이터 블록 할당 체계
#### 가용 아이노드 및 블록 비트맵

### 신뢰성 및 데이터 무결성
#### 저널링 (Journaling) 메커니즘
#### 일관성 검사 및 복구 (fsck)

### 파일 제어 및 인터페이스
#### 파일 디스크립터 테이블
##### 프로세스별 열린 파일 제한
##### 시스템 전체 파일 핸들 관리
#### 확장 속성 (xattr) 및 ACL
#### 파일 변경 이벤트 통지 (inotify)

## 프로세스 제어

### 프로세스 식별 및 구조
#### 프로세스 ID (PID) 및 계층 구조
#### 프로세스 제어 블록 (task_struct)
#### 자격 증명 (UID/GID) 및 실효 권한

### 프로세스 생성 및 전환
#### 프로세스 복제 (fork/vfork)
#### 실행 이미지 교체 (execve)
#### 경량 프로세스 생성 (clone)
#### 쓰기 시 복사 (Copy-on-Write) 최적화

### 프로세스 상태 및 생명 주기
#### 커널 프로세스 상태 모델
##### 실행 및 대기 상태 (TASK_RUNNING)
##### 수면 상태 (Interruptible vs Uninterruptible)
#### 프로세스 종료 및 자원 회수
##### 종료 상태 전달 (Exit Status)
##### 좀비 프로세스(Zombie)와 자원 고갈
##### 고아 프로세스(Orphan)와 재입양 (Reparenting)

### 프로세스 스케줄링 및 우선순위
#### 스케줄링 클래스 (Scheduling Classes)
##### 완전 공정 스케줄러 (CFS)
##### 실시간 스케줄링 (Real-time)
#### 스케줄링 우선순위 제어
##### Nice 값과 가중치
##### 정적 및 동적 우선순위
#### 프로세서 친화도 (CPU Affinity)

### 시그널(Signal) 메커니즘
#### 표준 및 실시간 시그널 분류
#### 시그널 핸들러 및 기본 동작 (Action)
#### 시그널 마스킹 및 블로킹 (sigprocmask)
#### 비정상 종료와 코어 덤프 (Core Dump)

### 스레드 모델링
#### 경량 프로세스 (LWP)와 스레드
#### NPTL (Native POSIX Thread Library)
#### 스레드 그룹 및 TGID 식별

### 프로세스 간 통신 (IPC)
#### 시스템 V 및 POSIX IPC 비교
#### 공유 메모리 및 세마포어
#### 유닉스 도메인 소켓 (UDS)

## 메모리 관리

### 가상 메모리 및 주소 공간
#### 프로세스 주소 공간 레이아웃
##### 사용자 공간 가상 주소 범위
##### 커널 주소 공간 매핑
#### 페이지 테이블 및 하드웨어 연동
##### 다단계 페이지 테이블 구조
##### 주소 변환 캐시 (TLB)

### 물리적 메모리 관리
#### 버디 시스템 (Buddy System)
#### 슬랩 할당자 (Slab/Slub/Slob)

### 유저 공간 메모리 할당
#### 힙 영역 확장 (brk/sbrk)
#### 메모리 매핑 인터페이스 (mmap)
##### 익명 매핑 (Anonymous Mapping)
##### 파일 기반 매핑 (File-backed Mapping)

### 페이지 폴트 메커니즘
#### 요구 페이징 (Demand Paging)
#### 메이저 및 마이너 페이지 폴트
#### 복사 시점 쓰기 (CoW) 폴트

### 페이지 캐시 및 버퍼
#### 파일 시스템 성능 최적화 (Caching)
#### 더티 페이지 및 지연 쓰기 (Writeback)
#### 익명 메모리 vs 파일 백업 메모리

### 메모리 회수 및 압박 처리
#### 스왑(Swap) 매커니즘
##### 스왑 인/아웃(Swap-in/out) 원리
##### 스왑 성향 제어 (swappiness)
#### 페이지 회수 알고리즘 (kswapd)
#### OOM (Out Of Memory) Killer
##### OOM 점수 및 가중치 조정
##### 희생자 프로세스 선정 로직

### 메모리 성능 및 효율화
#### 메모리 오버커밋 (Overcommit) 정책
#### 휴즈 페이지 (Huge Pages)
##### 표준 휴즈 페이지 (Static)
##### 투명한 휴즈 페이지 (Transparent)
#### NUMA 메모리 정책 (Affinity)

## 네트워크 스택

### 리눅스 네트워킹 아키텍처
#### 커널 내 프로토콜 스택 흐름
#### 소켓 추상화 계층 (BSD Sockets)
#### 네트워크 장치와 드라이버 인터페이스

### 소켓 및 전송 계층 (L4)
#### TCP 상태 모델 및 연결 제어
##### 3-Way/4-Way 핸드쉐이크 내부 동작
##### TIME_WAIT 및 리소스 재사용 (Keepalive)
#### UDP 데이터그램 처리 매커니즘

### IP 및 라우팅 관리
#### 커널 라우팅 테이블 및 포워딩
#### ARP 및 Neighbor 캐시 매커니즘
#### IP 데이터그램 분할 및 재조합 (Fragmentation)

### 패킷 필터링 및 보안 (Netfilter)
#### 넷필터(Netfilter) 프레임워크 후크
#### 연결 추적 (Connection Tracking)
#### iptables / nftables 아키텍처

### 네트워크 성능 최적화
#### 수신 패킷 처리 모델 (NAPI)
#### 인터럽트 병목과 RSS/RPS
#### 커널 소켓 버퍼 (SNDBUF/RCVBUF)
#### 하드웨어 오프로딩 (Checksum, TSO/LRO)

### 가상 네트워킹
#### 가상 이더넷 장치 (veth)
#### 소프트웨어 브리지 (Linux Bridge)
#### TUN/TAP 및 가상 터널링

### 네트워크 가시성 및 진단 인터페이스
#### 소켓 통계 인터페이스 (ss/netstat)
#### 패킷 캡처 및 필터링 (libpcap/tcpdump)
#### 네트워크 정보 커널 파일 시스템
##### /proc/net (커널 통계 및 상태)
##### /sys/class/net (장치 속성)
#### 트래픽 제어 및 큐잉 (tc/qdisc)

## 입출력 및 스토리지

### I/O 스택 아키텍처
#### 유저 공간 I/O 인터페이스
##### 표준 입출력 (Buffered I/O)
##### 직접 입출력 (Direct I/O)
#### 고성능 비동기 I/O 매커니즘
##### 리눅스 AIO (Async I/O)
##### io_uring 인터페이스
#### 제로 카피 (Zero-copy) 기술
##### sendfile 시스템 콜
##### splice 및 vmsplice

### 블록 장치 레이어
#### 바이오(bio) 구조체 및 요청 큐
#### I/O 스케줄링 및 병합
##### 멀티 큐 스케줄링 (blk-mq)
##### 스케줄러 알고리즘 (None, Deadline, BFQ)

### 저장 장치 물리적 관리
#### 디스크 파티셔닝 체계
##### MBR 및 GPT 레이아웃
#### 플래시 스토리지 최적화
##### TRIM 및 가비지 컬렉션 연동
##### NVMe 드라이버 인터페이스

### 스토리지 추상화 및 가상화
#### 장치 매퍼(Device Mapper) 프레임워크
#### 논리 볼륨 관리 (LVM)
##### PV, VG, LV 계층 구조
##### 스냅샷 및 동적 온라인 확장
#### 소프트웨어 RAID (mdadm)

### 마운트 옵션 및 할당량
#### 입출력 동기화 정책 (Sync/Async)
#### 메타데이터 갱신 최적화 (noatime/relatime)
#### 디스크 할당량 제어 (Quota)

### 가상 블록 장치 및 상태 가시성
#### 루프백 장치 (Loopback Device)
#### 블록 장치 상태 인터페이스 (/sys/block)
#### I/O 지연 시간 및 통계 지표

## 사용자 및 보안

### 사용자 및 그룹 모델
#### 사용자 식별자 (UID)
##### 루트(root) 계정 및 시스템 사용자
##### 유저 공간 식별자 매핑
#### 그룹 식별자 (GID)
#### 계정 정보 데이터베이스 (/etc/passwd, /etc/group)
#### 비밀번호 해시 및 섀도우 속성 (/etc/shadow)

### 표준 파일 시스템 권한
#### 소유권 및 권한 모드 (rwx)
#### 특수 권한 비트
##### SetUID (SUID)
##### SetGID (SGID)
##### 스티키 비트 (Sticky Bit)
#### 기본 생성 권한 마스크 (umask)

### 고급 권한 및 기능 제어
#### 액세스 제어 리스트 (ACL)
##### 사용자/그룹별 세분화된 접근 제어
##### 상속을 위한 기본(Default) ACL
#### 프로세스 기능 (Capabilities)
##### 루트 권한의 세분화 및 격리
##### 기능 상속 및 바운딩 세트 (Bounding Set)

### 권한 상승 및 인증 프레임워크
#### sudo (SuperUser Do) 시스템
##### sudoers 정책 및 환경 변수 제한
#### PAM (Pluggable Authentication Modules)
##### 인증 시퀀스 및 모듈 아키텍처

### 강제적 접근 제어 (MAC)
#### LSM (Linux Security Modules) 아키텍처
#### SELinux (Security-Enhanced Linux)
##### 컨텍스트(Context) 및 레이블링
##### 보안 정책 및 모드 (Enforcing/Permissive)
#### AppArmor (Application Armor)
##### 프로파일(Profile) 기반 경로 제어

### 시스템 감사 및 로그 관리
#### 감사(Audit) 프레임워크 (auditd)
##### 시스템 콜 추적 및 규칙 설정
#### 보안 로그 분석 (/var/log/secure, auth.log)

### 네트워크 보안 및 접속 강화
#### SSH 보안 강화
##### 키 기반 인증 및 에이전트 포워딩
##### 접속 제한 및 설정 최적화 (sshd_config)
#### 서비스 노출 제한 (TCP Wrappers)

## 시스템 서비스

### 초기화 시스템 (Init System)
#### 부팅 시퀀스 및 PID 1의 역할
#### Systemd 아키텍처 및 설계 철학
#### 유닛(Unit) 추상화 및 종속성 모델

### Systemd 유닛 구성
#### 서비스 유닛 (.service)
##### 실행 유형 (Type=simple, forking, notify)
##### 자동 재시작 및 생존 정책 (Restart)
##### 실행 환경 제어 (Environment, User, Group)
#### 타겟 유닛 (.target)
##### 부트 타겟 및 런레벨(Runlevel) 매핑

### 서비스 관리 및 상태 추적
#### 서비스 생명 주기 제어 (systemctl)
##### 시작, 중지, 재시작 및 로드
##### 활성화(Enable) 및 비활성화(Disable)의 메커니즘
#### 유닛 상태 및 의존성 분석
##### 서비스 런타임 상태 진단
##### 종속성 그래프 분석 (list-dependencies)

### 시스템 로깅 및 저널 관리
#### 통합 로깅 아키텍처 (Journald)
#### 저널 데이터 쿼리 및 필터링 (journalctl)
##### 부팅 세션별 로그 추적
##### 실시간 로그 모니터링
#### 로그 보존 정책 및 회전(Rotation) 관리

### 작업 예약 및 자동화
#### 시간 기반 작업 (Cron)
##### Crontab 구문 및 구성
##### 전역(/etc/cron.d) vs 사용자별 설정
#### 일회성 작업 예약 (at)
#### Systemd 타이머 (.timer)
##### 유닛 기반 실행 및 종속성 관리
##### 정확도 및 지연 실행(Accuracy) 제어

## 리소스 격리

### 커널 네임스페이스 (Namespaces)
#### 프로세스 트리 격리 (PID Namespace)
#### 네트워크 스택 격리 (Network Namespace)
##### 가상 인터페이스 및 루프백 독립화
#### 파일 시스템 마운트 격리 (Mount Namespace)
##### 공유 서브트리 및 전파 매커니즘
#### 호스트 및 도메인 이름 격리 (UTS Namespace)
#### 프로세스 간 통신 격리 (IPC Namespace)
#### 사용자 및 권한 격리 (User Namespace)

### 제어 그룹 (Cgroups)
#### Cgroup V1 vs V2 아키텍처
#### CPU 리소스 제어
##### CFS 할당량 및 주기 (Quota/Period)
##### CPU 공유 비중 (Shares/Weight)
#### 메모리 리소스 제어
##### 메모리 한계 설정 및 회수 정책
##### OOM Killer 알림 및 제어
#### 블록 입출력(I/O) 제어
##### 대역폭 제한 및 가중치

### 파일 시스템 루트 격리
#### chroot 환경 및 한계
#### pivot_root 기반 루트 전환

### 시스템 콜 보안 및 필터링
#### Seccomp (Secure Computing Mode)
##### BPF 기반 필터링 규칙

### 계층형 파일 시스템
#### Union Mount 및 OverlayFS

## 성능 모니터링

### 시스템 부하 및 개요
#### 평균 부하 (Load Average) 해석
#### 가동 시간 및 시스템 요약 (uptime)

### CPU 성능 분석
#### 유저 및 커널 시간 점유율
#### 입출력 대기 (I/O Wait) 및 스틸(Steal) 타임
#### 프로세스별 자원 점유 (top, htop)
#### 컨텍스트 스위칭 및 인터럽트 빈도

### 메모리 성능 분석
#### 가용 메모리 및 캐시/버퍼 활용량
#### 메모리 활동 및 페이징 통계 (vmstat)
#### 커널 슬랩 할당자 가시성 (slabtop)
#### 스왑 및 메모리 회수 활동 모니터링

### 저장 장치 I/O 성능 분석
#### 디스크 사용률 및 데이터 전송률
#### I/O 지연 시간(Latency) 및 큐 깊이
#### 프로세스별 I/O 소모량 추적 (iotop)

### 커널 및 프로세스 가시성 (Observability)
#### 시스템 호출 흐름 추적 (strace)
#### 프로세스 자원 점유 현황 (lsof)
#### 커널 성능 프로파일링 (perf)

### 동적 추적 (eBPF)
#### eBPF 실행 모델 및 안전성
#### BCC 및 bpftrace 도구 활용

### 시스템 활동 이력 분석
#### 성능 데이터 수집 및 기록 (sar/sysstat)

## 패키지 및 인프라

### 소프트웨어 패키지 관리
#### 패키지 관리자 아키텍처 (Apt, Yum/Dnf)
#### 패키지 데이터베이스 및 의존성 해결
#### 로컬 패키지 제어 (Dpkg, Rpm)
#### 레포지토리 및 미러 사이트 관리

### 아카이브 및 압축
#### 파일 아카이빙 (tar)
#### 데이터 압축 알고리즘 (Gzip, Bzip2, Xz)

### 컴파일 및 빌드 환경
#### 컴파일러 툴체인 (GCC, LLVM)
#### 빌드 자동화 도구 (Make, CMake)

### 공유 라이브러리 관리
#### 동적 링커와 로더 (ld.so)
#### 공유 라이브러리 탐색 경로 (LD_LIBRARY_PATH)
#### 라이브러리 의존성 분석 (ldd)
#### 라이브러리 캐시 갱신 (ldconfig)

### 시스템 시간 및 동기화
#### 시스템 시계와 하드웨어 시계
#### NTP 및 Chrony 서비스
#### 타임존(Timezone) 관리 및 로컬라이징

### 원격 인프라 및 데이터 전송
#### 원격 쉘 접속 및 제어 (SSH)
#### 효율적인 증분 데이터 전송 (Rsync)
#### 네트워크 기반 파일 복제 (SCP)