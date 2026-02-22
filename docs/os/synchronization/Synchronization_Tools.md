# Synchronization Tools : 프로세스 동기화 툴

> 작성일 : 2026.02.13  
> 작성자 : 노미경
---

# 0. Synchronization Tools?

동기화 툴이란, 여러 프로세스나 스레드가 공유 자원(데이터, 메모리 등)에 동시에 접근할 때 발생하는  
충돌을 방지하고 실행 순서를 제어하는 모든 메커니즘

### 동기화 툴의 종류

#### 1. 하드웨어/기초

- **Atomic Instructions**    
  "값 읽기+수정"을 중간에 끊기지 않게 한 번에 처리하는 CPU 명령어 (예: test_and_set)

- **Memory Barrier**  
  CPU가 성능 때문에 코드 실행 순서를 제멋대로 바꾸지 못하게 막는 장벽

#### 2. OS/커널

- **Mutex (뮤텍스)**    
  딱 하나의 열쇠를 가진 사람만 자원을 쓸 수 있게 하는 '화장실 자물쇠' 방식

- **Semaphore (세마포어)**  
  열쇠가 여러 개일 수 있습니다. 가용 자원의 개수를 세는 '주차장 관리 시스템' 방식

- **Spinlock (스핀락)**  
  열쇠가 생길 때까지 잠들지 않고 계속 확인하며 버티는 방식

#### 3. 언어/라이브러리

- **Monitor (모니터)**  
  자물쇠와 공유 데이터를 하나로 묶어 관리하는 '자동 세차기' 같은 도구  
  개발자가 직접 락을 걸고 푸는 실수를 하지 않도록 도움 (예: Java의 synchronized)

### 3. 동기화 툴이 보장해야 하는 3조건

모든 동기화 툴은 아래 세 가지를 완벽히 해결해야 '성공적'이라고 평가받는다.

1. **상호 배제 (Mutual Exclusion)** : 한 번에 한 명만 들어가야 한다.
2. **진행 (Progress)** : 아무도 없으면 들어가고 싶은 사람이 바로 들어갈 수 있어야 한다.
3. **한정 대기 (Bounded Waiting)** : 기다리는 사람은 언젠가는 들어갈 수 있어야 한다. (무한 대기 금지)

---

# 1. Mutex Locks : 뮤텍스

상호 배제를 구현하는 가장 간단한 동기화 툴

- `mutex` = mutual exclusion(상호 배제)
- 임계 영역 진입 시 열쇠(Lock)를 획득하고, 나올 때 반납하는 구조
- `acquire()` / `release()` 두 개의 명령 사용
- `available` Boolean 변수로 Lock 상태 관리
- Busy Waiting 발생 : 락을 기다리는 동안 CPU를 계속 점유하며 대기하는 현상
- Deadlock과 Starvation 문제는 해결되지 않음

```text
while (true) {
    acquire() lock    // lock 획득 : entry-section
        critical section
    release() lock    // lock 반납 : exit-section
        remainder section
}
```

## **Spinlock**

- Busy Waiting이 발생하는 뮤텍스 락
- 놀고 있는 CPU 코어가 여러 개인 멀티코어 시스템에서는 문제되지 않음
- Context Switch 발생을 방지하여 시간 절약 가능

# 2 Semaphore : 세마포어

N개의 프로세스 동시 접근 제어 가능

- `wait()` / `signal()` 또는 `P()` / `V()` 명령어 사용
    - `P()` = Proberen (to test)
    - `V()` = Verhogen (to increment)

> **\*wait() 연산의 논리**
> 1. 확인: "지금 남는 자원(열쇠)이 있나?"
> 2. 대기: "없으면? 있을 때까지 기다려(wait)."
> 3. 획득: "있으면? 내가 하나 집어서(값 1 감소) 들어갈게.
>
> **스레드가 wait() 호출 후 통과 = 자원을 성공적으로 획득**

```text
// S = 인스턴스 개수
wait(S) {
    while (S <= 0)
        ; // busy wait
    S--;
}

signal(S) {
    S++;
}
```

## Binary Semaphore

- S = 1로 초기화
- Mutex Lock과 동일하게 동작

### Binary Semaphore와 Mutex의차이

```text
1. 소유권 (Ownership)

- 뮤텍스 (Mutex) : 소유권 O  
  Lock을 건 스레드만 Unlock 할 수 있음

- 이진 세마포어 (Binary Semaphore) : 소유권 X  
  스레드 A가 자원을 쓰기 시작(wait)했어도, 스레드 B가 신호(signal)를 보내서 자원을 해제하거나 강제로 뺏는 것이 가능

2. 사용 목적

- Mutex  
  공유 자원을 보호하기 위한 상호 배제(Mutual Exclusion)가 목적

- Binary Semaphore  
  다른 스레드에게 순서를 알려주는 용도, 동기화(Synchronization) 및 신호(Signaling)가 목적

3. 우선순위 역전 (Priority Inversion) 해결 여부

- Mutex 
  소유권 개념으로 인해 '우선순위 상속(Priority Inheritance)' 기술 적용하여 해결 가능
  
- Binary Semaphore  
  누가 주인인지 모르기 때문에 우선순위 상속이 불가능하여, 우선순위 역전 현상에 취약
```

