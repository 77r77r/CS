# Thread(스레드)

> 작성일 : 2026.02.06  
> 작성자 : 노미경
---

# 1. Thread

## 1.1 Thread의 정의

프로세스(Process) 내에서 실행되는 최소 단위의 작업 흐름

### 특징

- 경량 프로세스(LWP, Light-Weight Process)
- CPU 이용의 기본 단위 (멀티 스레딩 제공 시 CPU 점유의 가장 작은 단위가 Thread)
- 하나의 프로세스가 여러 개의 스레드의 흐름을 가질 수 있음  
  PC(Program Counter)와 Register set만 별도로 유지하면 같은 프로그램 안에서 여러 실행 흐름 관리 가능
- 프로세스 내 자원을 공유

## 1.2 Thread의 구조

1. Thread ID: 스레드 식별자
2. Program Counter(PC): 다음으로 실행할 명령어 주소
3. Register set: CPU 레지스터 집합
4. Stack: 함수 호출 및 지역 변수 저장

## 1.3 프로세스 vs 스레드

| 구분        | 프로세스        | 스레드           |
|-----------|-------------|---------------|
| **정의**    | 실행 중인 프로그램  | 프로세스 내부 실행 흐름 |
| **단위**    | 운영체제의 작업 단위 | 프로세스 내 실행 단위  |
| **자원**    | 독립적 메모리 공간  | 프로세스 자원 공유    |
| **생성 비용** | 높음          | 낮음            |

#### 용어

- Single thread of control: 흐름이 하나밖에 없는 프로세스
- 멀티프로세스 : 각 프로세스는 독립적으로 실행되며 각각 별개의 메모리를 차지
- 멀티 프로세싱 : 여러 개의 싱글 스레드 프로세스가 동시에 실행, 독립된 메모리 공간
- 멀티스레드 : 프로세스 내의 메모리를 공유
- 멀티 스레딩 : 하나의 프로세스 내에서 여러 개의 스레드가 메모리를 공유

## 1.4 Thread 자원

- 공유 자원 : Code / Data / Files
- 독립 자원 : Registers / PC / Stack

# 2. Multi Thread

## 2.1 Multi Thread이란?

하나의 프로세스가 여러 개의 스레드로 동시에 여러 작업을 수행하는 상태

## 2.2 Multi Threading

### 장점

#### Responsiveness (응답성)

- 일부 스레드가 차단되어도 나머지 부분은 계속 실행 가능

#### Resource Sharing (자원 공유)

code와 data 영역을 공유하기 때문에 shared-memory처럼 사용 가능

#### Economy (경제적)

프로세스 생성 및 문맥 교환보다 오버헤드가 훨씬 낮음  
코드 영역을 복사할 필요가 없어서 생성보다 비용이 저렴

> Context switching vs Thread switching 차이? 왜 더 경제적인가?  
> 프로세스 간 문맥 교환은 독립된 주소공간과 메모리 관리 정보를 교체해야 함  
> 쓰레드 스위칭은 쓰레드 공유의 정보만 교체하면 되어서 오버헤드가 훨씬 낮음

#### Scalability (확장성)

멀티코어 시스템에서 병렬 실행을 통해 성능을 높일 수 있음  
Multi Processor 아키텍처(=코어가 여러 개)의 경우, 각 스레드를 연결하여 병렬 처리 가능

### 단점

각각의 스레드 중 어떤 것이 먼저 실행될지 그 순서를 알 수 없음

- 실행 조건에 따른 다른 결과 (경쟁조건) → 세마포어 방법을 통해 해결 가능

> 세마포어?  
> 공유 자원에 동시에 접근할 수 있는 스레드(또는 프로세스) 수를 제한하는 동기화 도구
> ```java
> Semaphore semaphore = new Semaphore(permits); // 허용 개수
> 
> semaphore.acquire();   // 진입 (wait / P)
> try {
>   // 임계 구역 (공유 자원 사용)
> } finally {
>   semaphore.release(); // 퇴장 (signal / V)
> }
> ```

