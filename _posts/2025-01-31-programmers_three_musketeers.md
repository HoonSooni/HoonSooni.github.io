---
title: "Programmers lv.1 - 삼총사"
date: 2025-01-31 00:10:00 +0800
categories: [CodingTest, Programmers]
tags: [coding test, algorithm, programmers] 
description: 프로그래머스 레벨1 삼총사 문제 풀이.
---

[https://school.programmers.co.kr/learn/courses/30/lessons/131705](https://school.programmers.co.kr/learn/courses/30/lessons/131705)

## 문제 설명 (난이도 1)

한국중학교에 다니는 학생들은 각자 정수 번호를 갖고 있습니다. 이 학교 
학생 3명의 정수 번호를 더했을 때 0이 되면 3명의 학생은 삼총사라고 합니다. 예를 들어, 5명의 학생이 있고, 각각의 정수 
번호가 순서대로 -2, 3, 0, 2, -5일 때, 첫 번째, 세 번째, 네 번째 학생의 정수 번호를 더하면 0이므로 세 학생은 
삼총사입니다. 또한, 두 번째, 네 번째, 다섯 번째 학생의 정수 번호를 더해도 0이므로 세 학생도 삼총사입니다. 따라서 이 경우
 한국중학교에서는 두 가지 방법으로 삼총사를 만들 수 있습니다.

한국중학교 학생들의 번호를 나타내는 정수 배열 `number`가 매개변수로 주어질 때, 학생들 중 삼총사를 만들 수 있는 방법의 수를 return 하도록 solution 함수를 완성하세요.
<hr />
### 제한사항

- 3 ≤ `number`의 길이 ≤ 13
- 1,000 ≤ `number`의 각 원소 ≤ 1,000
- 서로 다른 학생의 정수 번호가 같을 수 있습니다.
<hr />
<table>
  <thead>
    <tr>
      <th>Number List</th>
      <th>Result</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>[-2, 3, 0, 2, -5]</td>
      <td>2</td>
    </tr>
    <tr>
      <td>[-3, -2, -1, 0, 1, 2, 3]</td>
      <td>5</td>
    </tr>
    <tr>
      <td>[-1, 1, -1, 1]</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
<hr />
### 입출력 예 설명

**입출력 예 #1**

- 문제 예시와 같습니다.

**입출력 예 #2**

- 학생들의 정수 번호 쌍 (-3, 0, 3), (-2, 0, 2), (-1, 0, 1), (-2, -1, 3), (-3, 1, 2) 이 삼총사가 될 수 있으므로, 5를 return 합니다.

**입출력 예 #3**

- 삼총사가 될 수 있는 방법이 없습니다.
<hr />
## 문제 풀이

### 투포인터
```cpp
#include <string>
#include <vector>
#include <algorithm>

using namespace std;

int solution(vector<int> number) { 
    int answer = 0;
    const unsigned short even = 0; 
    const unsigned short size = number.size(); 
    unsigned short iteration = 0; 

    if (size % 2 == even) { iteration = (size / 2) - 1; } 
    else { iteration = size / 2; }

    // 정렬하기    
    sort(number.begin(), number.end()); 

    // 만약 배열 안에 0이 있다면 다르게 처리해주기 위한 변수
    bool isZero = true ? count(number.begin(), number.end(), 0) > 0 : false;

    for (int i = 0; i < iteration; ++i) { 
        unsigned short left = 0 + i;
        unsigned short right = (size - 1) - i;
        short addition = number[left] + number[right]; 

        if (addition < 0) { addition += number[right - 1]; } 
        else if (addition > 0) { addition += number[left + 1]; } 
        else { if(isZero == false) continue; }

        if (addition == 0) { answer++; }   
    }        
    
    return answer;
}
```

처음에 이 문제를 보고 완전탐색으로 풀 생각은 전혀 하지못했다. 그래서 머리를 싸메고 고민하다 투포인터 방법을 고안해냈고 애먹어가며 복잡하게 구현을 마쳤다.

나는 배열을 정렬한 뒤 앞 뒤 앞 뒤를 비교해가며 합이 0이 되는 
삼총사를 찾으면 쉽게 풀릴 줄 알았다. 하지만 두 번째 예시 반례에서 오답이 나왔다. -3, -2, -1, 0, 1, 2, 3에서
 (-3, 0, 3), (-2, 0, 2), (-1, 0, 1)의 경우들만 잡아내고 (1, 2, -3)같은 경우는 고려하지 못한 
것이다.

이러한 상황을 투포인터로 해결하는 방법은 떠올리지 못해 포기했고 우선 완전탐색 풀이부터 시도해보자는게 다음 계획이었다.

### 완전탐색
```cpp
for (int i = 0; i < number.size(); ++i) { 
    for (int j = i + 1; j < number.size(); ++j) { 
        for (int k = j + 1; k < number.size(); ++k) { 
            if (number[i] + number[j] + number[k] == 0) { answer++; } 
        } 
    }
}
```

가장 간단한 방법은 위 방법처럼 3중 for문을 사용하는 것이다. n중 for문은 배열에서 원소 n개를 포함하는 조합을 구할 때 유용하게 쓰인다.

그런데 이렇게만 끝내버리면 공부하는 의미가 없는 것 같아서 이번엔 ‘재귀함수’를 이용한 풀이 방식도 고민해봤다.
### 완전탐색 (DFS 재귀함수)
```cpp
#include <vector>

using namespace std;

unsigned int answer = 0;

void DFS(
    const vector<int>& number, 
    unsigned int index, 
    unsigned int count, 
    unsigned int sum)
{
    if (count == 3)
    {
        if (sum == 0) { answer++; return; }
    }
    
    if (index >= number.size()) { return; }
    
    DFS(number, index + 1, count + 1, sum + number[index]);
    DFS(number, index + 1, count, sum);
}

int solution(vector<int> number)
{
    DFS(number, 0, 0, 0);
    return answer;
}
```
특별한 것은 없고 단순히 위의 3중 for문을 재귀함수로 구현한 것 뿐이다. 재귀함수가 항상 그러하듯 직관적으로 이해하기가 힘들 것 같아 해당 코드의 과정을 담은 짧은 예시 영상을 준비했다.

<iframe width="560" height="315" src="https://www.youtube.com/embed/nmC9lsu_GvY?si=Chxa7CFhx8W3858_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>