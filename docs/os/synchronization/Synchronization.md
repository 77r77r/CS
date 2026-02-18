# Synchronization : 프로세스 동기화

> 작성일 : 2026.02.13  
> 작성자 : 노미경
---

# 1. Synchronization

## 1.1 배경

여러 프로세스가 공유 데이터에 동시에 접근할 때 **Data Inconsistency(데이터 불일치)** 발생

- 하나의 CPU에서 Context Switch로 인해 처리 중인 작업 중간에 다른 프로세스가 끼어드는 경우
- 분리된 CPU에서 동시에 실행되는 경우

> **Race Condition** : 경쟁 상태  
> 두 개 이상의 프로세스가 공유 데이터에 동시 접근할 때, 접근 순서에 따라 결과가 달라지는 것

## 1.2 해결 방법

**Synchronization (동기화)**: 특정 시간에 한 개의 프로세스만 공유 데이터를 다룰 수 있도록 접근을 순차적으로 제어

### Single-core 환경

- 인터럽트가 발생하지 않도록 차단
- 단, 복잡한 환경에서는 사용 불가
- 멀티프로세서 환경에서는 장점이 사라지므로 적합하지 않음

### 커널 방식

- **비선점형 커널**: Race Condition 발생 가능성 낮음
- **선점형 커널**: 응답성이 좋아 대부분의 OS에서 선호, Race Condition 처리 필요

### Java Threads

- `static` 선언으로 저장 공간 공유
- `synchronized` 키워드로 Critical Section 보호

# 2. Critical-Section Problem(CSP)

## 2.1 Critical-Section : 임계 구역

여러 개의 프로세스(또는 스레드)가 공유 데이터를 수정하려 할 때, 한 번에 하나의 프로세스만 접근해야 하는 코드 구역

### 소스 코드 영역 구분

- **Entry Section**: 임계 영역 진입 전 락 획득을 시도하는 코드 `lock()`
- **Critical Section**: 공유 자원에 접근하여 데이터를 수정하는 코드, 동시에 하나의 스레드만 실행
- **Exit Section**: 임계 영역 사용 후 락 해제 `unlock()`
- **Remainder Section**: 임계 영역과 무관한 나머지 코드

> **Q. 임계 영역의 구분 방법**
> - 공유 자원 사용 여부  
    > 공유 자원: 전역 변수, static 변수, Heap 객체, 파일, DB, Buffer, Queue 등
> - 데이터 수정 여부 (Read-Write)
> - 연산 사이에 다른 스레드의 작업이 끼어들 수 있는지

## 2.2 해결하기 위한 요구사항

아래 세 가지 요건을 모두 만족해야 해결이라 할 수 있지만, 현실적으로 어렵기 때문에 2, 3번은 발생 시 해결하는 방식으로 진행

### 1. Mutual Exclusion (상호 배제)

하나의 프로세스가 Critical Section에서 실행 중이면 다른 프로세스는 진입 불가  
단독으로는 Deadlock, Starvation 문제를 해결하지 못함

### 2. Progress (진행, Deadlock 회피)

Critical Section에 진입이 무한정 연기되는 상황 없어야 함

### 3. Bounded Waiting (한정 대기, Starvation 회피)

대기 시간에 제한을 두어 접근하지 못하는 프로세스가 없어야 함

> **Deadlock?**  
> 서로가 서로의 자원을 기다리며 아무도 움직이지 못하는 무한 대기 상태

> **Starvation?**  
> 특정 프로세스나 스레드가 자원을 할당받지 못하고 무한정 대기하는 상태

## 2.3 Software Solutions

#### Dekker Algorithm

- 2개의 프로세스가 경쟁할 때 사용
- flag[](의사표시) + turn(양보 변수) 조합
- 구현이 복잡하고 busy waiting 사용
- 최초로 올바름이 증명된 소프트웨어 상호배제 알고리즘

#### Eisenberg and McGuire's Algorithm

- N개의 프로세스가 경쟁할 때 사용
- 각 프로세스 상태 관리 + 순번 기반 접근
- 구현 복잡, busy waiting, 확장성/실용성 낮음
- Dekker를 N개로 일반화한 시도

