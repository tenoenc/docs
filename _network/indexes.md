## 네트워크 아키텍처

### 계층 참조 모델 (Reference Models)
#### OSI 7계층 모델
##### 물리 계층 (Physical)
##### 데이터 링크 계층 (Data Link)
##### 네트워크 계층 (Network)
##### 전송 계층 (Transport)
##### 세션 계층 (Session)
##### 표현 계층 (Presentation)
##### 응용 계층 (Application)
#### TCP/IP 4계층 모델
##### 네트워크 액세스 계층
##### 인터넷 계층
##### 전송 계층
##### 응용 계층

### 데이터 전송 매커니즘 (Transmission)
#### 통신 단위 (PDU)
##### 메시지 (Message)
##### 세그먼트 (Segment)
##### 패킷 (Packet)
##### 프레임 (Frame)
##### 비트 (Bit)
#### 데이터 제어 원리
##### 캡슐화 (Encapsulation)
##### 역캡슐화 (Decapsulation)
##### 페이로드 (Payload)
##### 오버헤드 (Overhead)

### 주소 지정 체계 (Addressing)
#### 계층별 식별자
##### MAC 주소
##### IP 주소
##### 포트 번호 (Port)
#### 주소 할당 및 확인
##### 정적 할당
##### 동적 할당 (DHCP)
##### 루프백 주소 (Loopback)
##### ARP (주소 확인 프로토콜)

### 통신 패러다임 (Paradigm)
#### 전송 방향 모드
##### 단방향 (Simplex)
##### 반이중 (Half Duplex)
##### 전이중 (Full Duplex)
#### 전송 대상 분류 (Cast)
##### 유니캐스트 (Unicast)
##### 브로드캐스트 (Broadcast)
##### 멀티캐스트 (Multicast)
##### 애니캐스트 (Anycast)

### 네트워크 구조 및 연결 (Topology)
#### 물리적/논리적 구조
##### 성형 (Star)
##### 망형 (Mesh)
##### 트리형 (Tree)
##### 버스형 (Bus)
#### 네트워크 범위 분류
##### PAN (Personal)
##### LAN (Local)
##### WAN (Wide)

## 애플리케이션 계층 (Application Layer)

### 웹 통신 프로토콜 (HTTP)
#### HTTP 메시지 구조
##### 시작 줄 (Start Line)
##### 헤더 (Header)
##### 본문 (Body)
#### HTTP 버전 아키텍처
##### HTTP/1.1 (Keep-Alive, Pipelining)
##### HTTP/2 (Multiplexing, Header Compression)
##### HTTP/3 (QUIC 기반)
#### HTTP 상태 관리 및 보안
##### 쿠키 (Cookie)
##### 세션 (Session)
##### HTTPS (SSL/TLS 터미네이션)

### 도메인 네임 시스템 (DNS)
#### DNS 구성 요소
##### 리졸버 (Resolver)
##### 루트 네임 서버 (Root)
##### TLD 네임 서버 (TLD)
##### 책임 네임 서버 (Authoritative)
#### DNS 레코드 및 질의
##### A / AAAA 레코드
##### CNAME 레코드
##### 재귀적 질의 (Recursive Query)
##### 반복적 질의 (Iterative Query)

### 데이터 표현 및 직렬화 (Serialization)
#### 데이터 교환 포맷
##### JSON
##### XML
##### Protocol Buffers (Protobuf)
#### 인터페이스 아키텍처
##### REST API
##### gRPC
##### GraphQL
##### 웹훅 (Webhook)

### 실시간 및 양방향 통신
#### 웹 실시간 기술
##### 웹소켓 (WebSocket)
##### SSE (Server-Sent Events)
##### WebRTC
#### 메시지 전송 프로토콜
##### MQTT
##### AMQP

### 기타 응용 서비스
#### 파일 및 메일 전송
##### FTP / SFTP
##### SMTP
##### POP3 / IMAP
#### 원격 접속 제어
##### SSH (Secure Shell)
##### Telnet

## 전송 제어 (Transport)

### 전송 계층 프로토콜 기초
#### TCP (Transmission Control Protocol)
##### 연결 지향형 (Connection-oriented)
##### 전이중 및 양방향 서비스
##### 바이트 스트림 (Byte Stream)
#### UDP (User Datagram Protocol)
##### 비연결형 (Connectionless)
##### 최소한의 오버헤드
##### 데이터그램 방식

