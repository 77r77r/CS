# 동시성 제어의 고전적 문제들

> 작성일 : 2026.02.18  
> 작성자 : 노미경
---

## 0. 대표적인 동기화 문제

1. Bounded-Buffer Problem : 생성자-소비자 문제를 유한 버퍼로 해결할 때의 문제
2. Readers-Writers Problem
3. Dining-Philosophers Problem : 철학자들의 저녁식사

> \*레이스 컨디션(Race Condition)?  
> 여러 스레드가 동시에 공유 자원에 접근할 때, 실행 순서에 따라 결과가 달라지는 문제

## 1. Bounded-Buffer Problem

Producer, Consumer 두 종류의 프로세스가 사용되어 Producer-Consumer(생산자 소비자) 문제라고도 불린다.  
제한된 크기의 공유 버퍼에 Producer(생산자)와 Consumer(소비자)가 동시에 접근할 때 발생하는 대표적인 동기화 문제로,  
Producer(생산자)는 버퍼를 가득 채우는 걸 목표로(full-buffer), Consumer(소비자)는 버퍼를 비우는 것을 목표로(empty-buffer)한다.

### 발생할 수 있는 동기화 이슈

1. **Buffer Overflow** : 생산자가 가득 찬 버퍼에 또 넣으려고 할 때 발생
2. **Buffer Underflow** : 소비자가 비어 있는 버퍼에서 꺼내려고 할 때 발생
3. **Race Condition** : 생산자와 소비자가 동시에 버퍼의 빈칸 위치를 수정할 때 데이터가 덮어씌워지거나 사라짐

### \* Mutex와 Semaphore

- **뮤텍스(Mutex)** : 공유 자원에 한 번에 하나의 스레드만 접근하도록 하는 동기화 객체  
  Lock 스레드만이 Unlock 할 수 있는 소유권이 핵심    
  상태는 Lock/Unlock(0 또는 1)만 존재
- **세마포어(Semaphore)** : 공유 자원에 동시에 접근할 수 있는 스레드 수를 제한하는 동기화 객체  
  값이 0이면 대기, 1 이상이면 자원을 사용하며 값을 1 감소  
  값이 0 또는 1만 갖는 **Binary Semaphore**와 값이 0 이상인 **Counting Semaphore**

> 단순히 하나의 공유 데이터를 보호할 때는 뮤텍스가 안전하고,  
> 특정 개수의 연결을 제한하거나 스레드 간 실행 순서를 맞출 때는 세마포어가 더 적합하다.

### Bounded-Buffer의 세마포어 구조

```text
int n;      // 총 버퍼의 개수

semaphore mutex = 1;  // 버퍼 접근 상호 배제용 (Binary Semaphore)
semaphore empty = n;  // 빈 슬롯 개수 - 생산자 대기용 (Counting Semaphore)
semaphore full  = 0;  // 채워진 슬롯 개수 - 소비자 대기용 (Counting Semaphore)
```

생산자와 소비자가 각각 여러 개 존재하는 구조를 가정한다.

#### 생산자 흐름

1. wait(empty)로 빈 슬롯이 생길 때까지 대기
2. wait(mutex)로 단독 진입 후 버퍼에 아이템을 추가
3. signal(mutex)로 뮤텍스를 반납
4. signal(full)로 소비자에게 신호를 전달

```text
while (true) {
    // 새 item 생성

    wait(empty);
    wait(mutex);

    // 버퍼에 새 item 추가

    signal(mutex);
    signal(full);
}
```

#### 소비자 흐름 (생산자와 대칭적으로 동작)

1. wait(full)로 채워진 슬롯이 생길 때까지 대기
2. wait(mutex)로 단독 진입 후 버퍼에서 아이템을 읽음
3. signal(mutex)로 뮤텍스를 반납
4. signal(empty)로 생산자에게 신호를 전달

