---
title: "Programmers lv.1 - 유연 근무제"
date: 2025-02-10 00:00:00 +0800
categories: [CodingTest, Programmers]
tags: [coding test, algorithm, programmers] 
description: 프로그래머스 레벨1 유연 근무제 문제 풀이.
---

[https://school.programmers.co.kr/learn/courses/30/lessons/388351](https://school.programmers.co.kr/learn/courses/30/lessons/388351)

## 문제 설명 (난이도 1)

프로그래머스 사이트를 운영하는 그렙에서는 재택근무와 함께 출근 희망 시각을 자유롭게 정하는 유연근무제를 시행하고 있습니다. 제도 정착을 위해 오늘부터 일주일 동안 각자 설정한 출근 희망 시각에 늦지 않고 출근한 직원들에게 상품을 주는 이벤트를 진행하려고 합니다.

직원들은 일주일동안 자신이 설정한 `출근 희망 시각 + 10분`까지 어플로 출근해야 합니다. 예를 들어 출근 희망 시각이 9시 58분인 직원은 10시 8분까지 출근해야 합니다. **단, 토요일, 일요일의 출근 시각은 이벤트에 영향을 끼치지 않습니다.** 직원들은 매일 한 번씩만 어플로 출근하고, 모든 시각은 시에 100을 곱하고 분을 더한 정수로 표현됩니다. 예를 들어 10시 13분은 1013이 되고 9시 58분은 958이 됩니다.

당신은 직원들이 설정한 출근 희망 시각과 실제로 출근한 기록을 바탕으로 상품을 받을 직원이 몇 명인지 알고 싶습니다.

직원 `n`명이 설정한 출근 희망 시각을 담은 1차원 정수 배열 `schedules`, 직원들이 일주일 동안 출근한 시각을 담은 2차원 정수 배열 `timelogs`, 이벤트를 시작한 요일을 의미하는 정수 `startday`가 매개변수로 주어집니다. 이때 상품을 받을 직원의 수를 return 하도록 solution 함수를 완성해주세요.

---

##### 제한사항

- 1 ≤ `schedules`의 길이 = `n` ≤ 1,000
    - `schedules[i]`는 `i + 1`번째 직원이 설정한 출근 희망 시각을 의미합니다.
    - 700 ≤ `schedules[i]` ≤ 1100
- 1 ≤ `timelogs`의 길이 = `n` ≤ 1,000
    - `timelogs[i]`의 길이 = 7
    - `timelogs[i][j]`는 `i + 1`번째 직원이 이벤트 `j + 1`일차에 출근한 시각을 의미합니다.
    - 600 ≤ `timelogs[i][j]` ≤ 2359
- 1 ≤ `startday` ≤ 7
    - 1은 월요일, 2는 화요일, 3은 수요일, 4는 목요일, 5는 금요일, 6은 토요일, 7은 일요일에 이벤트를 시작했음을 의미합니다.
- 출근 희망 시각과 실제로 출근한 시각을 100으로 나눈 나머지는 59 이하입니다.

---

##### 테스트 케이스 구성 안내

아래는 테스트 케이스 구성을 나타냅니다. 각 그룹 내의 테스트 케이스를 모두 통과하면 해당 그룹에 할당된 점수를 획득할 수 있습니다.

| 그룹  | 총점  | 추가 제한 사항                                             |
| --- | --- | ---------------------------------------------------- |
| #1  | 10% | `n` = 1. 이벤트 시작일이 월요일이고, 출근 희망 시각이 정각으로 된 입력만 주어집니다. |
| #2  | 10% | 이벤트 시작일이 월요일이고, 출근 희망 시각이 정각으로 된 입력만 주어집니다.          |
| #3  | 15% | 출근 희망 시각이 정각으로 된 입력만 주어집니다.                          |
| #4  | 15% | 이벤트 시작일이 월요일인 입력만 주어집니다.                             |
| #5  | 50% | 제한사항 외 추가조건이 없습니다.                                   |

---

##### 입출력 예

| schedules              | timelogs                                                                                                                                               | startday | result |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | -------- | ------ |
| `[700, 800, 1100]`     | `[[710, 2359, 1050, 700, 650, 631, 659], [800, 801, 805, 800, 759, 810, 809], [1105, 1001, 1002, 600, 1059, 1001, 1100]]`                              | 5        | 3      |
| `[730, 855, 700, 720]` | `[[710, 700, 650, 735, 700, 931, 912], [908, 901, 805, 815, 800, 831, 835], [705, 701, 702, 705, 710, 710, 711], [707, 731, 859, 913, 934, 931, 905]]` | 1        | 2      |

---

##### 입출력 예 설명

**입출력 예 #1**

이벤트를 시작한 날은 금요일입니다. 직원 3명의 일주일 간 출근 시각을 나타내면 다음과 같습니다.

|출근 희망 시각|출근 인정 시각|
|---|---|
|7:00|7:10|
|8:00|8:10|
|11:00|11:10|

| 금     | 토     | 일     | 월    | 화     | 수     | 목     |
| ----- | ----- | ----- | ---- | ----- | ----- | ----- |
| 7:10  | 23:59 | 10:50 | 7:00 | 6:50  | 6:31  | 6:59  |
| 8:00  | 8:01  | 8:05  | 8:00 | 7:59  | 8:10  | 8:09  |
| 11:05 | 10:01 | 10:02 | 6:00 | 10:59 | 10:01 | 11:00 |

모든 직원이 평일에 늦지 않고 출근했습니다. 따라서 상품을 받을 직원은 3명입니다.

**입출력 예 #2**

이벤트를 시작한 날은 월요일입니다. 직원 4명의 일주일 간 출근 시각을 나타내면 다음과 같습니다. 취소선으로 표시된 시각이 출근이 필요한 평일에 지각한 날입니다.

|출근 희망 시각|출근 인정 시각|
|---|---|
|7:30|7:40|
|8:55|9:05|
|7:00|7:10|
|7:20|7:30|

|월|화|수|목|금|토|일|
|---|---|---|---|---|---|---|
|7:10|7:00|6:50|7:35|7:00|9:31|9:12|
|~~9:08~~|9:01|8:05|8:15|8:00|8:31|8:35|
|7:05|7:01|7:02|7:05|7:10|7:10|7:11|
|7:07|~~7:31~~|~~8:59~~|~~9:13~~|~~9:34~~|9:31|9:05|

첫 번째, 세 번째 직원이 평일에 늦지 않고 출근했습니다. 따라서 상품을 받을 직원은 2명입니다.

---

## 문제 풀이
```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <string>
#include <map>
#include <unordered_map>
#include <set>
#include <unordered_set>
#include <numeric>

using namespace std;

bool determineAllowingTime(int preferredTime, int actualTime) {
    // converting preferred time into hour and minute
    int preferredMinute = preferredTime % 100;
    int preferredHour = (preferredTime - preferredMinute) / 100;
    
    // converting actual time into hour and minute
    int actualMinute = actualTime % 100;
    int actualHour = (actualTime - actualMinute) / 100;
    
    // getting the allowed range of preferred time
    int allowedMinute = preferredMinute + 10;
    int allowedHour = preferredHour;
    if (allowedMinute > 59) {
        allowedHour += 1;
        allowedMinute -= 60;
    }

    bool isOnTime = true;
    if (allowedHour < actualHour) { isOnTime = false; }
    else if (allowedHour == actualHour) {
        if (allowedMinute < actualMinute) { isOnTime = false; }
    }
    
    return isOnTime;
}

int solution(vector<int> schedules, vector<vector<int>> timelogs, int startday) {
    int answer = 0;
    
    for (int i = 0; i < timelogs.size(); ++i) {
        int day = startday;
        bool isOnTime = true;
        
        for (int j = 0; j < 7; ++j) {
            if (day < 6) {
                if (determineAllowingTime(schedules[i], timelogs[i][j]) == false) {
                    isOnTime = false;
                    break;
                }
            }
            
            day++;
            if (day == 8) { day = 1; }
        }
        
        if (isOnTime == true) { answer++; }
    }
    
    return answer;
}
```

지난 주에(2025/02/10 기준) 프로그래머스 코딩 대회에서 출제됐던 문제이다.<br />
입출력 예시들에 비해 문제 설명은 상당히 간단하다. 프로그래머스 회사의 직원들의 희망 출근 시간을 담은 1차원 배열과 각 직원들의 일주일간의 출근 기록이 담긴 2차원 배열이 주어진다. <br />

주어진 매개변수들을 적절히 이용하여 유연근무제를 시행했을 때 실제로 출근시간을 제대로 지키는 직원의 수는 몇 명인가를 판별하는 문제이다.<br />

출근시간을 담은 배열들 뿐만 아니라 `startday`라는 변수도 주어지는데, 이는 1부터 7까지의 값을 갖고 1은 월요일 ~ 7은 일요일을 의미한다. 토요일과 일요일의 기록은 신경쓰지 않는다는 것이 조건이다.
즉, day가 6 혹은 7일 때는 해당 직원이 출근을 언제했는지 상관하지 않는다.

#### solution() 함수
```cpp
int solution(vector<int> schedules, vector<vector<int>> timelogs, int startday) {
    int answer = 0;
    
    for (int i = 0; i < timelogs.size(); ++i) {
        int day = startday;
        bool isOnTime = true;
        
        for (int j = 0; j < 7; ++j) {
            if (day < 6) {
                if (determineAllowingTime(schedules[i], timelogs[i][j]) == false) {
                    isOnTime = false;
                    break;
                }
            }
            
            day++;
            if (day == 8) { day = 1; }
        }
        
        if (isOnTime == true) { answer++; }
    }
    
    return answer;
}
```
가장 주가 되는 함수를 살펴보면 `timelogs[i]`(한 직원의 일주일간의 출근 기록)를 7번씩 돌아가며 `determineAllowingTime()` 함수를 이용해서 만일 단 하루라도 출근 시간을 제대로 지키지 않았다면 해당 직원은 출근 시간을 지키지 못한 것으로 판단한다. <br />
만을 일주일간 매일매일 출근 시간을 온전히 지켰다면 `answer`을 1 증가시켜 성실한 직원의 수를 기록하는 코드이다.

이때 헷갈릴만한 부분은 토요일, 일요일을 생략하는 부분이다. <br />
매개변수로 주어진 `startday`를 매 직원마다 `day`라는 변수에 초기화해주고, 해당 직원의 일주일 출근 기록을 살펴보면서 `day`를 1씩 증가하고 있다. `day`가 6이나 7이라는 것은 해당 기록이 토요일 혹은 일요일에 쓰인 기록이라는 것이기 때문이다. 그래서 코드에서는 if (day < 6)로 day가 6보다 더 작을 때만 판별하는 것이다. <br />
`day`를 1씩 계속 증가시키다가 만일 `day`가 8이 되면 `day`를 1로 초기화해준다. `startday`가 5(금요일)일 때 6(토), 일(7), 그 다음에 8이 아닌 1(월)로 넘어갈 수 있게 하기 위함이다.

#### determineAllowingTime() 함수
```cpp
bool determineAllowingTime(int preferredTime, int actualTime) {
    // converting preferred time into hour and minute
    int preferredMinute = preferredTime % 100;
    int preferredHour = (preferredTime - preferredMinute) / 100;
    
    // converting actual time into hour and minute
    int actualMinute = actualTime % 100;
    int actualHour = (actualTime - actualMinute) / 100;
    
    // getting the allowed range of preferred time
    int allowedMinute = preferredMinute + 10;
    int allowedHour = preferredHour;
    if (allowedMinute > 59) {
        allowedHour += 1;
        allowedMinute -= 60;
    }

    bool isOnTime = true;
    if (allowedHour < actualHour) { isOnTime = false; }
    else if (allowedHour == actualHour) {
        if (allowedMinute < actualMinute) { isOnTime = false; }
    }
    
    return isOnTime;
}
```
이 함수는 매개변수로 `preferredTime` 그리고 `actualTime`을 받는다.<br />
`preferredTime`은 각 직원의 희망 출근 시간을 의미하고 `actualTime`은 해당 직원의 진짜 출근 시간을 의미한다. 이 함수에서 가장 먼저 하는 일은 각 시간들을 비교하기 편하게 만들기 위해 시와 분을 분리하는 작업이다. <br />

> 모든 시각은 시에 100을 곱하고 분을 더한 정수로 표현됩니다. 예를 들어 10시 13분은 1013이 되고 9시 58분은 958이 됩니다.

문제 설명을 살펴보면 각 시간은 `(hour * 100) + minute`로 구성됐다는 걸 알 수 있다. 그래서 이를 역연산하여 시와 분을 알아내는 방식을 쓰고있다.<br />

```cpp
int allowedMinute = preferredMinute + 10;
int allowedHour = preferredHour;
if (allowedMinute > 59) {
	allowedHour += 1;
	allowedMinute -= 60;
}
```
여기서 또 신경써야 하는 부분은 희망하는 출근시간까지 반드시 출근해야 하는 것이 아니라 10분까지는 넘어가준다는 조건이다. 그래서 `preferredMinute`에 10을 더해주고 있는데 만일 이 더해준 분 값이 60분을 넘는다면 시간에 1을 더해주어야 하기 때문에 해당 과정을 추가해줬다.<br />
예를 들어 희망 출근 시간이 7:55이면 여기에 10분을 더하면 7:65가 된다. 시간을 표현하는 상황에서 60분이 넘는 것은 말이 안되기 때문에 이때 시에 1시간을 더해주고 분에서 60을 빼주어 제대로된 시간을 확보한다.<br />

```cpp
bool isOnTime = true;
if (allowedHour < actualHour) { isOnTime = false; }
else if (allowedHour == actualHour) {
	if (allowedMinute < actualMinute) { isOnTime = false; }
}

return isOnTime;
```
마지막으로 최종적으로 변환된 시간과 실제 출근했던 시간을 비교하여 정해진 시간 내에 제대로 출근했는지 판단해준다. <br />

해당 문제는 2중 반복문을 사용하고 있고 최악의 상황에서는 모든 직원의 모든 출근 기록을 전부(토, 일을 제외한 5일) 확인해야 하지만 `timelogs[i]`의 길이는 7로 항상 고정돼있다. 그리고 반복문에서 매번 호출하는 determineAllowingTime() 함수는 상수 복잡도를 가지기 때문에 시간 복잡도가 O(n * 7 + 1)이므로 최종적으로 O(N)이다.
<br />
주어진 매개변수 이외에는 다른 n개의 길이를 가지는 배열은 사용하지 않기 때문에 공간 복잡도는 O(1)이라고 할 수 있다.

> 시간 복잡도: O(N), 공간 복잡도: O(1)