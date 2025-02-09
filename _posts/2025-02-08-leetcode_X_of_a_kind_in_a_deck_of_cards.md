---
title: "LeetCode Easy - 914. X of a Kind in a Deck of Cards"
date: 2025-02-08 00:00:00 +0800
categories: [CodingTest, LeetCode]
tags: [coding test, algorithm, leetcode] 
description: 릿코드 Easy X of a Kind in a Deck of Cards 문제 풀이.
---

[https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/description/](https://leetcode.com/problems/x-of-a-kind-in-a-deck-of-cards/description/)

## 문제 설명

You are given an integer array `deck` where `deck[i]` represents the number written on the `ith` card.

Partition the cards into **one or more groups** such that:

- Each group has **exactly** `x` cards where `x > 1`, and
- All the cards in one group have the same integer written on them.

Return `true` _if such partition is possible, or_ `false` _otherwise_.

**Example 1:**

**Input:** deck = [1,2,3,4,4,3,2,1] <br />
**Output:** true <br />
**Explanation**: Possible partition [1,1],[2,2],[3,3],[4,4]. 

**Example 2:** 

**Input:** deck = [1,1,1,2,2,2,3,3] <br />
**Output:** false <br />
**Explanation**: No possible partition.

**Constraints:**

- `1 <= deck.length <= 10^4`
- `0 <= deck[i] < 10^4`
---

## 문제 풀이
### 풀이 1. 틀린 코드

정수로 이루어진 배열 하나가 주어지고 해당 배열을 특정 조건에 따라 그룹 할 수 있냐 없냐 판별하는 문제이다. 
특정 조건은 아래와 같다.
1. 모든 그룹은 반드시 길이가 같아야한다.
2. 모든 그룹 중 단 하나라도 길이가 1보다 같거나 작으면 안된다.

```cpp
class Solution {
public:
    bool hasGroupsSizeX(vector<int>& deck) {
        unordered_map<int, int> occurrences;
        
        for (const int card : deck) {
            occurrences[card]++;
        }
        
        bool isAllSame = true;
        bool isBiggerThanOne = true;
        
        auto firstElement = occurrences.begin();
        const int baseOccurrenceCount = firstElement->second;
        int minCount = 1000000; // 10^5
        
        // getting the minimum occurrence
        for (const pair<int, int> card : occurrences) {
            minCount = min(minCount, card.second);
        }
        
        // checking the conditions
        for (const pair<int, int> card : occurrences) {
            if (card.second != baseOccurrenceCount &&
                card.second % minCount != 0) {
                isAllSame = false;
                break;
            }
        }
        if (minCount < 2) { isBiggerThanOne = false; }
        
        return isAllSame && isBiggerThanOne;
    }
};
```



이 조건을 기준으로 코드를 작성해보았다.
1. `decks`를 반복해서 돌면서 각 숫자의 등장 횟수를 `map`에 저장한다.
2. `map`을 반복해서 돌아주면서 조건에 부합하는지 확인한다.
	1. `isAllSame`, `isBiggerThanOne`을 통해 모든 그룹의 길이가 같은지 확인하고 그룹의 길이가 1보다 큰지 확인한다.
	2. `firstElement.second`를 통해 `baseOccurrenceCount`를 사용하는 이유는 `map`중 아무 `occurrence`를 잡아서 그것과 다른 것들을 비교하며 하나라도 다르면 `isAllSame`을 `false`로 초기화하기 위함이다.
3. 만일 `isAllSame`과 `isBiggerThanOne` 모두 다 참이라면 참을 반환하고 그렇지 않으면 거짓을 반환한다.

#### 이 풀이의 문제점
**그룹의 숫자 판별 오류** <br />
이 코드는 배열이 [1, 2, 3, 1, 2, 3]과 같이 각 숫자마다 등장하는 숫자가 모두 "정확하게" 같을 때만 작동한다.
예를 들어 [1, 1, 2, 2, 2, 2]라고 가정했을 때, [1, 1], [2, 2], [2, 2]로 그룹을 나눌 수 있어 조건을 만족한다. 각 숫자마다 반드시 하나의 그룹에 들어가야하는 것이 아니다.

그래서 생각해낸 것이 for문 안에 있는 `card.second % minCount != 0` 조건이다. 위의 배열이 주어졌을 때 map의 모습은 다음과 같다 [1: 2, 2: 4]. 이때 가장 작은 등장 수인 2로 숫자 '2'의 등장 수인 4를 나눴을 때 나머지가 없다면 그룹을 지을 수 있다고 판단하는 것이다.

하지만 이 또한 맞는 조건이 아니다.
### 풀이 2. 정답 코드
```cpp
class Solution {
public:
    bool hasGroupsSizeX(vector<int>& deck) {
        unordered_map<int, int> occurrences;
        
        for (const int card : deck) {
            occurrences[card]++;
        }
        
        // checking the GCD of every occurrence
        int gcdValue = 0;
        for (const pair<int, int> card : occurrences) {
            gcdValue = gcd(gcdValue, card.second);
        }
        
        // if gcd is greater than 2, it means it can have groups having same sizes
        // and the smallest length of a group is greater than 1 at the same time
        return gcdValue >= 2;
    }
};
```

위의 문제점들을 보완한 코드이다. <br />
`map`에서의 최소 등장 수를 구해 그것으로 나누어지는지 판별하는 것이 아니라, 각 숫자의 등장 수의 최대공약수가 2보다 더 크거나 같은지를 판별해야한다.

예를 들어 `[1, 1, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3, 3, 3]` 배열이 주어졌을 때, 1이 4번, 2가 4번, 3이 6번 등장한다.
이때 원래 코드라면 `minCount`가 4가 돼 3의 등장 수인 6과 비교할 때 6 % 4 == 2이라서 false를 리턴한다.

하지만 새로운 코드에서는 4와 6의 최대공약수는 2이기 때문에 원소 2개씩 가지는 그룹으로 나눌 수 있다는 의미가 되어 참이 된다. 
```
[1,1] [1,1] [2,2] [2,2] [3,3] [3,3] [3,3]
```

해당 코드의 시간 복잡도는 `O(n(deck의 길이) + m(occurrences의 길이))`로 총 `O(n + m)`이 되어 최종적으로 선형 복잡도를 가진다.<br />
`map`을 하나 사용하고 있는데 `map`은 `deck`에 들어있는 숫자들의 중복없이 들어가기 때문에 `O(n)`을 모두 차지하지는 않지만 최악의 상황(deck의 모든 숫자가 반복하지 않을 때)에서는 선형 복잡도를 가진다.

> 시간 복잡도: O(n), 공간 복잡도 O(n)