```text
while (true) {
    wait(full);
    wait(mutex);

    // 버퍼에서 item 꺼냄

    signal(mutex);
    signal(empty);
}
```

이처럼 `wait(empty) ↔ signal(empty)`, `wait(full) ↔ signal(full)`의 대칭 구조를 반드시 맞춰야 하기 때문에, 순서가 조금이라도 꼬이면 데드락이나 데이터 오염이 발생할 수
있다.    
이러한 복잡성 때문에 실무에서는 잘 사용하지 않는다.

---

## Bounded-Buffer Solution

### PThread Solution

PThread(POSIX Thread)를 이용한 구현으로, 버퍼 1개 또는 n개, 생산자와 소비자가 1:1, 1:n, n:1, n:m 등 다양한 구조를 다룬다.  
동기화는 세마포어를 이용해 해결한다.

**생산자 흐름**  
sem_wait(&empty)로 빈 슬롯을 확인하고, pthread_mutex_lock으로 단독 진입 후 buffer[in]에 아이템을 쓰고 in = (in + 1) % n으로 인덱스를 이동한다. 이후 뮤텍스를 반납하고 sem_post(&full)로 소비자에게 신호를 보낸다.  
**소비자 흐름**  
대칭적으로 sem_wait(&full)로 채워진 슬롯을 확인하고, buffer[out]에서 아이템을 읽은 뒤 out = (out + 1) % n으로 인덱스를 이동하고 sem_post(&empty)로 생산자에게 신호를 보낸다.

```text
int buffer[n];      // 크기 n의 공유 버퍼
pthread_mutex_t mutex;          // 버퍼 접근 상호배제
sem_t empty;        // 빈 슬롯 개수 (초기값 n)
sem_t full;         // 채워진 슬롯 개수 (초기값 0)

int in = 0;         // 생산자가 다음에 쓸 버퍼 인덱스
int out = 0;        // 소비자가 다음에 읽을 버퍼 인덱스
```

## Java Solution

Java에서는 CashBox 같은 공유 객체 인스턴스(buffer)를 생산자와 소비자가 함께 사용한다.  
`synchronized` 블록 안에서 호출하면 Monitor Lock을 획득하여 동기화가 이루어진다.

`wait()`와 `notify()` 구조를 사용한다.

- **wait()**  
  현재 스레드를 일시 정지시키고 모니터 락을 반납한다.    
  다른 스레드가 notify() 또는 notifyAll()을 호출할 때까지 대기한다.
- **notify()**  
  InterruptException를 발생시킨다.  
  대기 중인 스레드 중 하나를 깨운다.
  > 어떤 스레드를 깨울지는 JVM 구현과 스케줄러 정책에 따라 달라지며, 특정 스레드를 지정할 수는 없다.  
  원하는 스레드만 깨우고 싶을 때는 `Condition`을 사용

```text
class CashBox {
    int[] buffer;   // 버퍼 크기
    int count;  // in, out 추적이 번거로우니 증감으로 확인
    int int, out;
    
    // 생산자
     synchronized void give() {
        while (count == buffer.length) {  // full
            try {
                wait();
            } catch (InterruptedException e) {}
        }

        // critical section

        notify();
    }
    
    // 소비자
    synchronized void take() {
        while (count == 0) {  // empty
            try {
                wait();
            } catch (InterruptedException e) {}
        }

        // critical section

        notify();
    }
}
```

---

## 2. Readers-Writers Problem

여러 프로세스가 하나의 공유 데이터(데이터베이스, 파일 등)에 접근할 때 발생하는 고전적인 동기화 문제.  
일반적인 동기화 문제와 달리, 이 문제는 프로세스를 역할에 따라 구분한다. **Reader는 읽기만 수행**하고, **Writer는 읽기와 쓰기를 모두 수행**한다고 가정한다.   
읽기 작업끼리는 데이터를 변경하지 않으므로 여러 Reader가 동시에 접근해도 문제가 없지만,  
여러 Writer가 동시에 쓰기 작업을 수행하거나, 쓰기 작업을 수행하는 도중 다른 프로세스가 읽어가면 데이터 불일치가 발생한다.
> 읽기 작업은 동기화가 불필요하므로 동시에 허용하면 성능을 향상시킬 수 있다.

