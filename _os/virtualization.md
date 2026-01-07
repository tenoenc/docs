### 가상화

가상화는 단일 컴퓨터 하드웨어에서 여러 개의 실행 환경(운영체제)을 동시에 실행할 수 있게 하는 기술이다. 각 실행 환경은 자신만의 독립적인 하드웨어를 가지고 있다고 착각하지만, 실제로는 가상화 소프트웨어가 물리적 자원을 논리적으로 분할하여 제공하는 것이다.

#### 가상화의 개념

운영체제는 기본적으로 자신이 하드웨어를 독점한다고 가정하고 설계되었다. 가상화는 이 가정을 깨고, **호스트(Host) 시스템**의 하드웨어 자원(CPU, 메모리, I/O)을 여러 **게스트(Guest) 시스템**이 나누어 쓸 수 있도록 추상화한다.



#### VMM (Virtual Machine Manager, Hypervisor)

VMM 또는 하이퍼바이저는 가상화를 가능하게 하는 핵심 소프트웨어 계층이다. 하드웨어와 게스트 OS 사이에 위치하며, 게스트 OS가 하드웨어에 접근하려고 할 때 이를 가로채서(Trap) 물리적 하드웨어에 매핑해준다.



- **Type 1 (Native/Bare-metal):** 하드웨어 바로 위에서 실행되는 하이퍼바이저 (예: Xen, VMware ESXi). 운영체제 없이 하이퍼바이저가 직접 하드웨어를 제어하므로 성능이 뛰어나다. 데이터센터나 클라우드 환경에서 주로 사용된다.
- **Type 2 (Hosted):** 호스트 OS(예: Windows, macOS) 위에서 응용 프로그램처럼 실행되는 하이퍼바이저 (예: VMware Workstation, VirtualBox). 설치가 쉽지만 호스트 OS를 거쳐야 하므로 오버헤드가 크다.

#### 가상화 vs 에뮬레이션 (Emulation)

두 기술 모두 "다른 환경을 흉내 낸다"는 점은 같지만, **CPU 아키텍처의 일치 여부**에서 결정적인 차이가 있다.

- **가상화 (Virtualization):** 호스트와 게스트의 CPU 아키텍처가 **동일**하다 (예: x86 CPU 위에서 x86 윈도우와 리눅스 실행). 대부분의 명령어를 CPU에서 직접 실행하므로 성능 저하가 적다.
- **에뮬레이션 (Emulation):** 호스트와 게스트의 CPU 아키텍처가 **다르다** (예: x86 PC에서 안드로이드 ARM 앱 실행). 에뮬레이터가 모든 기계어 명령을 소프트웨어적으로 번역해야 하므로 속도가 매우 느리다.

> **백엔드 관점: 클라우드와 컨테이너의 기반**
>
> 1. **IaaS (AWS EC2):** 아마존의 EC2 인스턴스는 거대한 물리 서버를 VMM(주로 Xen이나 KVM)을 통해 잘게 쪼갠 가상 머신이다. 우리가 "서버를 빌린다"는 것은 실제로는 "하이퍼바이저 위에서 돌아가는 격리된 공간"을 받는 것이다.
> 2. **Docker와의 차이:** Docker는 하드웨어를 가상화하는 것이 아니라 **OS 커널을 공유**하면서 프로세스 격리(Namespace, Cgroup)를 수행하는 **'OS 수준 가상화'**다. 이 때문에 Docker 컨테이너는 VMM을 거치는 오버헤드가 거의 없어 VM보다 훨씬 가볍고 빠르다. 하지만 "격리(Isolation)"라는 큰 틀에서 가상화 기술의 연장선에 있다.
{: .block-tip }

#### [Practice] 게스트 OS의 착각: "나는 진짜 서버인가?"

가상 머신(VM) 위에서 동작하는 운영체제는 자신이 실제 물리 하드웨어를 점유하고 있다고 믿는다. 하지만 하이퍼바이저(VMM)는 게스트 OS에게 가상의 자원을 제공하며 그 뒤에서 실제 자원을 가로채 관리한다. 시스템 명령어를 통해 운영체제가 보고 있는 '하드웨어 제조사' 정보가 실재하는 것인지 확인해보자.

##### 검증 시나리오 (Shell Script)

리눅스의 저수준 시스템 정보 조회 명령어인 `dmidecode`와 `lscpu`를 사용하여, 운영체제가 인식하는 시스템 제조사(System Vendor)와 하이퍼바이저 플래그를 확인한다.

```bash
#!/bin/bash

echo "=== [System Virtualization Check] ==="

# 1. lscpu를 통한 가상화 벤더 확인
# 물리 서버라면 'Hypervisor vendor' 항목이 나타나지 않거나 비어 있다.
VENDER_INFO=$(lscpu | grep "Hypervisor vendor")

if [ -z "$VENDER_INFO" ]; then
    echo "[Result] This system is likely a Physical Bare-metal server."
else
    echo "[Result] This system is a Virtual Machine."
    echo "Detail: $VENDER_INFO"
fi

echo -e "\n=== [DMI (Desktop Management Interface) Info] ==="

# 2. dmidecode를 통한 시스템 제조사 확인 (root 권한 필요)
# 가상 머신일 경우 VMware, Inc. 또는 VirtualBox 등이 출력된다.
sudo dmidecode -s system-manufacturer 2>/dev/null || echo "Unable to read DMI info (Check root permission)"

# 3. 추가 힌트: 시스템 모델명 확인
sudo dmidecode -s system-product-name 2>/dev/null
```

##### 실행 결과 및 해석

```text
=== [System Virtualization Check] ===
[Result] This system is a Virtual Machine.
Detail: Hypervisor vendor:       KVM

=== [DMI (Desktop Management Interface) Info] ===
QEMU
Standard PC (Q35 + ICH9, 2009)
```

```text
=== [System Virtualization Check] ===
[Result] This system is likely a Physical Bare-metal server.

=== [DMI (Desktop Management Interface) Info] ===
Dell Inc.
PowerEdge R740
```

-   **Hypervisor Vendor:** 가상 머신일 경우 `VMware`, `KVM`, `Microsoft` 등의 이름이 명시적으로 나타난다. 이는 운영체제가 인식하는 '하드웨어의 부모'가 물리 칩셋이 아닌 소프트웨어(VMM)임을 의미한다.
-   **추상화의 힘:** 게스트 OS는 실제 CPU가 Intel인지 AMD인지, 메모리가 어떤 제조사인지 정확히 알 필요가 없다. VMM이 제공하는 표준화된 인터페이스만 보고 동작하기 때문이다.

> **백엔드 관점: 가상화 환경의 성능 오버헤드 (Steal Time)**
> 
> 클라우드 환경(AWS, GCP 등)에서 백엔드 서버의 CPU 사용량이 100%가 아닌데도 애플리케이션이 느려질 때가 있다. 이때 리눅스의 `top` 명령어로 **`st` (Steal Time)** 수치를 확인해야 한다. 
> 이는 가상 머신이 CPU를 사용하려고 요청했으나, 하이퍼바이저가 다른 VM에게 자원을 할당하느라 '기다리게 만든 시간'을 의미한다. 운영체제가 하드웨어를 직접 소유하지 못하고 VMM의 스케줄링을 받아야 하는 가상화 환경 특유의 오버헤드이다.
{: .block-tip }