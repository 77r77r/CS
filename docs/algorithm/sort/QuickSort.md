# Quick Sort : 교환 정렬

> 작성일 : 2026.02.05  
> 작성자 : 노미경

---

## 개념

기준값(Pivot)을 중심으로 데이터를 분할하며 정렬하는 **분할 정복(Divide & Conquer)** 기반 알고리즘

> **분할 정복이란?**  
> 문제를 더 이상 나눌 수 없을 때까지 분해한 뒤, 부분 해를 결합해 전체 해를 얻는 기법

---

## 성능 및 특징

- **시간 복잡도**
    - 평균: O(n log n)
    - 최악: O(n²)
- **공간 복잡도**: O(log n) - 재귀 호출 스택
- **In-place 정렬**: 추가 메모리 사용 최소화
- **불안정 정렬**: 동일한 값의 상대적 순서 유지 안 됨
- **실제로 가장 빠른 정렬**: 다른 O(n log n) 알고리즘보다 평균적으로 빠름

> **In-place 정렬이란?**  
> 입력 배열 자체를 변경하며 정렬하는 방식으로, 병합 과정 없이 추가 메모리 사용이 적음

---

## 알고리즘

1. **Pivot 선택**: 하나의 기준값을 선택
2. **분할(Partition)**: Pivot보다 작은 값은 왼쪽, 큰 값은 오른쪽으로 이동
3. **재귀 정렬**: 분할된 두 배열에 대해 재귀적으로 반복 (크기가 0 또는 1이 될 때까지)

---

> **Pivot 선택 최적화**  
> 많은 라이브러리에서는 세 값(좌측 끝, 중앙, 우측 끝)의 중위값을 사용하여 중앙 분할 가능성을 높임

### 파티션(Partition) 상세 과정

#### 방법 1: 단일 지시자 방식

두 개의 지시자를 사용:

- **i (빨간색)**: 현재 확인 중인 원소
- **sortedIndex (검은색)**: 피벗보다 작은 원소가 올 위치

**동작 규칙:**

1. 현재 원소(i)가 **피벗보다 작으면**:

- i와 sortedIndex 위치의 원소를 **교환(swap)**
- sortedIndex를 **한 칸 전진**

2. i를 **한 칸 전진**
3. 모든 원소를 확인할 때까지 반복

#### 방법 2: 양방향 탐색 방식

두 개의 인덱스를 양쪽에서 이동:

- **i**: 왼쪽에서 시작, Pivot보다 **큰 값** 탐색
- **j**: 오른쪽에서 시작, Pivot보다 **작은 값** 탐색

**동작 규칙:**

1. **i 이동**: Pivot보다 큰 값이 나타날 때까지 i 증가
2. **j 이동**: Pivot보다 작은 값이 나타날 때까지 j 감소
3. **i ≤ j 이면**: i와 j 위치의 값을 교환 후 i 증가, j 감소
4. **i > j 이면**: 파티션 완료
5. **재귀 호출**:

- 왼쪽: `left < j`이면 `quickSort(arr, left, j)` 재귀
- 오른쪽: `i < right`이면 `quickSort(arr, i, right)` 재귀

---

## 장단점

### 장점

- 평균적으로 가장 빠른 정렬 속도
- 추가 메모리 사용 최소화 (In-place)
- 캐시 효율성이 높음

### 단점

- 최악의 경우 O(n²) 성능
- 불안정 정렬
- 이미 정렬된 데이터에서 성능 저하 가능

---

## 활용 분야

- 프로그래밍 언어의 표준 정렬 라이브러리
- 대규모 데이터 정렬
- 메모리 제약이 있는 환경
- 데이터베이스 및 인덱싱 시스템

---

## 구현 코드

```java
public static void quictSort(int[] arr, int left, int right) {
	if (left >= right) {
		return;
	}

	int pivot = arr[right];
	int sortedIdx = left;

	// i = 현재 탐색 위치
	// sortedIdx = 교환 위치
	for (int i = left; i < right; i++) {
		if (arr[i] <= pivot) {
			swap(arr, i, sortedIdx);
			sortedIdx++;
		}
	}

	// pivot 정렬
	swap(arr, sortedIdx, right);

	quictSort(arr, left, sortedIdx - 1);
	quictSort(arr, sortedIdx + 1, right);
}

private static void swap(int[] arr, int i, int sortedIdx) {
	int temp = arr[i];
	arr[i] = arr[sortedIdx];
	arr[sortedIdx] = temp;
}
```

---

## 실제 사용에서의 개선

실제 데이터에서 효율성을 위한 수정 사항:

1. **하이브리드 알고리즘**: 작은 데이터에서는 삽입 정렬로 전환 (오버헤드 감소)
2. **정렬된 데이터 처리**: 이미 정렬된 데이터에서 O(n) 성능 유지
3. **안정성 보완**: Timsort(병합 정렬 기반), Intro sort(퀵 정렬 + 힙 정렬) 같은 정교한 알고리즘 사용

---

#### 참고 자료

- [위키백과 - 퀵 정렬](https://ko.wikipedia.org/wiki/%ED%80%B5_%EC%A0%95%EB%A0%AC)
- [Wikipedia - Sorting Algorithm](https://en.wikipedia.org/wiki/Sorting_algorithm)
- [모두의 코드 - 퀵 정렬](https://modoocode.com/248)