## Counting Semaphore

세마포의의 정수 값(S)이 1 이상의 값을 가질 수 있는 세마포어

- S = n으로 초기화
- 여러 개의 인스턴스를 가진 자원에 사용
- `wait()` / `P()`: 자원 감소
- `signal()` / `V()`: 자원 증가
- S = 0: 모든 리소스 사용 중 → 대기

## Busy Waiting 해결

- `wait()` 실행 시 Waiting Queue에 넣는 방식: `sleep()`
- `signal()` 실행 시 Ready Queue에 넣는 방식: `wakeup()`

> 세마포어 변수를 수정하는 도중에 문맥 교환이 일어나면, 상호 배제가 깨질 수 있음  
> 아래 방법으로 해결 가능
> - 인터럽트 금지 (Interrupt Disabling)
> - 하드웨어 원자적 명령 (Atomicity)

# 3. Monitor : 모니터

세마포어의 잘못된 사용 순서(`wait()` → `signal()` 순서 미준수)로 인한 오류를 방지하기 위해 등장  
개발자 실수로 인한 오류 발생 가능성을 낮춘 고수준 동기화 도구

## 3.1 정의

상호 배제를 제공하는 데이터 타입 및 자료구조  
모니터 내부에 작성된 함수들을 동기화 코드로 인식

## 3.2 모니터의 구조

- **Shared Data**: 공유 데이터
- **Operations**: 연산/함수
- **Initialization Code**: 초기화 코드
- **Entry Queue**: 진입 큐 - 상호 배제 목적 (1개)
- **Condition Queue**: 대기 큐 - 실행 순서 제어 목적 (n개)

### Condition Variable (조건 변수)

특정 조건이 만족될 때까지 프로세스나 스레드를 대기 상태로 머물게 하는 동기화 도구  
개발자가 임계구역 표시만 하면 컴파일러가 알아서 처리해줌

- 각 조건 변수마다 별도의 `Condition Queue` 보유
- `wait()`: 조건 미충족인 경우, 현재 보유한 모니터 락 해제 후 대기 상태로 전환
- `signal()` / `notify()`: 대기 중인 스레드를 깨움

### signal() 호출 방식

- **Hoare 방식**: signal() 호출 시 즉시 대기 스레드로 전환, 현재 스레드는 대기
- **Mesa 방식**: signal() 호출 후 현재 스레드가 계속 실행, 깨어난 스레드는 조건을 다시 확인 후 진행 (Java 채택 방식)

## 3.3 모니터 내부 흐름

1. **진입**: 프로세스가 모니터에 진입
2. **검사**: 필요 자원 여부 확인
3. **대기(wait)**: 자원이 없는 경우 → Condition Queue 진입, Entry Queue의 다른 프로세스 진입
4. **신호(signal)**: 다른 프로세스가 자원 처리 후 signal 호출
5. **재개**: Condition Queue에 있던 프로세스가 깨어나 작업 진행

# 4. Java Monitors

Java의 기본 동기화 단위는 **Thread**  
`monitor-lock` (intrinsic-lock) 사용

## 4.1 synchronized 키워드

임계 영역에 해당하는 코드 블록을 명시적으로 선언할 때 사용  
synchronized가 붙은 함수는 여러 스레드가 동시에 호출하더라도, 자바 가상 머신(JVM)이 모니터를 이용해 한 번에 하나의 스레드만 실행되도록 보장

- 모니터 락을 획득해야 진입 가능
- 메소드에 선언 시, 메소드 전체가 임계 영역으로 지정 (`this` 객체가 모니터 락 보유)
- 같은 인스턴스 변수를 넘겨준다면 서로 다른 Thread가 `this`를 공유

```java
// 특정 블록 잠그기
synchronized (object){
	// critical section
	}

// 메소드 전체 잠그기
public synchronized void 메소드명() {
	// critical section
}
```

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
		synchronized (object) {
			count++;
		}
	}
}
```

## 4.2 wait() / notify() / notifyAll()

`java.lang.Object` 클래스에 선언 → 모든 Java 객체가 보유

- `wait()`: 현재 스레드를 대기 상태로 전환
- `notify()`: 대기 중인 스레드 하나를 깨움 (JVM 스케줄러 정책에 따라 결정)
- `notifyAll()`: 대기 중인 스레드 전부를 깨움 → Ready Queue에서 경쟁

## 5. Liveness

Deadlock과 Starvation 문제를 해결하기 위한 개념  
상호 배제뿐만 아니라 시스템의 지속적인 진행을 보장

### Deadlock : 교착 상태

두 개 이상의 프로세스가 서로 상대방이 점유한 자원을 기다리며 무한히 대기하는 상태  
어떤 프로세스도 진행되지 못하고 영원히 블록됨

### Priority Inversion : 우선 순위 역전

높은 우선순위를 가진 프로세스가 낮은 우선순위의 프로세스에게 밀리는 현상

- 우선순위 상속 (Priority Inheritance)으로 해결 가능  
  낮은 우선순위 프로세스가 높은 우선순위 프로세스가 필요로 하는 자원을 점유 중일 때  
  일시적으로 높은 우선순위를 상속받아 빠르게 자원을 해제하도록 함