### 두 가지 구현 방식(Variations)

우선순위를 어디에 두느냐에 따라 두 가지 방식으로 나뉜다.

**First R/W Problem (Readers-preference)**  
읽기 작업을 쓰기 작업보다 우선 할당한다.  
Reader가 많은 환경에서 Writer가 계속 대기하는 Writer Starvation 문제가 발생한다.

**Second R/W Problem (Writers-preference)**  
쓰기 작업을 읽기 작업보다 우선 할당한다.  
Writer가 많은 환경에서 Reader가 계속 대기하는 Reader Starvation 문제가 발생한다.

두 방식 모두 특정 프로세스가 무한히 대기하는 Starvation(기아) 문제를 완전히 해결하지는 못한다.

### Solution: First Readers-Writers Problem

```text
semaphore rw_mutex = 1; // Reader와 Writer가 공유 (Binary Semaphore)
semaphore mutex = 1;    // read_count 수정을 위한 상호배제 (Binary Semaphore)
int read_count = 0;     // 현재 읽기 중인 Reader 수
```

rw_mutex는 Reader와 Writer가 공유하며, Writer의 접근을 제어하는 역할을 한다.  
mutex는 read_count를 수정할 때 상호배제를 보장하기 위해 사용하고,  
read_count는 현재 읽기 중인 프로세스의 수를 추적한다.

#### Reader 흐름

Reader는 먼저 wait(mutex)로 read_count 수정 권한을 얻고 read_count를 1 증가시킨다. 이때 자신이 최초의 Reader(read_count == 1)라면 wait(rw_mutex)를
호출해 Writer의 접근을 차단한다. 이후 signal(mutex)로 권한을 반납하고 읽기 작업을 수행한다.  
읽기가 끝나면 다시 wait(mutex)로 권한을 얻어 read_count를 1 감소시키고, 마지막 Reader(read_count == 0)라면 signal(rw_mutex)를 호출해 Writer에게 접근을
허용한다.

```text
while (true) {
    wait(mutex);
    read_count++;
    if (read_count == 1) wait(rw_mutex);
    signal(mutex);
    
    /* 읽기 수행 */
    
    wait(mutex)
    read_count--;
    if (read_count == 0) signal(rw_mutex);
    signal(mutex);
}
```

> **\* 'read_count > 0' 이 아니라 'read_count == 1'인 이유**  
> Writer를 차단하는 역할은 첫 번째로 진입한 Reader가 대표로 수행한다. wait(rw_mutex)를 여러 번 호출하면 로직이 꼬이거나 데드락이 발생할 수 있으므로, 첫 진입 시 한 번만 호출한다.
> read_count == 0일 때 signal(rw_mutex)를 한 번만 호출하는 것도 같은 이유이다.

#### Writer 흐름

wait(rw_mutex)로 접근 권한을 요청하고 대기하다가, 권한을 얻으면 쓰기 작업을 수행한 뒤 signal(rw_mutex)로 권한을 반납한다.

```text
while (true) {
    wait(rw_mutex);
    
    /* 쓰기 수행 */
   
   signal(rw_mutex);
}
```

Writer 한 명이 임계구역에 진입한 순간, 나머지 프로세스들은 rw_mutex 대기 큐에서 대기한다.  
이 대기 큐에서 Reader에게 먼저 할당되면 First, Writer에게 먼저 할당되면 Second 구현이 된다.

현대 운영체제에서는 이 문제를 해결하기 위해 **Reader-Writer Lock API를 별도로 제공**하고 있다.

---

## Readers-Writers Solution