### TCP 연결 및 상태 관리
#### 연결 설정 및 해제
##### 3-Way Handshake (연결 설정)
##### 4-Way Handshake (연결 해제)
#### TCP 제어 비트 (Flags)
##### SYN (Synchronize)
##### ACK (Acknowledgment)
##### FIN (Finish)
##### RST (Reset)
#### TCP 상태 변이 (State)
##### LISTEN / SYN_SENT / SYN_RECV
##### ESTABLISHED
##### FIN_WAIT / CLOSE_WAIT
##### TIME_WAIT (포트 고갈 문제)

### 데이터 전송 단위 및 포트
#### 세그먼트 (Segment)
##### TCP 헤더 구조
##### MSS (Maximum Segment Size)
#### 포트 관리 (Port)
##### 포트 다중화 (Multiplexing)
##### 잘 알려진 포트 (Well-known Ports)
##### 동적 포트 (Ephemeral Ports)

### TCP 신뢰성 및 흐름 제어 (Reliability & Flow Control)
#### 재전송 매커니즘 (Retransmission)
##### 재전송 타이머 (Retransmission Timer)
##### RTO (Retransmission Time-Out)
##### 빠른 재전송 (Fast Retransmission)
#### 흐름 제어 (Flow Control)
##### 정지-대기 (Stop-and-Wait)
##### 슬라이딩 윈도우 (Sliding Window)
##### 수신 윈도우 (RWND)
##### 윈도우 크기 조절 (Window Scaling)
#### 오류 제어 (Error Control)
##### 체크섬 (Checksum)
##### 누적 응답 (Cumulative ACK)
##### 선택적 응답 (SACK)

### TCP 혼잡 제어 (Congestion Control)
#### 혼잡 제어 알고리즘 단계
##### 느린 시작 (Slow Start)
##### 혼잡 회피 (Congestion Avoidance)
##### 빠른 회복 (Fast Recovery)
#### 혼잡 제어 핵심 요소
##### 혼잡 윈도우 (CWND)
##### 임계치 (ssthresh)
##### 타임아웃 처리
#### 주요 혼잡 제어 정책
##### TCP Tahoe / Reno
##### TCP BBR (Google)

### UDP 응용 및 변형
#### UDP 서비스 모델
##### 최선형 전달 (Best-effort)
##### 멀티캐스트 전송
#### 전송 계층의 현대적 확장
##### QUIC (Quick UDP Internet Connections)
##### SCTP (Stream Control Transmission Protocol)
##### 가상 회선 프로토콜

## 인터넷 프로토콜 (Network)

### IP 주소 체계 (IP Addressing)
#### IPv4 구조
##### 32비트 주소 체계
##### 클래스 기반 할당 (Classful)
##### 서브넷 마스크 (Subnet Mask)
##### CIDR (Classless Inter-Domain Routing)
##### 서브네팅 (Subnetting)
#### IPv6 구조
##### 128비트 주소 체계
##### IPv6 주소 표기법
##### 유니캐스트/멀티캐스트/애니캐스트
#### 특수 IP 주소
##### 공인 IP (Public IP)
##### 사설 IP (Private IP)
##### 루프백 주소 (127.0.0.1)
##### 0.0.0.0 (Wildcard Address)

### 주소 변환 및 제어 (Translation & Control)
#### NAT (Network Address Translation)
##### Static NAT (1:1)
##### Dynamic NAT (N:N)
##### PAT (Port Address Translation)
##### 포트 포워딩 (Port Forwarding)
#### ICMP (Internet Control Message Protocol)
##### 에러 메시지 (Destination Unreachable)
##### 질의 메시지 (Echo Request/Reply)
##### Ping 동작 원리
##### Traceroute 동작 원리

### IP 데이터 전송 특성
#### 패킷 구조 및 처리
##### IP 헤더 구조
##### TTL (Time To Live)
##### 단편화 (Fragmentation)
##### MTU (Maximum Transmission Unit)

