---
title: "LeetCode Easy - 2423. Remove Letter To Equalize Frequency"
date: 2025-02-01 00:00:00 +0800
categories: [CodingTest, LeetCode]
tags: [coding test, algorithm, leetcode] 
description: 릿코드 Easy Remove Letter To Equalize Frequency 문제 풀이.
---
[https://leetcode.com/problems/remove-letter-to-equalize-frequency/description/](https://leetcode.com/problems/remove-letter-to-equalize-frequency/description/)

## 문제 설명 (난이도 Easy)

You are given a **0-indexed** string `word`, consisting of lowercase English letters. You need to select **one** index and **remove** the letter at that index from `word` so that the **frequency** of every letter present in `word` is equal.

Return `true` if it is possible to remove one letter so that the frequency of all letters in `word` are equal, and `false` otherwise.

Note:

* The frequency of a letter x is the number of times it occurs in the string.
* You must remove exactly one letter and cannot choose to do nothing.

 

Example 1:

> **Input:** word = "abcc"
**Output:** true
**Explanation:** Select index 3 and delete it: word becomes "abc" and each character has a frequency of 1.

Example 2:

> **Input:** word = "aazz" 
**Output:** false
**Explanation:** We must delete a character, so either the frequency of "a" is 1 and the frequency of "z" is 2, or vice versa. It is impossible to make all present letters have equal frequency.

 

Constraints:
* 2 <= word.length <= 100 
* word consists of lowercase English letters only.
<hr />

## 문제 풀이
### 풀이 1. 가능한 모든 테스트 케이스 검열
```cpp
class Solution {
public:
    bool equalFrequency(string word) {
        unordered_map<char, int> checker;
        
        // Count the frequency of each character
        for (const char letter : word) {
            checker[letter]++;
        }
        
        // Create a frequency map
        unordered_map<int, int> freqCount;
        for (const auto& pair : checker) {
            freqCount[pair.second]++;
        }
        
        // If there's only one character, we can always remove it
        if (freqCount.size() == 1) {
            int freq = freqCount.begin()->first;
            // If the frequency is 1, we can remove any character
            if (freq == 1) return true;
            // If all characters have the same frequency > 1, we can remove one character
            if (checker.size() == 1) return true;
        }
        
        // If there are more than 2 different frequencies, it's impossible
        if (freqCount.size() != 2) {
            return false;
        }
        
        // Get the two frequencies
        auto it = freqCount.begin();
        int freq1 = it->first;
        int count1 = it->second;
        it++;
        int freq2 = it->first;
        int count2 = it->second;
        
        // Case 1: One character has a frequency of 1, and the rest have the same frequency
        if ((freq1 == 1 && count1 == 1) || (freq2 == 1 && count2 == 1)) {
            return true;
        }
        
        // Case 2: One character has a frequency that is one more than the others
        if ((freq1 - freq2 == 1 && count1 == 1) || (freq2 - freq1 == 1 && count2 == 1)) {
            return true;
        }
        
        return false;
    }
};
```

map을 하나 이용해서 frequency(얼마나 등장했는가)를 체크해준 후에 frequency에 따라서 또 새로운 map에 값들을 넣어주는 방식을 사용했다.<br />
그 이후에 적절한 조건문을 이용해서 가능한 모든 경우를 다 체크해주는 로직을 짜는데 상당히 애를 먹었다. 더 쉬운 방법이 있을 거라고 짐작은 하고 있었지만 이미 해당 방식으로 코드를 짜기 시작했기 때문에 우선 끝을 봐야겠다 하여 풀게됐다.

조건문이 많아서 이해가 힘들 수 있지만 차근차근 확인해보면 별로 복잡하지 않다는 것을 알 수 있다.

1. 첫 번째 조건문
    ```cpp
    // If there's only one character, we can always remove it
    if (freqCount.size() == 1) {
        int freq = freqCount.begin()->first;
        // If the frequency is 1, we can remove any character
        if (freq == 1) return true;
        // If all characters have the same frequency > 1, we can remove one character
        if (checker.size() == 1) return true;
    }
    ```
    이 부분은 freqCount의 길이가 1일 때 작동하는 코드이다. freqCount가 1이라는 것은 주어진 문자열의 모든 알파벳들의 frequency가 동일하다는 것이다.
    `"abc"`, `"aabbcc"`, `"aaabbbccc"`같은 경우를 의미한다. 이러한 경우에 `freqCount`는 {1 : n}을 가지기 때문이다. 그렇지만 `"abc"`의 경우엔 괜찮지만 `"aabbcc"`의 경우엔 글자 하나를 지울 수 없기 때문에 `freqCount.begin()`의 `first` 값이 1일 때 `true`를 리턴한다.

    두번째 `checker`의 길이를 체크하는 이유는 `"zz"`같은 경우를 대비한 것이다. 모든 글자가 다 같다면 하나를 지워도 frequency는 고유하기 때문이다.
    <br />
2. 두 번째 조건문 
    ```cpp
    // If there are more than 2 different frequencies, it's impossible
    if (freqCount.size() != 2) {
        return false;
    }
    ```
    만약 문자열이 `"aaabbc"`처럼 frequency가 2가지 경우 이상일 때는 어떤 짓을 해도 고유한 frequency를 얻을 수 없기 때문에 false를 리턴해주는 조건이 필요하다.
    <br />
3. 세 번째 조건문
    ```cpp
    // Get the two frequencies
    auto it = freqCount.begin();
    int freq1 = it->first;
    int count1 = it->second;
    it++;
    int freq2 = it->first;
    int count2 = it->second;
    
    // Case 1: One character has a frequency of 1, and the rest have the same frequency
    if ((freq1 == 1 && count1 == 1) || (freq2 == 1 && count2 == 1)) {
        return true;
    }
    ```
    우선 조건문을 간단하게 만들기 위해서 2개의 `freqCount` 원소를 세부적으로 나눠줬다.

    세 번째로는 `"abbcc"`와 같이 딱 한 글자의 frequency만 1이고 나머지는 같을 때의 경우이다. 이때 frequency가 1인 글자만 딱 지워주면 해결되기 때문이다.
    <br />
4. 네 번째 조건문
    ```cpp
    // Case 2: One character has a frequency that is one more than the others
    if ((freq1 - freq2 == 1 && count1 == 1) || (freq2 - freq1 == 1 && count2 == 1)) {
        return true;
    }
    ```
    이번에는 `"abbcc"`처럼 하나의 글자만 적게 나타는 경우가 아니라 `"abcc"`처럼 모든 글자가 다 1의 frequency를 가지는데 한 글자만 2번 등장하는 경우를 포착하기 위한 조건문이다.

    `(freq1 - freq2)` 혹은 `(freq2 - freq1)`이 1이라는 것은 해당 문자열에 다른 글자들은 다 같은 횟수로 등장하는데 딱 한 글자만 한 번 더 등장했다는 의미가 된다. 
    다른 것으로 예를 들면 `"sseettt"`처럼 s와 e가 두 번씩 등장했는데 t가 세 번 등장하여 t 하나만 제거해주면 조건을 만족하게 되는 경우를 체크하는 것이다.
    `count1`과 `count2`가 1인지 체크해주는 것은 "ttt"같이 많이 등장한 글자가 문자열에서 단 하나인지 체크해준다.

> 시간 복잡도: O(N), 공간 복잡도: O(N)

### 풀이 2. 겹치는 문자를 하나씩 제거
```cpp
class Solution {
public:
    bool equalFrequency(string word) {
        unordered_map<char, int> frequencyPerChar;
        
        // counting frequencies
        for (char letter : word) {
            frequencyPerChar[letter]++;
        }
        
        for (pair<char, int> freqFirst : frequencyPerChar) {
            // trying to remove a character
            frequencyPerChar[freqFirst.first]--;
            
            // checking if the elimination satisfies the condition or not
            unordered_set<int> freqSet;
            for (pair<char, int> freqSecond : frequencyPerChar) {
                if (freqSecond.second > 0) {
                    freqSet.insert(freqSecond.second);
                }
            }
            
            if (freqSet.size() == 1) return true;

            frequencyPerChar[freqFirst.first]++;
        }
        
        return false;
    }
};
```
이번엔 좀 연산량은 늘어났지만 구현하기에 훨씬 편한 방법으로 풀어보았다. 
frequency를 세는 코드는 이전과 똑같다. 다른 점은 이번엔 문자 하나하나를 직접 지워보면서 문자열이 고유한 frequency를 가진 문자인지 확인하는 방식이다.

`"abcc"`가 주어졌을 때 `frequencyPerChar`은 `[{'a': 1}, {'b': 1}, {'c': 2}]`처럼 될 것이다. 이때 a를 지워보고 "bcc"를 체크, 다음으로 b를 지워보고 "acc"를 체크, 다음으로 c를 지워보고 "abc"를 체크하는 원리이다.

c를 하나 지웠을 때 `freqSet`은 1이라는 원소를 하나만 가지고 있으니까 참을 반환한다.

모든 문자를 하나하나 빼보아도 만족하는 경우가 없으면 `false`를 리턴하며 끝내는 코드이다.

이중 반복문을 쓰는 만큼 코드 자체는 비효율 적이지만 구현은 굉장히 간단한 풀이었다.

> 시간 복잡도: O(N^2), 공간 복잡도: O(N)