### Java Solution

sharedDB 클래스는 readerCount(현재 읽고 있는 Reader 수)와 isWriting(쓰기 진행 여부)으로 상태를 관리한다. Read와 Write의 락을 따로 구현하여 Reader끼리는 동시 접근을
허용하고, Writer는 단독 접근을 보장한다.

#### 사용 흐름

```text
sharedDB.acquireReadLock();
sharedDB.read();
sharedDB.releaseReadLock();

sharedDB.acquireWriteLock();
sharedDB.write();
sharedDB.releaseWriteLock();
```

#### Lock 구현

- **acquireReadLock**  
  Writer가 쓰기 중이면(isWriting == true) 대기하고, 쓰기가 끝나면 readerCount를 증가시켜 읽기 진입을 허용한다.
- **releaseReadLock**  
  readerCount를 감소시키고, 마지막 Reader(readerCount == 0)라면 notify()로 대기 중인 Writer에게 신호를 보낸다.
- **acquireWriteLock**  
  Reader가 한 명이라도 읽고 있거나(readerCount > 0) 다른 Writer가 쓰고 있으면(isWriting == true) 대기하고, 둘 다 아닐 때 isWriting = true로 설정하고
  진입한다.
- **releaseWriteLock**  
  isWriting = false로 초기화하고 notifyAll()을 호출해 대기 중인 모든 프로세스가 다시 경쟁하도록 한다.
  > \* notifyAll()을 사용하여 Starvation이 발생여부 차단

```text
class SharedDB {
    int readerCount = 0; 
    boolean isWriting = false;

    synchronized public void acquireReadLock() {
        while (isWriting) {
            try { wait(); } catch (InterruptedException e) {}
        }
        readerCount++;
    }

    synchronized public void releaseReadLock() {
        readerCount--;
        if (readerCount == 0)
            notify();
    }

    synchronized public void acquireWriteLock() {
        while (readerCount > 0 || isWriting) {
            try { wait(); } catch (InterruptedException e) {}
        }
        isWriting = true;
    }

    synchronized public void releaseWriteLock() {
        isWriting = false;
        notifyAll();
    }
}
```

---

## 3. Dining-Philosophers Problem : 철학자들의 저녁식사

다섯 명의 철학자들에게 다섯 개의 젓가락이 주어진다.
두 개의 젓가락이 있어야 밥을 먹을 수 있다.
동시에 젓가락을 집으려고 하면 어떻게 되는가? 를 해결하는문제

> 철학자 = 프로세스/스레드  
> 젓가락 = 공유자원  
> 식사 = 임계 구역 실행 (자원 사용 O)  
> 생각 = 나머지 구역 실행 (자원 사용 X)

각 철학자 사이에 놓인 젓가락에 동시에 접근하면 레이스 컨디션이 발생한다.

=> 해결 젓가락에 상호 배제를 적용하면 된다.

종류가 다른 여러개의 리소스를 다른 프로세스 사이에 넣어야하는 상황에서 상호 배제를 해야한다.  
deadlock과 기아문제 발생한다.

> \* DeadLock : 교착 상태
> 프로세스나 스레드가 서로 가진 자원을 기다리며 무한 대기에 빠져 시스템이 멈춰버리는 현상

## Dining-Philosophers Solution

### Semaphore Solution

각 젓가락에 세마포어를 할당하여 자원 사용 시 wait()을, 사용 완료 시 signal()을 호출해 상호배제를 구현한다. i번째 철학자는 i번째와 (i+1)번째 젓가락을 함께 관리한다.  
그러나 이 방식은 상호배제는 해결하지만 Deadlock이 발생할 수 있다. 모든 철학자가 동시에 왼쪽 젓가락을 집으면, 아무도 오른쪽 젓가락을 집지 못해 무한 대기 상태에 빠진다.

#### Deadlock 해결 방법