### 라우팅 및 경로 선택 (Routing)
#### 라우팅 기본 원칙
##### 라우팅 테이블 (Routing Table)
##### 디폴트 게이트웨이 (Default Gateway)
##### 넥스트 홉 (Next Hop)
##### 메트릭 및 우선순위 (Administrative Distance)
#### 라우팅 알고리즘 분류
##### 정적 라우팅 (Static Routing)
##### 동적 라우팅 (Dynamic Routing)
##### 거리 벡터 알고리즘 (Distance Vector)
##### 링크 상태 알고리즘 (Link State)
#### 주요 라우팅 프로토콜
##### RIP (Routing Information Protocol)
##### OSPF (Open Shortest Path First)
##### BGP (Border Gateway Protocol)
##### AS (Autonomous System)

### 네트워크 논리적 구획 및 확장 (Segmentation)
#### 가상 로컬 네트워크
##### VLAN (Virtual LAN)
##### 트렁킹 (Trunking / 802.1Q)
##### VTP (VLAN Trunking Protocol)
#### 사설망 확장 및 터널링
##### VPN (Virtual Private Network)
##### IPsec (IP Security)
##### SSL VPN
##### MPLS (Multi-Protocol Label Switching)

### 계층별 논리 장비 (Logical Layer Devices)
#### L3 전송 장비
##### 라우터 (Router)
##### L3 스위치 (Layer 3 Switch)
##### 유니캐스트 루프 방지

## 네트워크 인터페이스

### 데이터 링크 계층 (Data Link)
#### 이더넷 (Ethernet)
##### 이더넷 프레임 구조
##### MTU (Maximum Transmission Unit)
##### CSMA/CD (충돌 감지)
#### 매체 접근 제어 (MAC)
##### MAC 주소 체계
##### 유니캐스트/멀티캐스트/브로드캐스트 주소
##### NIC (네트워크 인터페이스 카드)

### 주소 해석 및 확인 (Address Resolution)
#### ARP (Address Resolution Protocol)
##### ARP 요청 및 응답
##### ARP 캐시 테이블
##### Gratuitous ARP (GARP)
#### 관련 프로토콜
##### RARP (Reverse ARP)
##### NDP (IPv6 인접 노드 탐색)

### L2 스위칭 기술 (Switching)
#### 스위치 동작 원리
##### 주소 학습 (Learning)
##### 플러딩 (Flooding)
##### 포워딩 및 필터링
##### 에이징 (Aging)
#### 가상 로컬망 제어
##### VLAN 태깅 (802.1Q)
##### 액세스 포트 vs 트렁크 포트
#### 루프 방지 및 이중화
##### 루프 발생 원리 (Broadcast Storm)
##### STP (Spanning Tree Protocol)
##### Bridge ID 및 Path Cost

### 물리 계층 (Physical)
#### 신호 및 전송 기초
##### 비트 (Bit)
##### 대역폭 (Bandwidth)
##### 전송 지연 (Propagation Delay)
#### 네트워크 하드웨어
##### 허브 (Hub)
##### 리피터 (Repeater)
##### 전송 매체 (UTP, 광섬유)

## 웹 시스템 인프라 (Web System Infrastructure)

### 로드 밸런싱 (Load Balancing)
#### 로드 밸런서 분류
##### L4 로드 밸런서 (Transport Layer)
##### L7 로드 밸런서 (Application Layer)
#### 부하 분산 알고리즘
##### 라운드 로빈 (Round Robin)
##### 최소 연결 (Least Connection)
##### IP 해시 (Source IP Hash)
#### 서버 가용성 관리
##### 상태 확인 (Health Check)
##### 세션 정착 (Sticky Session)
##### 서버 드레인 (Server Drain)

### 프록시 서버 (Proxy Server)
#### 프록시 배치 유형
##### 포워드 프록시 (Forward Proxy)
##### 리버스 프록시 (Reverse Proxy)
#### 주요 수행 기능
##### SSL 오프로딩 (SSL Offloading)
##### 암호화 가속
##### 데이터 압축
##### 요청 버퍼링

### 콘텐츠 전송 및 가속 (Content Delivery)
#### CDN (Content Delivery Network)
##### 에지 서버 (Edge Server)
##### 오리진 서버 (Origin Server)
#### 캐싱 매커니즘
##### 캐시 적중 (Cache Hit)
##### 캐시 미스 (Cache Miss)
##### 캐시 무효화 (Invalidation)
##### TTL (Time To Live) 관리

