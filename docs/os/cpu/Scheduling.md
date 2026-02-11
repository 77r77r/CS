# CPU Scheduling

> 작성일 : 2026.02.10  
> 작성자 : 노미경
---

## 1. 개념

**멀티 프로그램 운영체제에서 Ready Queue의 프로세스 중 어느 프로세스에게 CPU를 할당할지 결정하는 것**

### 1.1 관련 용어

-   **멀티프로그래밍**: CPU 이용률 제고. I/O 대기 시간 동안 다른 프로그램 실행
-   **멀티태스킹**: 응답 시간 단축. 시분할(Time Sharing)로 여러 앱을 동시에 쓰는 느낌 제공
-   **멀티프로세싱**: 하드웨어 구조. 하나 이상의 CPU(코어)로 여러 작업을 동시에 물리적 처리

### 1.2 CPU Burst vs I/O Burst

-   **CPU burst**: 실제 CPU를 사용하는 시간 (Running 상태)
-   **I/O burst**: I/O 대기 시간 (Waiting, Ready 상태)

**I/O bound가 CPU bound보다 많기 때문에 타임 쉐어링 시 효율 상승**

## 2. 목적

**CPU 활용도를 높이기 위해**

I/O를 기다리는 동안 CPU가 놀지 않도록 컨텍스트 스위치를 통해 시분할로 CPU 자원을 인터리빙하여 사용

## 3. Preemptive vs Non-preemptive

### 3.1 Non-preemptive (비선점형)

프로세스가 CPU를 자발적으로 release할 때까지 기다림

### 3.2 Preemptive (선점형)

스케줄러가 강제로 CPU를 뺏어올 수 있음

### 3.3 CPU 스케줄링이 발생하는 경우

1.  **Running → Waiting** (예: I/O 대기) → **Non-preemptive** (자발적)
2.  **Running → Ready** → **Preemptive** (비자발적)
3.  **Waiting → Ready** (예: I/O 종료) → **Preemptive** (비자발적)
4.  **Terminates** (종료) → **Non-preemptive** (자발적)

## 4. Scheduler와 Dispatcher

### 4.1 Scheduler

어떤 프로세스를 선택할지 결정

### 4.2 Dispatcher

**프로세스에게 CPU 제어권을 넘겨주는 모듈 (실제 Context Switch 수행)**

**역할:**

-   Context를 다른 프로세스로 전환
-   User mode로 변경

### 4.3 Dispatch Latency (디스패치 지연시간)

**짧아야 함**  
CPU 사용 시간보다 지연시간이 길어지면 계속 디스패치만 실행하는 문제 발생

**확인 명령어:**

```bash
vmstat
cat /proc/1/status | grep ctxt

```

## 5. 스케줄링 목표

1.  **CPU Utilization**: CPU를 가장 많이 사용
2.  **Throughput**: 단위 시간당 종결된 프로세스 수 최대화
3.  **Turnaround Time**: 프로세스 제출~종료 시간 최소화
4.  **Waiting Time**: Ready Queue 대기 시간 합 최소화 ⭐ **가장 중요**
5.  **Response Time**: 응답 시간 최소화 (예: UI)

**핵심: Waiting Time을 해결하면 나머지는 자동으로 개선됨**

## 6. Ready Queue 구현

### 6.1 자료구조

-   **Linked List**: 빈번한 삽입/삭제 효율성, 유연한 크기, 중간 삽입/삭제 가능
-   **Binary Tree**: 우선순위 관리 최적화 (정렬 상태 유지)
-   **FIFO Queue**: 단순 선입선출
-   **Priority Queue**: 우선순위 기반

**레디 큐의 자료구조 선택은 사용하는 알고리즘에 따라 달라짐**

-   단순 적재: Linked List
-   우선순위: Tree 계열

> **Q. Ready Queue가 꽉 차면?**  
> 운영체제 커널의 메모리 부족 상태. 시스템은 "자원 고갈" 상태가 되어 새로운 작업을 거부하고, 기존 작업들은 극도로 느려지며, 최악의 경우 시스템 안정성을 위해 기존 프로세스를 강제 종료하거나 시스템 다운

## 7. 스케줄링 알고리즘

### 7.1 FCFS (First-Come, First-Served)

**먼저 온 순서대로 처리**

-   FIFO Queue 사용
-   **Non-preemptive**
-   CPU-burst time에 따라 대기 시간이 크게 차이남