1. 철학자 수를 4명으로 제한하는 방법  
   젓가락이 5개이므로 반드시 하나가 남아, 누군가는 양쪽 젓가락을 모두 집을 수 있게 된다.
2. 양쪽 젓가락이 모두 사용 가능할 때만 집도록 하는 방법  
   하나라도 사용 중이면 아예 집지 않는다.
3. 비대칭으로 해결하는 방법  
   홀수 번호 철학자는 왼쪽 젓가락을 먼저 집고, 짝수 번호 철학자는 오른쪽 젓가락을 먼저 집는다. 이렇게 하면 같은 젓가락을 동시에 집는 상황이 방지되어 둘 중 하나는 반드시 식사를 할 수 있다.

세 가지 방법 모두 Deadlock은 해결할 수 있지만, Starvation 문제는 해결하지 못한다.  
완벽한 방지나 회피가 현실적으로 어렵기 때문에, 실제로는 데드락 발생 후
탐지 및 복구(Detection & Recovery) 기법을 사용하는 경우가 많다.

---

### Synchronization Tools Solution : Monitor Solution

양쪽 젓가락이 모두 사용 가능한 경우에만 집을 수 있도록 하는 방법이다. 철학자에게 세 가지 상태를 부여한다.

- thinking : 생각 중 (젓가락 불필요)
- hungry : 배고픈 상태 (젓가락을 집으려는 상태)
- eating : 식사 중 (양쪽 젓가락을 모두 사용 중)

hungry에서 eating으로 전환할 때, 양쪽 젓가락을 사용할 수 없다면 wait()으로 대기하고, 사용 가능하면 상태를 eating으로 전환 후 signal()/notify()로 진행한다. 자신의 왼쪽과 오른쪽
철학자가 모두 eating 상태가 아닐 때만 자신을 eating으로 전환할 수 있다.

Condition Variable 은 특정 조건이 충족될 때까지 스레드를 대기시키는 동기화 도구다. 세마포어의 wait()/signal()과 유사하지만, Monitor 내부에서 조건에 따라 선택적으로 대기/재개할 수
있어 더 세밀한 제어가 가능하다.

DiningPhilosophers 클래스는 pickup()과 putdown() 두 가지 인터페이스를 제공한다. 철학자는 식사 전 pickup()으로 젓가락을 집고, 식사 후 putdown()으로 젓가락을 내려놓는다.

상호배제와 데드락은 해결하지만, Starvation(기아) 문제는 여전히 해결하지 못한다.

#### Monitor Solution 원리

초기 상태는 모든 철학자가 THINKING으로 설정

- **pickup(i)**  
  상태를 HUNGRY로 바꾼 뒤 test()로 양쪽 젓가락 사용 가능 여부를 확인한다. test() 이후에도 EATING으로 전환되지 않았다면 누군가 젓가락을 선점하고 있는 것이므로 self[i].wait()으로
  대기한다.
- **putdown(i)**  
  상태를 THINKING으로 되돌리고, 자신의 왼쪽(i+4 % 5)과 오른쪽(i+1 % 5) 철학자에 대해 test()를 호출한다. 이때 대기 중이던 인접 철학자가 조건을 충족하면 signal()로 깨워준다.
- **test(i)**  
  왼쪽과 오른쪽 철학자가 모두 EATING 상태가 아니고, 본인이 HUNGRY인 경우에만 EATING으로 전환하고 signal()을 호출한다.