### 네트워크 경계 아키텍처
#### 네트워크 격리
##### DMZ (De-Militarized Zone)
##### 배스천 호스트 (Bastion Host)
#### 서비스 진입점
##### API 게이트웨이 (API Gateway)
##### 인그레스 컨트롤러 (Ingress Controller)
##### 서비스 메시 (Service Mesh) 기초

## 네트워크 보안 (Network Security)

### 인증 및 접근 제어 (Authentication & Access Control)
#### 식별 및 인증
##### 다중 요소 인증 (MFA)
##### SSO (Single Sign-On)
##### OAuth 2.0 / OIDC
#### 접근 권한 관리
##### 접근 제어 리스트 (ACL)
##### 역할 기반 접근 제어 (RBAC)
##### 제로 트러스트 아키텍처 (Zero Trust)

### 암호화 및 전송 보안 (Encryption & Transport Security)
#### TLS/SSL 매커니즘
##### TLS 핸드쉐이크 (Handshake)
##### 공개키 증명서 (Certificate)
##### 인증 기관 (CA)
##### 암호화 제품군 (Cipher Suite)
#### 암호화 기초 원리
##### 대칭키 암호화
##### 비대칭키 암호화 (공개키)
##### 해시 함수 (Hash Function)
##### 디지털 서명 (Digital Signature)

### 네트워크 위협 및 공격 (Threats & Attacks)
#### 서비스 거부 공격
##### DoS (Denial of Service)
##### DDoS (Distributed DoS)
##### DRDoS (Reflection DoS)
#### 침입 및 데이터 탈취
##### 패킷 스니핑 (Sniffing)
##### IP/MAC 스푸핑 (Spoofing)
##### 세션 하이재킹 (Session Hijacking)
##### 중간자 공격 (MITM)
#### 애플리케이션 보안 위협
##### SQL 인젝션
##### 사이트 간 스크립팅 (XSS)
##### CSRF (사이트 간 요청 위조)

### 보안 방어 체계 (Defense Systems)
#### 경계 보안 장치
##### 방화벽 (Firewall)
##### 웹 방화벽 (WAF)
##### 포워드/리버스 프록시 보안
#### 탐지 및 대응 시스템
##### IDS (침입 탐지 시스템)
##### IPS (침입 방지 시스템)
##### SIEM (보안 정보 및 이벤트 관리)
#### 가상 사설 통신
##### VPN (Virtual Private Network)
##### IPsec 프로토콜

## 성능 및 최적화 (Performance & Optimization)

### 핵심 성능 지표 (Performance Metrics)
#### 지연 시간 (Latency)
##### 전파 지연 (Propagation Delay)
##### 전송 지연 (Transmission Delay)
##### 처리 지연 (Processing Delay)
##### 큐잉 지연 (Queuing Delay)
#### 대역폭 및 처리량
##### 대역폭 (Bandwidth)
##### 처리량 (Throughput)
##### 유효 처리량 (Goodput)
#### 가용성 및 품질
##### 패킷 손실 (Packet Loss)
##### 지터 (Jitter)
##### 왕복 시간 (RTT)

### 전송 계층 최적화 (Transport Tuning)
#### TCP 가속 기법
##### TCP Fast Open (TFO)
##### 윈도우 스케일링 (Window Scaling)
##### 선택적 응답 (SACK)
#### 데이터 전송 효율화
##### Nagle 알고리즘 (Nagle's Algorithm)
##### 지연된 응답 (Delayed ACK)
##### 제로 카피 (Zero-Copy)

### 웹 및 콘텐츠 최적화 (Web Optimization)
#### 데이터 압축 및 인코딩
##### Gzip / Brotli 압축
##### 이미지 포맷 최적화 (WebP / AVIF)
##### 미니피케이션 (Minification)
#### 리소스 로딩 전략
##### 커넥션 풀링 (Connection Pooling)
##### 프리페칭 (Prefetching)
##### 프리로딩 (Preloading)
##### 도메인 샤딩 (Domain Sharding)

