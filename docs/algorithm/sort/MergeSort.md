# Merge Sort : 병합 정렬

> 작성일 : 2026.02.04  
> 작성자 : 노미경

---

## 개념

배열을 반으로 계속 나눈 뒤, 정렬된 상태로 병합하는 분할 정복(Divide & Conquer) 기반 알고리즘

> **분할 정복이란?**  
> 문제를 더 이상 나눌 수 없을 때까지 분해한 뒤, 부분 해를 결합해 전체 해를 얻는 기법

--- 

## 성능 및 특징

- 시간 복잡도: O(n log n) - 모든 경우에 보장
- 공간 복잡도: O(n) - 병합 과정에서 추가 메모리 필요
- 안정 정렬: 동일한 값의 상대적 순서 유지

---

## 알고리즘

#### 하향식 2-way

리스트의 길이가 1 이하면 이미 정렬된 것으로 간주, 그렇지 않으면:

1. **분할(Divide)**: 배열을 절반으로 나눔
2. **정복(Conquer)**: 각 부분을 재귀적으로 정렬
3. **병합(Combine)**: 정렬된 두 부분을 하나로 합침
4. **복사(Copy)**: 임시 배열의 결과를 원본에 반영

---

## 장단점

### 장점

- 항상 O(n log n) 성능 보장
- 안정 정렬로 순서 유지 필요 시 유용
- 대용량 데이터 처리에 적합

### 단점

- 추가 메모리 공간 필요 (O(n))
- 제자리 정렬(in-place) 아님
- 작은 데이터셋에는 비효율적

---

## 활용 분야

- 대용량 데이터 정렬
- 외부 정렬 (메모리에 다 올릴 수 없는 데이터)
- 표준 라이브러리 정렬 알고리즘의 기반
- 안정성이 중요한 정렬 작업

---

## 구현 코드

### System.arraycopy 메서드

```java
System.arraycopy(Object src, int srcPos, Object dest, int destPos, int length)
```

- **src**: 원본 배열
- **srcPos**: 원본 시작 인덱스
- **dest**: 대상 배열
- **destPos**: 대상 시작 인덱스
- **length**: 복사할 요소 개수

### 코드

```java
public class MergeSort {
	public static void main(String[] args) {
		int[] arrs = new int[] {21, 10, 12, 20, 25, 13, 15, 22};
		int size = arrs.length;

		mergeSort(arrs, 0, size - 1);

		System.out.println(Arrays.toString(arrs));
	}

	public static void mergeSort(int[] arr, int start, int end) {
		if (start >= end) {
			return;
		}

		int mid = (start + end) / 2;
		mergeSort(arr, start, mid);
		mergeSort(arr, mid + 1, end);
		merge(arr, start, mid, end);
	}

	static void merge(int[] arr, int start, int mid, int end) {
		// 길이 만큼 배열 생성
		int[] temp = new int[end - start + 1];
		int i = start, j = mid + 1, k = 0;

		while (i <= mid && j <= end) {
			temp[k++] = arr[i] <= arr[j] ? arr[i++] : arr[j++];
		}

		while (i <= mid) {
			temp[k++] = arr[i++];
		}

		while (j <= end) {
			temp[k++] = arr[j++];
		}

		System.arraycopy(temp, 0, arr, start, temp.length);
	}
}
```

---

#### 참고 자료

- [위키백과 - 합병 정렬](https://ko.wikipedia.org/wiki/%ED%95%A9%EB%B3%91_%EC%A0%95%EB%A0%AC)
- [O(n log n) 정렬 알고리즘 정리](https://velog.io/@eunchae2000/%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0-Onlogn-%EC%A0%95%EB%A0%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-with-Python)