**문제점: Convoy Effect (호송 효과)**  
실행 시간이 매우 긴 프로세스 하나가 먼저 도착하여 짧은 프로세스들이 오래 기다려야 함

**현재는 거의 사용 안 함**

### 7.2 SJF (Shortest Job First)

**남은 시간이 가장 짧은 프로세스를 먼저 실행**

-   각 프로세스의 next CPU burst를 계산하여 가장 작은 것 배정
-   같다면 먼저 들어온 순서대로
-   **Non-preemptive**
-   **Provably optimal** (최적이라는 것이 증명 가능)

**변형: SRTF (Shortest Remaining Time First)** - Preemptive 버전

**이론적으로는 최고지만 계산(구현)이 어려워 실제로는 사용 안 함**

> **Q. 스레드의 실행 시간을 어떻게 알 수 있는가?**  
> 알 수 없음. Next CPU Burst를 예측하는 방법으로 근사치 계산:
>
> 1.  **커널 계층**: 지수 평균법 (Exponential Averaging)
> 2.  **응용 프로그램 계층**: 프로파일링 (Profiling)
> 3.  **코드 분석 계층**: 점근적 복잡도 (Big-O)
>
> **과거 측정 데이터 활용**: 과거에 많이 쓰는 프로세스는 이번에도 많이 쓸 것으로 예측

### 7.3 Priority Scheduling (우선순위 스케줄링)

**우선순위가 높은 프로세스를 먼저 실행**

-   SJF도 일종의 우선순위 스케줄링 (next CPU burst가 우선순위)
-   **문제점: Starvation (기아 현상)** - 낮은 우선순위 프로세스가 무한정 대기
-   **해결책: Aging** - 오래 대기한 프로세스에게 가중치 부여

### 7.4 RR (Round-Robin)

**시분할 (Time Sharing) 방식**

-   정해진 시간(Time Quantum)만큼만 CPU 사용 후 교체
-   **Preemptive**
-   **선점형 FCFS**
-   Circle Queue로 구현

**동작:**

-   CPU burst < Time Quantum: 자발적으로 종료
-   CPU burst > Time Quantum: Interrupt → Context Switch → Ready Queue

**보통 RR과 우선순위 스케줄링을 함께 사용** (우선순위 우선, 같으면 RR)

> **Q. RR을 단독으로 사용하지 않는 이유**
>
> 1.  **우선순위 무시**: UI 프로세스와 로그 수집 프로세스를 동일하게 취급 → UX 경험 감소
> 2.  **Time Quantum 설정의 딜레마**: 작을수록 응답성이 좋지만, 너무 작으면 잦은 문맥 교환으로 오버헤드 증가
> 3.  **I/O 바운드 프로세스의 손해**: 연산보다 대기가 많은 프로세스는 CPU 활용률 떨어짐
>
> **해결책**: MLFQ (Multi-Level Feedback Queue) 조합 방식 사용

### 7.5 MLQ (Multi-Level Queue)

**프로세스 우선순위에 따라 Ready Queue를 여러 개로 분리**

**Queue 계층 예시:**

1.  Real-time
2.  System
3.  Interactive
4.  Batch

각 큐마다 다른 스케줄링 알고리즘 적용 가능

### 7.6 MLFQ (Multi-Level Feedback Queue)

**MLQ + 동적 우선순위 조정**

**문제점**: 하위 레벨 큐에서 기아 현상 발생 가능

**해결책:**

-   상위 큐: 짧은 Time Quantum
-   하위 큐: 긴 Time Quantum
-   Aging 적용

**실전 OS에서 가장 많이 사용하는 스케줄링 방식**

## 8. Thread Scheduling

**실제로는 CPU 스케줄링이 아니라 Thread Scheduling을 수행**

### 8.1 Thread 종류

1.  **User Thread**: 스레드 라이브러리가 관리, 커널은 모름
2.  **Kernel Thread**: OS 커널에서 관리

**OS 커널은 Kernel Thread만 스케줄링**

## 9. 실시간 OS (Real-Time OS)

**주어진 시간 내에 작업을 완료해야 하는 시스템**

### 9.1 분류

-   **Soft Real-time**: 크리티컬한 프로세스가 먼저 실행 (약간의 오차 허용)
-   **Hard Real-time**: 오차 허용 불가 (데드라인 엄수 필수)

## 10. Future-knowledge Scheduling

미래의 작업 정보를 알고 있다고 가정한 이론적 스케줄링 (실제 구현 불가)