```text
monitor DiningPhilosophers {
    enum {THINKING, HUNGRY, EATING} state[5];  // 각 철학자의 상태
    condition self[5];  // 각 철학자의 조건 변수

    void pickup(int i) {
        state[i] = HUNGRY;
        test(i);                        // 양쪽 젓가락 사용 가능한지 확인
        if (state[i] != EATING)         // EATING으로 전환 실패 시 (누군가 선점 중)
            self[i].wait();             // 대기
    }

    void putdown(int i) {
        state[i] = THINKING;
        test((i + 4) % 5);             // 왼쪽 철학자 확인 및 깨우기
        test((i + 1) % 5);             // 오른쪽 철학자 확인 및 깨우기
    }

    void test(int i) {
        if ((state[(i + 4) % 5] != EATING)   // 왼쪽이 식사 중이 아니고
         && (state[i] == HUNGRY)              // 본인이 배고프고
         && (state[(i + 1) % 5] != EATING)) { // 오른쪽이 식사 중이 아닐 때
            state[i] = EATING;
            self[i].signal();                  // 대기 중이던 철학자 깨우기
        }
    }
}
```

### Monitor Solution - PThread

### Monitor Solution - Java

아래 라이브러리가 제공됨

- `java.util.concurrent.locks.Condition`
- `java.util.concurrent.locks.Lock`
- `java.util.concurrent.locks.ReentrantLock` : 재진입 ()

Thread의 monitor은 wait()와 notify()를, Condition variable에서는 await()와 signal()을 사용한다.

재진입?  
cpu에서 문맥 교환이 일어나서 레디 큐/웨이트 큐로 갔다가 다시 cpu로 들어오는 것

#### 사용 흐름

```text
think();
monitor.pickup(id);
eat();
monitor.putdown(id);
```

```java
class DiningPhilosopherMonitor {
	private int numOfPhils; // 철학자 수
	private State[] state;  // 상태
	private Condition[] self;   // 조건 변수
	private Lock lock;

	public DiningPhilosopherMonitor(int num) {
		numOfPhils = num;
		state = new State[num];
		self = new Condition[num];
		lock = new ReentrantLock();
		for (int i = 0; i < num; i++) {
			state[i] = State.THINKING;
			self[i] = lock.newCondition();
		}
	}
}
```

---

## Thread-Safe Concurrent Applications

여러 스레드가 동시에 실행되더라도 데이터가 깨지지 않고 항상 올바르게 동작하도록 설계된 애플리케이션

#### Imperative Programming(명령형 프로그래밍)

C, Java, Python 같은 전통적인 프로그래밍 방식으로, '어떻게(How)'를 단계별로 명령하는 방식이다. 명령문을 순서대로 나열하며 메모리의 상태(State)를 계속해서 변경한다. 이 명령문들이 여러
스레드에서 동시에 실행될 때 겹치면서 Race Condition이 발생해 동시성 문제가 생긴다.

### 설계 방식

1. **Transaction Memory**  
   메모리 접근 자체를 원자적(Atomic)으로 만드는 방법이다.  
   데이터베이스의 트랜잭션 개념을 메모리에 적용한 것으로, 특정 코드 블록을 트랜잭션으로 선언하면 해당 블록이 전부 성공하거나 전부 실패(롤백)하도록 보장한다.  
   락 없이도 안전한 동시성을 구현할 수 있다는 장점이 있다.
2. **OpenMP**  
   컴파일러에게 #pragma 지시어를 주어 병렬화를 돕는 라이브러리다.  
   `#pragma omp parallel for`를 사용하면 컴파일러가 루프를 자동으로 여러 스레드로 분할해 병렬 실행한다.  
   개발자가 직접 스레드를 생성하거나 동기화 코드를 작성하지 않아도 된다는 점에서 편의성이 높다.
3. **Functional Programming Language**  
   명령형 프로그래밍과 달리 '무엇을(What)'계산할지를 표현하는 방식이다.  
   함수형 프로그래밍의 핵심은 불변성(Immutability)으로, 데이터를 변경하지 않고 새로운 값을 반환한다.  
   공유 상태 자체가 없으므로 Race Condition이 근본적으로 발생하지 않는다.
   하드웨어 성능이 비약적으로 향상된 현재는 불변성으로 인한 성능 부담이 크게 줄어, 동시성 문제를 근본적으로 해결하는 방법으로 주목받고 있다.