#### Bakery Algorithm

- N개의 프로세스가 경쟁할 때 사용
- 프로세스들이 번호표를 받아 번호가 낮은 프로세스 부터 실행
- 번호가 같다면 프로세스 번호(ID)가 작은 쪽이 우선권을 가짐
- 번호표 숫자가 무한히 커질 수 있음
- 번호 비교 비용, busy waiting, 현대 메모리 모델에서 안전하지 않음
- 공정성(fairness)을 명확히 설명하는 대표 알고리즘

#### Peterson's Algorithm

```text
Dekker = 이론적 완성도
Peterson = 이해와 설명의 최적화
```

# Peterson's Algorithm

## 3.1 개념

두 개의 프로세스가 공유 자원을 안전하게 사용(상호 배제)할 수 있도록 설계된 알고리즘

- 임계 영역 문제를 가장 완벽하게 해결
- 깃발(flag)와 차례(turn)를 모두 만족하면 Critical Section 수행

```c
int turn;
boolean flag[2];

// Pi 프로세스
while (true) {
    flag[i] = true;
    turn = j;
    while (flag[j] && turn == j)
        ;
    
        /* critical section */
    
    flag[i] = false;
    
        /* remainder section */
}

// Pj 프로세스
while (true) {
    flag[j] = true;
    turn = i;
    while (flag[i] && turn == i)
        ;
    
        /* critical section */
    
    flag[j] = false;
    
        /* remainder section */
}
```

- 상호 배제, 진행, 한정 대기를 모두 만족하는 이론적 모델

> 상호 배제 : 두 프로세스가 동시에 while문을 통과하려면 turn이 0인 동시에 1이어야 함(불가능)     
> 진행 : 임계 영역 밖에 있는 프로세스는 자신의 flag를 false로 유지  
> 한정 대기 : 임계 영역을 나온 뒤 재진입 전에 반드시 turn을 상대방에게 넘김

> Q. 왜 flag 변수 하나만 사용하는 게 아니라 turn 이라는 값도 사용할까?  
> flag만 사용 : 서로 동시에 들었을 때 충돌 발생  
> turn만 사용 : 들어가고 싶지 않은 경우 대기 발생 발생

## 3.2 실제 현대 컴퓨터 시스템에서의 한계

Entry Section에서 권한을 확인하는 과정 자체가 문맥 교환에 의해 깨질 수 있음

> 현대의 CPU와 컴파일러는 실행 속도를 높이기 위해 서로 의존성이 없는 코드의 순서를 임의로 바꿔서 실행

1. **명령어 재배치 문제 (Instruction Reordering)**  
   실행 속도를 높이기 위해 의존성이 없는 코드의 순서를 임의로 변경
2. **메모리 가시성 문제 (Memory Visibility)**  
   멀티코어 CPU는 각 코어마다 별도의 캐시를 사용
3. **Busy Waiting에 의한 자원 낭비**  
   임계 영역에 들어가지 못하는 동안 `while` 루프로 CPU 낭비

> \* **게런티(guarantee) 없음**  
> 이 알고리즘이 모든 환경에서 임계 영역의 요구조건을 항상 만족한다고 보장할 수 있는가

## 3.3 현대 시스템에서의 해결책

1. Memory Barriers/Fences (메모리 장벽) : 메모리 연산의 순서를 보장하는 하드웨어 명령어
2. Hardware Instructions (하드웨어 지원) : 하드웨어 수준에서 원자적 연산을 지원
3. test_and_set : 하드웨어에서 지원하는 원자적 명령어로 상호 배제 구현
4. Atomic Variables (원자적 변수)

## 3.4 Atomic Variables (원자적 변수)

- Atomicity(원자성) : 더 이상 쪼갤 수 없는 명령(uninterruptible)으로 구성 (atomic operation, ex. 함수)
- 단일 변수에 대한 경쟁 상태를 상호 배제로 처리
- `java.util.concurrent.atomic.AtomicBoolean` 제공

```text
AtomicBoolean[] flag;
flag[] = new AtomicBoolean();
flag[].set(true);
flag[].get(true);
```

## Java로 피터슨 알고리즘 구현