# 3. Thread 구현

## 3.1 Java에서 Thread 구현 방법

### 3.1.1 방법 1: Thread 클래스 상속

```java
class MyThread extends Thread {
	@Override
	public void run() {
		// 스레드가 실행할 코드
		while (true) {
			// 보통 무한 반복하면서 작업 수행
			// 하나만 반복할 거면 스레드 안 쓰니까
		}
	}
}

// 사용
MyThread thread = new MyThread();
thread.start(); // 내부적으로 run() 호출
```

**특징**:

- `extends Thread` → `run()` 오버라이딩
- `thread.start()` 호출 → 내부적으로 `run()` 호출
- `run()`이 실행되고 context switch 발생
- 컨텍스트 스위치 전에는 main 스레드 그대로 진행

**단점**:

- Java는 다중 상속을 지원하지 않음 (2번 방식으로 개선)

**종료 처리**:

- 종료를 위한 `InterruptedException` 예외 처리 필요

### 3.1.2 방법 2: Runnable 인터페이스 구현

```java
class MyRunnable implements Runnable {
	@Override
	public void run() {
		// 스레드가 실행할 코드
	}
}

// 사용
Thread thread = new Thread(new MyRunnable());
thread.start();
```

**특징**:

- Runnable 인터페이스를 구현하여 `run()` 오버라이딩
- 다중 상속 문제 해결
- 더 유연한 설계 가능

### 3.1.3 방법 3: Lambda 익명 함수

```java
Runnable task = () -> {
	// 스레드가 실행할 코드
};

Thread thread = new Thread(task);
thread.start();
```

**특징**:

- Java 8 이상에서 사용 가능
- 간결한 코드 작성
- 함수형 프로그래밍 스타일

## 3.2 주요 메서드

### 3.2.1 thread.start()

- 스레드를 시작
- 내부적으로 `run()` 메서드 호출
- 새로운 실행 흐름 생성

### 3.2.2 thread.join()

- **대기**: 자식 스레드 수행 후에 부모 스레드 수행
- 현재 스레드가 대상 스레드의 종료를 기다림

```java
Thread childThread = new Thread(() -> {
	// 자식 스레드 작업
});

childThread.start();
childThread.join(); // 자식이 끝날 때까지 대기
// 자식 종료 후 계속 진행
```

### 3.2.3 thread.interrupt()

- 스레드 종료 요청
- `InterruptedException` 발생시킴


# 4. Multicore 프로그래밍

## 4.1 Multicore 시스템의 Multithreading

### 4.1.1 싱글 코어

- **Interleaved(인터리빙)**: 사이사이에 끼어 넣는다 (타임 쉐어링)
- **동시성(Concurrency)** 제공
- 실제로는 순차 실행이지만 빠른 전환으로 동시 실행처럼 보임

### 4.1.2 다중 코어

- **Parallel(병렬)** 가능, **병렬성(Parallelism)** 제공
- 진정한 동시 실행 가능


# 5. 멀티쓰레딩 모델

## 5.1 User Thread (사용자 스레드)

- **운영체제와 무관함**, 사용자 모드에서 관리
- User 모드 위에서 쓰레딩
- 커널의 개입 없이 사용자 수준에서 관리
- 생성 및 관리 오버헤드가 낮음

## 5.2 Kernel Thread (커널 스레드)

- **운영체제가 직접 관리**하는 스레드
- **CPU 코어를 직접 제어**할 수 있음
- OS가 직접 스케줄링
- 진정한 병렬 실행 가능

#### 참고 자료

하이퍼스레딩
문맥교환을 하는 순간에도 쓰레드가 돌기때문에 애초에 스레드를 2개를 들고옴
하나가 끝나면 다른걸 해.
처음부터 2개를 가져오면 