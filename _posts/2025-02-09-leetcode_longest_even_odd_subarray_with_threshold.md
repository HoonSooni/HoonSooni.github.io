---
title: "LeetCode Easy - 2760. Longest Even Odd Subarray With Threshold"
date: 2025-02-09 00:00:00 +0800
categories: [IT, 코딩테스트]
tags: [coding test, algorithm, leetcode] 
description: 릿코드 Easy Longest Even Odd Subarray With Threshold 문제 풀이.
---

[https://leetcode.com/problems/longest-even-odd-subarray-with-threshold/description/](https://leetcode.com/problems/longest-even-odd-subarray-with-threshold/description/)

## 문제 설명 (난이도 Easy)

You are given a **0-indexed** integer array `nums` and an integer `threshold`.

Find the length of the **longest subarray** of `nums` starting at index `l` and ending at index `r` `(0 <= l <= r < nums.length)` that satisfies the following conditions:

- `nums[l] % 2 == 0`
- For all indices `i` in the range `[l, r - 1]`, `nums[i] % 2 != nums[i + 1] % 2`
- For all indices `i` in the range `[l, r]`, `nums[i] <= threshold`

Return _an integer denoting the length of the longest such subarray._

**Note:** A **subarray** is a contiguous non-empty sequence of elements within an array.

**Example 1:**

**Input:** nums = [3,2,5,4], threshold = 5<br />
**Output:** 3<br />
**Explanation:** In this example, we can select the subarray that starts at l = 1 and ends at r = 3 => [2,5,4]. This subarray satisfies the conditions.
Hence, the answer is the length of the subarray, 3. We can show that 3 is the maximum possible achievable length.

**Example 2:**

**Input:** nums = [1,2], threshold = 2<br />
**Output:** 1<br />
**Explanation:** In this example, we can select the subarray that starts at l = 1 and ends at r = 1 => [2]. 
It satisfies all the conditions and we can show that 1 is the maximum possible achievable length.

**Example 3:**

**Input:** nums = [2,3,4,5], threshold = 4<br />
**Output:** 3<br />
**Explanation:** In this example, we can select the subarray that starts at l = 0 and ends at r = 2 => [2,3,4]. 
It satisfies all the conditions.
Hence, the answer is the length of the subarray, 3. We can show that 3 is the maximum possible achievable length.

**Constraints:**

- `1 <= nums.length <= 100`
- `1 <= nums[i] <= 100`
- `1 <= threshold <= 100`

---

## 문제  풀이 
```cpp
class Solution {
public:
    int longestAlternatingSubarray(vector<int>& nums, int threshold) {
        int maxLength = 0;

        for (int i = 0; i < nums.size(); ++i) {
            if (nums[i] % 2 == 0) {
                const int l = i;
                int r = l;
                
                for (; r < nums.size() - 1; ++r) {
                    if (nums[r] % 2 == nums[r + 1] % 2 || nums[r] > threshold) {
                        break;
                    }
                }
                int surplus = nums[r] <= threshold ? 1 : 0;
                maxLength = max(maxLength, r - l + surplus);
                
                i = r;
            }
        }
        
        return maxLength;
    }
};
```
해당 문제는 주어진 배열에서 특정 조건을 만족하는 가장 큰 부분 배열의 길이를 찾는 문제이다.

조건은 다음과 같다:
`l`번째 원소부터 `r`까지의 원소까지, `0 <= l, r < nums의 길이`, 그리고 각 원소의 위치를 `i`로 나타낼 때:
1. `nums[l]`가 짝수일 때.
2. `nums[i]`와 `nums[i + 1]`가 서로 다른 수의 성질(짝수홀수)을 가질 때.
3. `nums[i]`가 `threshold`보다 작거나 같을 때.

즉, 매 부분 배열은 항상 짝수로 시작해야하고 그 다음 원소부터 홀 짝 홀 짝 순서를 가져야 하면서 동시에 `threshold`보다 더 작거나 같아야 한다.

위 코드가 이 모든 조건을 만족하는 코드이다.

```cpp
if (nums[i] % 2 == 0) {
	const int l = i;
	int r = l;
	
	for (; r < nums.size() - 1; ++r) {
		if (nums[r] % 2 == nums[r + 1] % 2 || nums[r] > threshold) {
			break;
		}
	}
	int surplus = nums[r] <= threshold ? 1 : 0;
	maxLength = max(maxLength, r - l + surplus);
	
	i = r;
}
```
다른건 별로 중요하지 않고 이 부분만 살펴보자.<br />
일단 가장 바깥에 있는 if문으로 부분 배열의 시작이 짝수인지 판별한다.<br />
그리고 `l(left)`을 현재 원소의 위치로 초기화하고 `r(right)`을 점점 늘려가면서 진행하는 for문을 하나 구현했다.

짝 홀 짝 홀이 번갈아가면서 진행되는지 그리고 `threshold`보다 작거나 같은지 판별하고 둘 중 하나라도 어긋나면 바로 `break`로 끝내서 해당 부분 배열의 끝을 알린다.

그리고 마지막으로 맨 처음 초기화했던 `maxLength`에 방금 구한 부분 배열의 길이를 넣어준다.
항상 최대값이 들어가야 하기 때문에 `max()` 함수를 사용했다.<br />
`surplus` 변수로 1을 더할지 말지 정해주는 이유는 for문이 `nums`의 맨 마지막 원소가 `threshold`보다 작거나 같은지 판단하지 않기 때문에 for문 바깥에서 한 번 더 체크해주고 참이라면 1을 부분 배열 길이에 더해주는 용도이다.

해당 코드는 최악의 경우 이중 반복문을 모두 시행하기 때문에 시간 복잡도는 O(n^2)이다. 반면에 주어진 배열 이외에는 달리 데이터 양에 따라 크기가 증가하는 변수는 없기 때문에 상수 복잡도를 가진다.

> 시간 복잡도: O(N^2), 공간 복잡도: O(1)