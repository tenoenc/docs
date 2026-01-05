### 스케줄링 알고리즘

* **FCFS, SJF:** 기본 이론 (SJF의 최적성과 예측 불가능성)
* **라운드 로빈 (Round Robin, RR):** 시분할 시스템의 핵심. Time Quantum 크기에 따른 트레이드오프.
* **우선순위 스케줄링 (Priority Scheduling):** 기아 상태(Starvation)와 해결책인 에이징(Aging).
* **멀티레벨 큐 (Multilevel Queue) & 멀티레벨 피드백 큐 (MFQ):** 현실적인 OS 스케줄러의 기반.
* **> 백엔드 관점:** 웹 서버(Nginx, Tomcat)는 처리량(Throughput)이 중요한가, 응답 시간(Response Time)이 중요한가? 목적에 따라 스케줄링의 평가 기준이 달라짐을 이해하세요.