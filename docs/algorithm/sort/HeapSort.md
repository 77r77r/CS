# Heap Sort : 힙 정렬

> 작성일 : 2026.02.05  
> 작성자 : 노미경

---

## 개념

**힙(완전 이진 트리)** 구조를 이용해 최댓값(또는 최솟값)을 반복적으로 추출하는 **in-place 정렬** 알고리즘

> **완전 이진 트리(Complete Binary Tree)란?**  
> 마지막 레벨을 제외한 모든 레벨이 꽉 차 있고, 마지막 레벨은 왼쪽부터 채워지는 이진 트리

---

## 성능 및 특징

- **시간 복잡도**: O(n log n) - 항상 보장
- **공간 복잡도**: O(1) - 추가 메모리 거의 없음 (in-place)
- **In-place 정렬**: 입력 배열 자체를 변경하며 정렬
- **불안정 정렬**: 동일한 값의 상대적 순서 유지 안 됨
- **최댓값/최솟값 추출 용이**: 우선순위 큐에 적합한 자료구조

---

## 알고리즘

### 전체 과정

1. **힙 구성**: n개의 노드로 완전 이진 트리 구성
2. **최대 힙 변환**: 부모 노드 > 자식 노드 조건을 만족하도록 재배치 (하향식)
3. **루트-끝 교환**: 가장 큰 값(루트)을 배열 끝과 교환
4. **힙 크기 감소 및 복구**: 힙 크기를 1 줄이고 heapify로 힙 성질 복구
5. **반복**: 힙이 빌 때까지 3-4 반복

> **힙(Heap)이란?**
> - **최대 힙**: 부모 노드 ≥ 자식 노드 조건을 만족하는 완전 이진 트리
> - **최소 힙**: 부모 노드 ≤ 자식 노드 조건을 만족하는 완전 이진 트리

### 상세 절차

1. 배열을 **최대 힙**으로 구성
2. 루트(최댓 값)와 마지막 원소 교환
3. 힙 크기 **1 감소**
4. 힙 성질 복구 (**heapify**)
5. 힙이 빌 때까지 반복

---

## 장단점

### 장점

- 항상 O(n log n) 성능 보장
- 추가 메모리 거의 불필요 (in-place)
- Quick Sort의 최악 케이스 회피 가능
- 이론적으로 안정적인 성능

### 단점

- 불안정 정렬
- 평균적으로 Quick Sort보다 느림
- 캐시 효율성이 낮음

---

## 활용 분야

- 최악 시간 복잡도 보장이 중요한 경우
- 메모리 사용이 제한적인 환경
- Quick Sort의 최악 케이스 회피 대안
- 우선순위 큐 구현

---

## 구현 코드

```java
static void heapSort(int[] arr) {
	int size = arr.length;

	// 최대 힙 구성
	for (int i = size / 2 - 1; i >= 0; i--) {
		heapify(arr, size, i);
	}

	// 하나씩 정렬
	for (int i = size - 1; i > 0; i--) {
		// 최댓값 끝으로 이동
		int temp = arr[i];
		arr[i] = arr[0];
		arr[0] = temp;

		// 힙 재정렬
		heapify(arr, i, 0);
	}
}

static void heapify(int[] arr, int size, int root) {
	int largest = root;
	int left = 2 * root + 1;
	int right = 2 * root + 2;

	if (left < size && arr[left] > arr[largest]) {
		largest = left;
	}

	if (right < size && arr[right] > arr[largest]) {
		largest = right;
	}

	if (largest != root) {
		int temp = arr[largest];
		arr[largest] = arr[root];
		arr[root] = temp;

		heapify(arr, size, largest);
	}
}
```

---

#### 참고 자료

- [위키백과 - 힙 정렬](https://ko.wikipedia.org/wiki/%ED%9E%99_%EC%A0%95%EB%A0%AC)
- [O(n log n) 정렬 알고리즘 정리](https://velog.io/@eunchae2000/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-Onlogn-%EC%A0%95%EB%A0%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-with-Python)