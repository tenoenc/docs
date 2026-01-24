## 컨테이너 기술의 기초와 가상화

### 가상화 모델의 이해
#### 하드웨어 수준 가상화
#### OS 수준 가상화
#### 하이퍼바이저 (Hypervisor)
#### 게스트 운영체제 (Guest OS)

### 리눅스 커널 격리 (Namespace)
#### mnt 네임스페이스 (파일시스템 격리)
#### pid 네임스페이스 (프로세스 격리)
#### net 네임스페이스 (네트워크 격리)
#### ipc 네임스페이스 (통신 자원 격리)
#### uts 네임스페이스 (호스트 네임 격리)
#### user 네임스페이스 (사용자 권한 격리)

### 리눅스 자원 제어 (Cgroups)
#### CPU 자원 할당 제어
#### 메모리 자원 제한
#### I/O 대역폭 제한
#### 프로세스 개수 제한 (PIDs)

### 파일시스템 추상화
#### 루트 디렉토리 격리 (chroot)
#### 피벗 루트 (pivot_root)
#### 유니온 파일 시스템 (UnionFS)
#### OverlayFS 구조

## Docker 아키텍처 및 런타임

### 도커 엔진 구조
#### 도커 데몬 (dockerd)
#### 도커 클라이언트 (CLI)
#### 도커 API (REST API)

### 컨테이너 런타임 계층
#### containerd (컨테이너 관리)
#### runC (OCI 런타임)
#### containerd-shim
#### gVisor (샌드박스 런타임)
#### Kata Containers (VM 기반 런타임)

### 실행 표준 및 통신
#### OCI (Open Container Initiative) 규격
#### 도커 소켓 (docker.sock)
#### CRI (Container Runtime Interface)

## 이미지 빌드 및 레이어 관리

### 이미지 추상화
#### 도커 이미지 (Image)
#### 이미지 레이어 (ReadOnly Layer)
#### 컨테이너 레이어 (Writable Layer)
#### 이미지 매니페스트 (Manifest)

### 저장 및 효율화 매커니즘
#### Copy-on-Write (CoW)
#### 콘텐츠 주소 지정 (CAS)
#### 가비지 컬렉션 (Image GC)
#### 레이어 캐싱 (Layer Caching)

### 이미지 배포 아키텍처
#### 도커 레지스트리 (Registry)
#### 도커 허브 (Docker Hub)
#### 프라이빗 레지스트리

## 컨테이너 실행 및 제어

### 컨테이너 생명 주기 (Lifecycle)
#### 컨테이너 상태 (Status)
#### 생성 (create)
#### 실행 (start/run)
#### 일시 정지 (pause)
#### 종료 (stop)
#### 재시작 (restart)
#### 강제 종료 (kill)
#### 삭제 (rm)

### 프로세스 관리 및 모니터링
#### 전경 프로세스 (Foreground)
#### 배경 프로세스 (Detached)
#### 인터랙티브 쉘 (Interactive)
#### 표준 스트림 (stdout/stderr)
#### 컨테이너 로그 (logs)
#### 리소스 통계 (stats)

### 실행 환경 제어
#### 엔트리포인트 (ENTRYPOINT)
#### 실행 명령 (CMD)
#### 환경 변수 (ENV)
#### 작업 디렉토리 (WORKDIR)
#### 상태 검사 (Health Check)
#### 종료 시그널 (SIGTERM/SIGKILL)

## 네트워크 아키텍처

### 네트워크 드라이버
#### 브리지 네트워크 (Bridge)
#### 호스트 네트워크 (Host)
#### 오버레이 네트워크 (Overlay)
#### 가상 네트워크 (None)
#### Macvlan 드라이버

### 네트워크 통신 매커니즘
#### 포트 바인딩 (Port Mapping)
#### 가상 이더넷 (veth pair)
#### 내부 DNS (Service Discovery)
#### 네트워크 별칭 (Alias)
#### IP 주소 관리 (IPAM)
#### 도커 링크 (Legacy Links)

### 트래픽 제어
#### 포트 포워딩 (DNAT)
#### 네트워크 격리 (Isolation)
#### 사용자 정의 네트워크

## 데이터 영속성 및 저장소 전략

### 스토리지 마운트 모델
#### 볼륨 (Volumes)
#### 바인드 마운트 (Bind Mounts)
#### tmpfs 마운트
#### 명명된 볼륨 (Named Volumes)
#### 익명 볼륨 (Anonymous Volumes)

### 데이터 관리 매커니즘
#### 볼륨 드라이버 (Volume Driver)
#### 데이터 공유 (Volumes From)
#### 백업 및 복구 (Export/Import)
#### 고정 파일 시스템 (Read-only Mount)

## 멀티 컨테이너 오케스트레이션 (Docker Compose)

### Compose 아키텍처
#### Compose 규격 (YAML Spec)
#### 서비스 정의 (Services)
#### 프로젝트 (Project)
#### 스택 (Stack)

### 서비스 구성 요소
#### 이미지 지정 (image/build)
#### 컨테이너 이름 (container_name)
#### 재시작 정책 (restart_policy)
#### 의존성 제어 (depends_on)

### 실행 및 관리
#### 프로젝트 생명 주기 (up/down)
#### 병렬 실행 제어
#### 환경 변수 확장 (.env)
#### 컨테이너 스케일링 (scale)
#### 컴포즈 프로파일 (Profiles)

## 보안 및 시스템 보호 (Protection & Security)

### 권한 및 격리 제어
#### 루트리스 모드 (Rootless Mode)
#### 리눅스 기능 제어 (Capabilities)
#### 특권 모드 (Privileged Mode)
#### 사용자 권한 격리 (User Namespace)

### 커널 보안 인터페이스 연동
#### Seccomp 프로파일 (Syscall Filtering)
#### AppArmor 프로파일
#### SELinux 보안 레이블

### 이미지 및 데이터 보안
#### 이미지 서명 (Docker Content Trust)
#### 보안 취약점 스캐닝 (Scanning)
#### 도커 시크릿 (Docker Secrets)
#### 읽기 전용 컨테이너 (Read-only Rootfs)

## 성능 최적화 및 유지보수

### 빌드 시스템 고도화
#### BuildKit 엔진
#### 멀티 스테이지 빌드 (Multi-stage)
#### 빌드 컨텍스트 (Build Context)
#### .dockerignore 최적화

### 시스템 자원 관리
#### 가용 자원 정리 (Pruning)
#### 도커 컨텍스트 (Contexts)
#### 로그 드라이버 설정 (Log Rotation)
#### 이벤트 스트림 (Events)