### 네트워크 캐싱 전략 (Caching)
#### 브라우저 캐싱
##### Cache-Control 헤더
##### ETag / Last-Modified
#### 인프라 캐싱
##### CDN 캐싱 전략
##### 무효화 (Invalidation) 및 갱신

## 가상화 네트워크 (Virtualized Networking)

### 클라우드 가상 네트워크 (Cloud Networking)
#### VPC (Virtual Private Cloud) 아키텍처
##### 서브넷 (Subnet) 격리
##### 인터넷 게이트웨이 (IGW)
##### NAT 게이트웨이
##### 가상 사설 게이트웨이 (VGW)
#### 클라우드 연결 인프라
##### VPC 피어링 (Peering)
##### 트랜짓 게이트웨이 (Transit Gateway)
##### 전용선 연결 (Direct Connect)
##### VPC 엔드포인트 (PrivateLink)

### 컨테이너 네트워크 (Container Networking)
#### 도커 네트워크 모델 (Docker)
##### 브리지 모드 (Bridge)
##### 호스트 모드 (Host)
##### 오버레이 모드 (Overlay)
##### None 모드
#### 쿠버네티스 네트워크 (Kubernetes)
##### Pod 간 통신 (Pod-to-Pod)
##### 서비스 (Service) 추상화
##### 인그레스 (Ingress) 컨트롤러
##### CNI (Container Network Interface)

### 소프트웨어 정의 네트워크 (SDN)
#### SDN 계층 구조
##### 컨트롤 플레인 (Control Plane)
##### 데이터 플레인 (Data Plane)
##### 오픈플로우 (OpenFlow) 프로토콜
#### 네트워크 기능 가상화 (NFV)
##### 가상 라우터 (vRouter)
##### 가상 방화벽 (vFW)
##### 로드 밸런서 가상화 (vLB)

### 오버레이 및 터널링 (Overlay & Tunneling)
#### 가상 터널링 프로토콜
##### VXLAN (Virtual Extensible LAN)
##### GRE (Generic Routing Encapsulation)
##### NVGRE
#### 가상화 성능 기술
##### SR-IOV (단일 루트 I/O 가상화)
##### DPDK (데이터 평면 개발 키트)

## 모니터링 및 분석 (Monitoring & Analysis)

### 네트워크 디버깅 도구 (CLI Tools)
#### 기본 연결성 확인
##### ping (ICMP 기반)
##### traceroute / tracert (경로 추적)
##### mtr (My Traceroute)
#### 상태 및 통계 확인
##### netstat (네트워크 통계)
##### ss (소켓 상태 확인)
##### ip (인터페이스 관리)
##### lsof (열린 파일 및 포트 확인)
#### DNS 및 라우팅 분석
##### nslookup / dig (DNS 질의)
##### route (라우팅 테이블 확인)
##### nmap (포트 스캐닝)

### 패킷 분석 및 스니핑 (Packet Analysis)
#### 패킷 캡처 도구
##### tcpdump (CLI 기반 캡처)
##### Wireshark (GUI 기반 분석)
#### 분석 핵심 개념
##### 패킷 덤프 (Packet Dump)
##### 프로토콜 계층 분석
##### 필터링 표현식 (BPF)
##### 스트림 추적 (Follow Stream)

### 네트워크 모니터링 지표 (Metrics)
#### 성능 메트릭 (Performance)
##### 대역폭 사용률 (Bandwidth Utilization)
##### 인터럽트 발생률 (Interrupt Rate)
##### 에러 및 드롭 패킷 (Errors/Drops)
#### 상태 모니터링
##### 활성 커넥션 수 (Active Connections)
##### 소켓 상태 분포 (TIME_WAIT 등)
##### 서버 응답 시간 (Response Time)

### 모니터링 아키텍처 (Observation)
#### 수집 및 시각화
##### 프로메테우스 (Prometheus)
##### 그라파나 (Grafana)
##### 대시보드 설계 원칙
#### 데이터 수집 방식
##### SNMP (Simple Network Management Protocol)
##### 흐름 기반 수집 (NetFlow / sFlow)
##### 로그 수집 (Syslog)
#### 경보 및 통보
##### 임계치 설정 (Threshold)
##### 알림 파이프라인 (Alerting)