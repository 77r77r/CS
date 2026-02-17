#프로세스 동기화 도구

> 작성일 : 2026.02.13  
> 작성자 : 노미경
---

# 1. Synchronization Tools

상호 배제 문제를 해결해 줌

## 1. Mutex Locks : 뮤텍스

- 상호 배제를 구현하는 가장 간단한 동기화 툴
- 2개 가능
- mutex = mutual exclusion
- 임계 영역에 진입 시 열쇠를 받고 들어가고 나오면 반납하는 구조
- Lock (열쇠)를 사용함
- acquire(), release() 두 개의 명령을 사용
- available 이라는 Boolean variable 사용
-

```text
while (true) {
    acquire() lock    // lock(열쇠) 획득 : entry-section
        critical section
    release() lock    // lock(열쇠) 반납 : exit-section
        remainder section
}
```

- Busy waiting 발생
- Spinlock : Busy waiting이 발생하는 뮤텍스 락  
  놀고 있는 cpu 코어가 여러개 있는 경우에는 문제가 되지 않음 (멀티코어 시스템)
  context switch의 발생을 방지함으로 시간을 아낄 수 있음

- 데드락과 기아 문제는 해결되지 않음

## 2. Semaphore : 세마포어

- N개 제어 가능
- N개의 프로세스 해결
- 신호기(Semaphore)를 이용하여 해결하자
- integer variable의 초기화 값에 따라 달라짐
- (wait(), signal()) / (P(), V())의 명령어를 사용
  > \* P() = Proberen(to test), V() = Verhogen(to increment)

```text
// S = 인스턴스 갯수
wait(S) {
    while (S <= 0)
        ; // busy wait
    S--;
}

signal(S) {
    S++;
}
```

### Binary Semaphore

- s = 1로 초기화
- mutex lock과 동일

### Counting Semaphore

- s = n으로 초기화
- 무한대로 늘어나니 여러개의 인스터스를 가진 자원들에 사용
- wait() / P() : 자원을 감소
- signal() / V() : 자원을 증가
- S = 0 모든 리소스 사용 중으로 대기

### 구현

- busy waiting 발생 (해결하고 싶음)
- wait() 실행시에 waiting queue에 넣는 방식으로 해결 가능 : `sleep()`
- signal() 실행 시에 ready Queue에 넣는 방식으로 해결 : `wakeup()`

\*세마포어의전제조건 : n개의 인스턴스가 있다. = 공유 자원 X
공유자원 1개를 사용하는 경우에는 동일하게 경쟁 상태가 발생함

## 3. Monitor : 모니터

- 최근 Java에서 사용하는 동기화 방법
- 세마포어에 잘못된 시퀀스를 사용하면 timing errors 발생
- wait() -> signal()의 순서를 지키지 않으면 오류 발생함
- 아무튼 개발자 실수로 오류가 발생할 가능성이 높음

=> 이 문제를 해결하기 위해 monitor 를 사용 (에러 발생 확률이 적음)

### 3.1 정의

상호배제를 제공해주는 데이터 타입, 자료구조  
monitor 내부에 작성된 함수들을 동기화 코드라고 인식함

### 3.2 모니터의 구조

Shared Data: 공유 데이터
Operations : 연산/함수
Initialization Code : 초기화 코드

Entry Queue : 진입 큐, 프로세스 대기 상태(Blocked), 상호 배제 목적
Condition Queue : 대기 큐, 각 조건 변수마다 별도의 큐를 가짐, 실행 순서 제어 목적

### 3.3 Conditional Variables

Condition Variable: 특정 조건이 만족될 때까지 프로세스나 스레드를 대기 상태로 머물게 하는 동기화 도구

Conditional Variables의 추가 필요.  
동기화 매커니즘을 제공
condition variable의 `wait()` 와 `signal() / notify()` 호출

- 각 조건 변수마다 별도의 `Condition Queue`를 가짐
- signal()로만 깨어남

### 3.4 모니터 내부 흐름

1. 진입 : 프로세스 모니터 인입
2. 검사 : 필요 자원 여부 확인
3. 대기(wait) : 자원이 없는 경우 -> Condition Queue 인입, Entry Queue에 있던 다른 프로세스 인입
4. 신호(signal): 다른 프로세스가 인입하여 자원 처리 후 signal 호출
5. 재개 : Condition Queue에 있던 프로세스가 깨어나 작업 진행

\* 추가 : sinnal() 호출시에
Hoare 방식 : wait()
Mesa 방식 :

### 3.5 Java Monitors

- monitor-lock(intrinsic-lock) 사용
- 자바의 기본 단위는 Thread

#### `synchronized` keyword

명시적으로 임계영역에 해당하는 코드 블록을 선언할 때 사용하는 자바 키워드  
명시적으로 임계 영역이라고 선언  
모니터 락을 획득해야 진입 가능
모니터락을 가진 객체 인스턴즈 지정 가능  
메소드에 선언 시, 메소드 코드 블록 전체가 임계 영역으로 지정  
-> 모니터락을 가진 객체 인스턴스는 this 객체

```java
synchronized (object){
	// critical section
	}

public synchronized void 메소드명() {
	// critical section
}
```

#### wait() and notify()

- java.lang.Object 클래스에 선언 : 모든 자바 객체가 가짐
- Thread가 wait() 메소드 호출
- Thread가 notify() 메소드 호출 -> 대기 중인 Thread 하나를 깨움
- Thread가 notifyAll() 메소드 호출 -> 대기 중인 Thread 전부를 깨움(이후 ready Queue에서 경쟁)

#### 예시

```java
// 클래스 전체를 잠그기
static class Counter {
	public static int count = 0;

	synchronized public static void increment() {
		count++;
	}
}

// 특정 객체를 지정해서 잠그기
static class Counter {
	private static Object object = new Object();
	public static int count = 0;

	public static void increment() {
		synchronized (Object) {
			count++;
		}
	}
}
```

#### -

- 같은 인스턴스 변수를 넘겨준다면 서로다른 Thread가 `this`를 공유함

## 4. Liveness

- 데드락과 기아 문제를 해결하기 위한 방법
- 상호 배제 뿐만 아니라 데드락도 같이 해결

### 4.1 DeadLock

두 개 이상의 프로세스가 영원히 기다리는 것
대기 중인 프로세스가

### 4.2 Priority Inversion : 우선 순위 역전

높은 우선순위를 가진 프로세스가 낮은 우선순위의 프로세스에게 밀리는 현상

- 우선순위 상속으로 해결 가능(priority-inheritance)

