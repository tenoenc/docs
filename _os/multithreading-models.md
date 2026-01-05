### 멀티스레딩 모델

* **User Thread vs Kernel Thread:** 커널이 스레드를 인지하는가?
* **매핑 모델:**
* **Many-to-One:** 그린 스레드 (Green Thread) - 과거 자바 1.1
* **One-to-One:** 현대 OS(Linux, Windows)와 현재 자바의 모델
* **Many-to-Many:** Go 루틴 등 경량 스레드 모델


* **> 백엔드 관점:** **Java 21의 가상 스레드(Virtual Threads, Project Loom)**가 왜 등장했는지, 기존 One-to-One 모델의 한계(OS 스레드 생성 비용, 컨텍스트 스위칭 비용)와 비교하여 깊게 파고들어야 할 핵심 포인트입니다.