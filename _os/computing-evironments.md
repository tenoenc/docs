### 다양한 컴퓨팅 환경

운영체제는 단일 컴퓨터를 넘어 다양한 환경에서 동작한다. 현대의 백엔드 개발은 대부분 단일 시스템이 아닌, 네트워크로 연결된 복합적인 환경 위에서 이루어진다.

#### 분산 시스템 (Distributed Systems)

물리적으로 떨어져 있는 여러 개의 컴퓨터 시스템이 네트워크(LAN, WAN)를 통해 연결되어, 사용자가 마치 하나의 거대한 시스템을 사용하는 것처럼 리소스를 공유하는 환경이다.

![](/docs/assets/img/2026-01-05-09-05-35.png)

- **네트워크 프로토콜:** 주로 **TCP/IP** 프로토콜 스택을 기반으로 통신한다.
- **목적:** 자원 공유(Resource Sharing), 연산 속도 향상(Computation Speedup), 신뢰성(Reliability) 확보.

#### 클라이언트-서버 컴퓨팅 (Client-Server Computing)

분산 시스템의 가장 대표적이고 일반적인 형태다. 네트워크상의 시스템을 **요청자(Client)**와 **제공자(Server)**로 명확히 구분한다.

- **컴퓨트 서버 (Compute-server):** 클라이언트가 요청한 작업을 수행하고 결과만 돌려주는 시스템 (예: Web Server, Database Server).
- **파일 서버 (File-server):** 클라이언트가 파일을 생성, 조회, 수정, 삭제할 수 있도록 파일 시스템 인터페이스를 제공하는 시스템.

> **백엔드 관점: 우리의 주 무대**
>
> 백엔드 개발자가 만드는 Spring Boot 애플리케이션이 바로 **'컴퓨트 서버'**에 해당한다. 클라이언트(브라우저, 모바일 앱)로부터 REST API 요청을 받아 비즈니스 로직을 수행하고, JSON 데이터를 응답하는 구조가 이 모델의 전형이다.
{: .block-tip }

#### 피어 투 피어 (Peer-to-Peer)

클라이언트와 서버의 엄격한 구분이 없는 모델이다. 시스템상의 모든 노드(Peer)가 클라이언트가 될 수도 있고, 동시에 서버 역할을 할 수도 있다.

- **동작 방식:** 중앙 서버를 거치지 않고 노드끼리 직접 통신하거나, 중앙 서버는 피어들의 주소만 관리하고 실제 데이터 전송은 피어 간에 일어난다.
- **예시:** 토렌트(BitTorrent), 블록체인 네트워크.

#### 클라우드 컴퓨팅 (Cloud Computing)

연산 기능, 스토리지, 애플리케이션을 네트워크(인터넷)를 통해 서비스 형태로 제공하는 컴퓨팅 유형이다. 가상화(Virtualization) 기술을 기반으로 한다.

- **SaaS (Software as a Service):** 인터넷을 통해 사용할 수 있는 소프트웨어 (예: Google Docs, Slack).
- **PaaS (Platform as a Service):** 애플리케이션 개발 및 배포를 위한 플랫폼 제공 (예: Heroku, Google App Engine).
- **IaaS (Infrastructure as a Service):** 서버, 스토리지 등 하드웨어 자원을 가상화하여 대여 (예: **AWS EC2**, GCP Compute Engine).

##### [Practice] Infrastructure as Code (IaC) with AWS CDK

과거에는 서버를 증설하려면 엔지니어가 물리적인 하드웨어를 구매하고, 랙(Rack)에 장착하고, OS를 설치해야 했다. 하지만 클라우드 환경(IaaS)에서는 이 모든 물리적 과정이 **소프트웨어 코드(Code)**로 대체된다.

이를 증명하기 위해, Java로 작성된 **AWS CDK(Cloud Development Kit)** 코드를 살펴본다. 이 코드는 단순한 객체 생성이 아니라, 실제 데이터센터의 물리 자원을 제어하는 설계도이다.

###### 검증 코드 (Java)

`Instance` 객체를 생성하고 속성을 설정하는 행위가 실제 가상 서버(VM)를 프로비저닝하는 과정과 1:1로 매핑된다.

```java
import software.amazon.awscdk.core.Construct;
import software.amazon.awscdk.core.Stack;
import software.amazon.awscdk.services.ec2.*;

public class InfrastructureStack extends Stack {
    public InfrastructureStack(final Construct scope, final String id) {
        super(scope, id);

        // 1. 가상 네트워크 (VPC) 정의
        // 물리적 데이터센터의 네트워크 케이블링 작업을 소프트웨어로 정의
        Vpc vpc = Vpc.Builder.create(this, "TenoLabVpc")
            .maxAzs(2) // 2개의 가용 영역(데이터센터)에 걸쳐 구축 (재해 복구 고려)
            .build();

        // 2. 가상 서버 (EC2 Instance) 정의
        // 물리 서버 조립 과정을 객체 생성으로 대체
        Instance webServer = Instance.Builder.create(this, "TenoWebServer")
            .vpc(vpc)
            .instanceType(InstanceType.of(InstanceClass.BURSTABLE3, InstanceSize.MICRO)) // CPU/RAM 사양 결정
            .machineImage(MachineImage.latestAmazonLinux2()) // OS 설치 (Linux)
            .build();
    }
}
```

###### 코드 해석 및 의의

-   **`Vpc` (Virtual Network):** 물리적인 라우터나 스위치 설치 없이, 코드 한 줄로 논리적인 격리 네트워크망을 구축한다.
-   **`InstanceType` (Hardware Spec):** `t3.micro` 같은 타입을 지정하는 것은 메인보드에 CPU와 RAM을 꽂는 행위를 추상화한 것이다.
-   **`MachineImage` (OS Installation):** 설치 디스크(CD/USB)를 넣고 부팅하는 과정이, 이미 빌드된 OS 이미지를 참조하는 것으로 대체되었다.

결론적으로 클라우드 환경에서 운영체제와 하드웨어는 더 이상 물리적인 실체가 아니라, **API로 제어 가능한 소프트웨어 오브젝트**가 되었다.

> **백엔드 관점: 서버리스와 클라우드 네이티브**
>
> 과거에는 직접 물리 서버(On-premise)에 OS를 설치했지만, 이제는 AWS 같은 클라우드 벤더가 제공하는 **IaaS** 위에서 개발합니다. 더 나아가 OS 관리조차 필요 없는 **Serverless (AWS Lambda)** 환경으로 넘어가고 있습니다. 하지만 이 모든 편리함의 기저에는 OS의 가상화 기술과 분산 시스템 이론이 깔려 있음을 기억해야 한다.
{: .block-tip }