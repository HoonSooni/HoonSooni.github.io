---
title: "LeetCode Easy - 1909. Remove One Element to Make the Array Strictly Increasing"
date: 2025-02-03 00:00:00 +0800
categories: [IT, 코딩테스트]
tags: [coding test, algorithm, leetcode] 
description: 릿코드 Easy Remove One Element to Make the Array Stricktly Increasing 문제 풀이.
---
[https://leetcode.com/problems/remove-one-element-to-make-the-array-strictly-increasing/description/](https://leetcode.com/problems/remove-one-element-to-make-the-array-strictly-increasing/description/)

## 문제 설명 (난이도 Easy)

Given a **0-indexed** integer array nums, return true if it can be made ***strictly increasing*** after removing ***exactly one*** element, or false otherwise. If the array is already strictly increasing, return true.

The array nums is **strictly increasing** if `nums[i - 1] < nums[i]` for each index `(1 <= i < nums.length)`.

 

**Example 1:**

> **Input:** nums = [1,2,10,5,7]
**Output:** true
**Explanation:** By removing 10 at index 2 from nums, it becomes [1,2,5,7].
[1,2,5,7] is strictly increasing, so return true.

**Example 2:**

> **Input:** nums = [2,3,1,2]
**Output:** false
**Explanation:**
[3,1,2] is the result of removing the element at index 0.
[2,1,2] is the result of removing the element at index 1.
[2,3,2] is the result of removing the element at index 2.
[2,3,1] is the result of removing the element at index 3.
No resulting array is strictly increasing, so return false.

**Example 3:**

> **Input:** nums = [1,1,1]
**Output:** false
**Explanation:** The result of removing any element is [1,1].
[1,1] is not strictly increasing, so return false.

 

**Constraints:**

* `2 <= nums.length <= 1000`
* `1 <= nums[i] <= 1000`

<hr />

## 문제풀이
### 풀이 1. 오름차순의 문제가 있는 원소 직접 지우는 풀이
```cpp
class Solution {
public:
    bool canBeIncreasing(vector<int>& nums) {
        bool isFound = false;

        for (int i = nums.size() - 1; i > 0; --i) {
            if (nums[i - 1] - nums[i] >= 0) {
                if (isFound == true) { return false; }
                
                auto erasePos = nums.begin();
                if ((i == (nums.size() - 1)) || (nums[i + 1] > nums[i - 1])) { erasePos += i; }
                else { erasePos += i - 1;}
                
                nums.erase(erasePos);
                isFound = true;
            }
        }
        
        for (int i = 0; i < nums.size() - 1; ++i) {
            if (nums[i] > nums[i + 1]) { return false; }
        }

        return true;
    }
};
```
문제에서 원하는건 해당 문자열에서 단 한 숫자만 지웠을 때 오름차순으로 정렬된 수열을 가지느냐를 물어보고 있다.
그렇기 때문에 처음에 생각해낸 방법은 배열을 순서대로 돌아다니면서 오름차순을 망가뜨리는 원소를 그 자리에서 바로바로 지워주는 것이다.

배열을 앞에서부터 시작해도 문제는 없을 것 같은데 개인적으로 뒤에서 시작하는 것이 직관적일 것 같아 그렇게 구현하였다.


```cpp
for (int i = nums.size() - 1; i > 0; --i) {
    if (nums[i - 1] - nums[i] >= 0) {
        if (isFound == true) { return false; }
        
        auto erasePos = nums.begin();
        if ((i == (nums.size() - 1)) || (nums[i + 1] > nums[i - 1])) { erasePos += i; }
        else { erasePos += i - 1;}
        
        nums.erase(erasePos);
        isFound = true;
    }
}
```
for문 안에서 우선 `nums[i - 1]`가 `nums[i]`보다 더 큰지 뺄셈을 이용하여 판단하고 있다. 더 클때 뿐만 아니라 동일할 경우에도 걸러내야 하기 때문에 두 원소의 차이가 0보다 크거나 같은가를 확인한다.

이미 이 전에 오름차순을 방해하는 원소가 있었는지를 `isFound`로 판단하고 없었다면 해당 원소를 배열에서 제거한다.

```cpp
auto erasePos = nums.begin();
if ((i == (nums.size() - 1)) || (nums[i + 1] > nums[i - 1])) { erasePos += i; }
else { erasePos += i - 1;}
```
주의 깊게 살펴야할 부분은 지워야하는 정확한 인덱스를 정하는 부분이다.
특정 조건을 만족하면 i번째를 지우거나, 만족하지 않는다면 `i - 1`번째를 지운다.

1. `i == (nums.size() - 1)`
예시 배열: `[1, 2, 3, 4, 1]` <br> i가 맨 마지막 원소인 '1'을 가리킬 때 (for문이 시작될 때) 1은 4보다 당연히 작기 때문에 지워줘야한다. 이때 `erasePos`를 i로 설정하여 5번째 원소(i == 4)를 지워준다.

2. `nums[i + 1] > nums[i - 1]`
예시 배열: `[2, 3, 1, 4]` <br /> i가 세 번째 원소인 '1'을 가리킬 때(i가 2일 때) 1이 3보다 작기 때문에 지운다. <br />
그런데 이때 `nums[i + 1] > nums[i - 1]` 조건을 체크하는 이유는 만일 배열이 `[2, 3, 1, 2]`처럼 세 번째 원소 '1'을 지워도 `[2, 3, 2]`가 되면서 조건을 만족하지 못하는 경우를 판별하기 위함이다.

이러한 두 조건 중 하나라도 만족하지 않는다면 `erasePos`를 `i - 1`로 초기화 해준다. <br />
예시 배열이 `[1, 2, 10, 5, 7]`이고 i가 3일 때 '5'를 가리킨다. `nums[i - 1]`보다 5가 더 작기 때문에 지워줘야 하는데 `erasePos`를 i로 두고 지우게 되면 '10'이 아니라 '5'를 지우기 때문에 조건을 만족하는 배열임에도 불구하고 `[1, 2, 10, 7]`이 돼 잘못된 결과가 발생한다.
그러한 이유로 `erasePos = i - 1`로 초기화한다.

```cpp
for (int i = 0; i < nums.size() - 1; ++i) {
    if (nums[i] > nums[i + 1]) { return false; }
}   
```

마지막으로 다시 한 번 배열 원소들을 하나씩 체크하며 오름차순이 맞는지 확인한다.
아까 예로 들었던 `[2, 3, 1, 2]`같은 배열의 경우 위 반복문이 끝나면 `[2, 3, 2]`로 끝나게 된다. 이러한 경우를 분별하기 위해서 다시 한 번 체크해주는 것이다.

이 풀이 방식은 for문 안에서 `erase()` 함수를 사용하고 있고 이 함수도 내부적으로는 O(n)의 시간 복잡도를 가지기에 최종 시간 복잡도는 O(n^2).

> 시간 복잡도: O(N^2), 공간 복잡도: O(1)

### 풀이 2. 직접 지우지 않고 지웠다고 가정하는 풀이
```cpp
class Solution {
public:
    bool canBeIncreasing(vector<int>& nums) {
        int count = 0;
        
        for (int i = 0; i < nums.size() -1; ++i) {
            if (nums[i] >= nums[i + 1]) {
                count++;
                
                if (i > 0 && nums[i - 1] >= nums[i + 1]) {
                    nums[i + 1] = nums[i];
                }
            }
        }
        
        return count < 2;
    }
};
```
이번에는 선형 시간 복잡도를 가지는 풀이이다. <br /> i가 0일 때부터 반복을 돌면서 `nums[i]`가 `nums[i + 1]`보다 클 때 `count`를 하나씩 늘려가면서 최종적으로 `count`가 1을 넘어가는지 반환한다.

```cpp
if (i > 0 && nums[i - 1] >= nums[i + 1]) {
    nums[i + 1] = nums[i];
}
```

중간에 특이한 조건문이 하나 있다. <br />
배열이 `[2, 3, 1, 2]`처럼 중간에 '1'을 지워도 실제로는 만족하지 않지만 만족하는 것 처럼 보이는 배열을 위해서이다.
반복문을 돌면 컴퓨터 입장에서는 2, 3은 올바르다, 3, 1은 오름차순이 아니다. 여기서 `count`를 1 증가시키지만 그 다음엔 1, 2를 살펴보게 되니 또 오름차순이니까 숫자 단 하나만 오름차순에 어긋나는 줄 착각하게 되는 것이다.

그래서 `nums[i + 1]`를 `nums[i]`로 초기화해줘서 `[2, 3, 3, 2]`가 되게끔 하여 잘못된 체크를 방지한다. 결과적으로 보면 1을 지웠으면 `[2, 3, 2]`가 됐을 배열을 `[2, 3, 3, 2]`로 만들어 실제로 ***지우는*** 대신에 ***지우는 척***을 하는 셈이다.

> 시간 복잡도: O(N), 공간 복잡도: O(1)