---
title: "Programmers lv.1 - [PCCP 기출문제] / 10번 공원"
date: 2025-02-06 00:00:00 +0800
categories: [IT, 코딩테스트]
tags: [coding test, algorithm, programmers] 
description: 프로그래머스 레벨1 [PCCP 기출문제] / 10번 공원 문제 풀이.
---

## 문제 설명 (난이도 1)

지민이는 다양한 크기의 정사각형 모양 돗자리를 가지고 공원에 소풍을 나왔습니다. 공원에는 이미 돗자리를 깔고 여가를 즐기는 사람들이 많아 지민이가 깔 수 있는 가장 큰 돗자리가 어떤 건지 확인하려 합니다. 예를 들어 지민이가 가지고 있는 돗자리의 한 변 길이가 5, 3, 2 세 종류이고, 사람들이 다음과 같이 앉아 있다면 지민이가 깔 수 있는 가장 큰 돗자리는 `3x3` 크기입니다.

![map-example](https://1drv.ms/i/c/5cb37aa515b56a00/IQQYCLZ7qkOrT6Hgh5BwqqTeAdBI2UC-Jdl_2qTCXi-E0Is?width=1024)

지민이가 가진 돗자리들의 한 변의 길이들이 담긴 정수 리스트 `mats`, 현재 공원의 자리 배치도를 의미하는 2차원 문자열 리스트 `park`가 주어질 때 지민이가 깔 수 있는 가장 큰 돗자리의 한 변 길이를 return 하도록 solution 함수를 완성해 주세요. 아무런 돗자리도 깔 수 없는 경우 -1을 return합니다.
<hr />

### 제한사항
* 1 ≤ `mats`의 길이 ≤ 10
        1 ≤ `mats`의 원소 ≤ 20
        `mats`는 중복된 원소를 가지지 않습니다.

* 1 ≤ `park`의 길이 ≤ 50
        1 ≤ `park[i]`의 길이 ≤ 50
        `park[i][j]`의 원소는 문자열입니다.
        `park[i][j]`에 돗자리를 깐 사람이 없다면 `"-1"`, 사람이 있다면 알파벳 한 글자로 된 값을 갖습니다.
<hr />

### 입출력 예
<table border="1">
  <tr>
    <th>mats</th>
    <th>park</th>
    <th>result</th>
  </tr>
  <tr>
    <td>[5,3,2]</td>
    <td>
      <table border="1">
        <tr><td>A</td><td>A</td><td>-1</td><td>B</td><td>B</td><td>B</td><td>B</td><td>-1</td></tr>
        <tr><td>A</td><td>A</td><td>-1</td><td>B</td><td>B</td><td>B</td><td>B</td><td>-1</td></tr>
        <tr><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td></tr>
        <tr><td>D</td><td>D</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>E</td><td>-1</td></tr>
        <tr><td>D</td><td>D</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>F</td></tr>
        <tr><td>D</td><td>D</td><td>-1</td><td>-1</td><td>-1</td><td>-1</td><td>E</td><td>-1</td></tr>
      </table>
    </td>
    <td>3</td>
  </tr>
</table>
<hr />

## 문제 풀이
### 풀이1. 완전탐색을 이용한 풀이
```cpp
int solution(vector<int> mats, vector<vector<string>> park) {
    unsigned int max = 0;
    
    for (int i = 0; i < park.size(); ++i) {
        for (int j = 0; j < park[i].size(); ++j) {
            if (park[i][j] != "-1") { continue; }
            else {
                unsigned int size = 1;

                bool isExpandable = true;
                while (isExpandable) {
                    // checking the value right next to the last one
                    for (int x = i; x < i + size; ++x) {
                        if (x < park.size() && j + size < park[0].size()) {
                            if (park[x][j + size] != "-1") {
                                isExpandable = false;
                                break;
                            }
                        } else { isExpandable = false; break; }
                    }
                    
                    if (isExpandable) {
                        // checking the last row
                        for (int x = j; x <= j + size; ++x) {
                            if (i + size < park.size() && x < park[0].size()) {
                                if (park[i + size][x] != "-1") {
                                    isExpandable = false;
                                    break;
                                }
                            } else { isExpandable = false; break; }
                        }
                        
                        size++;
                    } else {
                        // when isExpandable is false save the maximum value
                        max = std::max(max, size);
                    }
                }
            }
        }
    }
    
    // sorting mats[] to make the comparison easier
    sort(mats.begin(), mats.end());
    
    // comparing from the biggest value in the mats
    for (int i = mats.size() - 1; i >= 0; --i) {
        if (mats[i] <= max) { return mats[i]; }
    }
    
    return -1;
}
```

아무래도 4중 반복문을 이용한 완전탐색을 시행했기 때문에 코드가 은근히 부담스럽게 느껴진다.
우선 이 코드의 전체적인 순서는 다음과 같다.
1. `park[n][m]`을 처음부터 끝까지 반복을 돌면서 해당 원소가 "-1"인지 체크.
    - "-1"이 아니라면 `continue`.
    - "-1"이 맞다면 2번으로 점프.
2. 빈자리가 맞기 때문에 size를 1로 시작.
3. 이때 `park[i][j]`를 기준으로 정사각형 모양으로 뻗어나가면서 빈자리가 맞는지 체크.
    - 정사각형 모양을 체크하는 알고리즘은 다음과 같음.
        예) 문제의 입출력 예시에 나온 park를 이용했을 때, 그리고 `i == 2`, `j == 2`일 때.
        1. `park[i][j]`가 "-1"이니 `size`는 1.
        2. `park[i][j + size]`부터 `park[i + size][j + size]`가 "-1"인지 확인
        3. `park[i + size][j]`부터 `park[i + size][j + size]`가 "-1"인지 확인
        모두 맞다면 `size`에 1을 더하고 3번으로 다시 돌아감.
        더 확장할 수 없다고 판단되면 4번으로 넘어감.
4. `park[i][j]`에서 뻗어나갈 수 있는 정사각형의 최대 크기를 max에 저장.
5. `mats`를 오름차순으로 정렬하고 끝에서부터 시작부분으로 반복을 돌면서 `mats[i]`값이 `max`보다 작거나 같은지 체크.
    - 조건을 만족한다면 해당 원소 리턴.
    - 그렇지 않고 `mats`를 모두 돌았다면 `-1` 리턴.
<br />

**정사각형 판단 알고리즘**
![square-expansion-example](https://1drv.ms/i/c/5cb37aa515b56a00/IQS1FjVZw6ofQ4XpVDMLZ96WAX0GKi_isVevkuDJXVHIs-g?width=660)
<br />

여기서 3번 과정이 직관적으로 이해하기 힘든 부분이라 다시 한 번 살펴보자. `park[i][j]`에서 정사각형 모양으로 뻗어나가는 알고리즘을 그림으로 나타내보았다.

정사각형이 1x1일 때부터 4x4가 될 때까지의 과정을 담아보았는데 여기서 일종의 패턴을 발견할 수 있다. 
* `i ~ (size - 1)`까지는 `j + size`의 값만 확인한다.
* 마지막 줄인 `i + size`에서는 `j ~ j + size`까지 값을 모두 확인한다.

즉, 현재 위치인 `[i][j]`에서 그보다 더 큰 정사각형이 되려면 각 라인의 `[j + 1]`을 모두 확인하고 마지막 가로 줄 전체를 확인하는 것이다. 

```cpp
if (park[i][j] != "-1") { continue; }
else {
    unsigned int size = 1;

    bool isExpandable = true;
    while (isExpandable) {
        // checking the value right next to the last one
        for (int x = i; x < i + size; ++x) {
            if (x < park.size() && j + size < park[0].size()) {
                if (park[x][j + size] != "-1") {
                    isExpandable = false;
                    break;
                }
            } else { isExpandable = false; break; }
        }
        
        if (isExpandable) {
            // checking the last row
            for (int x = j; x <= j + size; ++x) {
                if (i + size < park.size() && x < park[0].size()) {
                    if (park[i + size][x] != "-1") {
                        isExpandable = false;
                        break;
                    }
                } else { isExpandable = false; break; }
            }
            
            size++;
        } else {
            // when isExpandable is false save the maximum value
            max = std::max(max, size);
        }
    }
}
```

가장 바깥에 있는 2중 for문 안에 있는 이 코드가 정사각형 판단 알고리즘이다. 
<br />
`isExpandable`이라는 변수로 현재 크기보다 더 뻗어나갈 수 있는지 체크를 한다. <br />
만약 뻗어나갔을 때 "-1"이 아닌 문자열이 발견되거나 `park`의 크기를 벗어나는 경우에 이 변수가 `false`로 초기화되고 현재의 정사각형 탐색을 종료하고 다음 시작 부분으로 넘어갈 때 쓰인다.
<br />

이렇게 가능한 모든 정사각형의 크기를 확인하고 그 중 가장 큰 값을 변수 `max`에 저장한다. 

```cpp
// sorting mats[] to make the comparison easier
sort(mats.begin(), mats.end());

// comparing from the biggest value in the mats
for (int i = mats.size() - 1; i >= 0; --i) {
    if (mats[i] <= max) { return mats[i]; }
}

return -1;
```

마지막으로 `mats`에 공원에 깔 수 있는 매트가 있는지 체크를 해준다. <br />
우선 `mats`를 정렬해주고 끝에서부터 처음부분으로 반복을 돈다. 이때 `mats[i]`가 `max`보다 작거나 같다는 것은 해당 매트를 깔 수 있다는 의미이기에 반환해준다.

`mats`에 들어있는 그 어떤 매트도 깔 수 없다면 `-1`을 반환하면서 마무리한다.

시간 복잡도를 계산해보면
우선 가장 바깥에 있는 for문 2개는 2차원 배열의 처음부터 끝까지 반복하기 때문에 고려할 것도 없이 O(n^2)이다.
그리고 그 안에서 i부터 i + size까지, j부터 j + size까지 반복을 해주고 있기 때문에 worse case일 때(park 전체가 빈 공간일 때) O(n^2)이다.
마지막에 정렬 알고리즘도 사용하고 있지만 아마 O(n log n)일 것이고 이미 위에서 O(n^2) 이상을 차지하고 있기 때문에 고려할 대상은 아니다.

주어진 매개변수 이외에는 달리 n 크기를 가지는 배열을 딱히 사용하지 않았기 때문에 공간 복잡도는 상수이다.

> 시간 복잡도: O(N^4), 공간 복잡도: O(1)

### 풀이 2. DP를 이용한 간단한 풀이
[*해당 풀이는 모징이님의 풀이를 참고하였습니다.*](https://mojing.tistory.com/entry/ProgrammersC-PCCE-%EA%B8%B0%EC%B6%9C%EB%AC%B8%EC%A0%9C-10%EB%B2%88-%EA%B3%B5%EC%9B%90)
```cpp
int solution(vector<int> mats, vector<vector<string>> park) {
    int answer = -1;
    int maxSize = 0;
    
    const unsigned int columnSize = park.size();
    const unsigned int rowSize = park[0].size();
    
    vector<vector<int>> dp(columnSize, vector<int>(rowSize, 0));
    
    if (park[0][0] == "-1") { dp[0][0] = 1; }
    
    for (int i = 0; i < columnSize; ++i) {
        for (int j = 0; j < rowSize; ++j) {
            if (park[i][j] == "-1") {
                if (i == 0 || j == 0) {
                    dp[i][j] = 1;
                } else {
                    dp[i][j] = min({dp[i - 1][j], dp[i][j - 1], dp[i - 1][j - 1]}) + 1;
                }
            }
            maxSize = max(maxSize, dp[i][j]);
        }
    }
    
    sort(mats.rbegin(), mats.rend());
    for (int matSize : mats) {
        if (matSize <= maxSize) {
            answer = matSize;
            break;
        }
    }
    
    return answer;
}
```
4중 반복문을 필요로 했던 완전탐색 풀이의 최적화 버전인 DP 알고리즘을 적용한 풀이 방법이다. <br />
DP 풀이가 항상 그렇듯 가장 주목해야 할 부분은 DP 리스트를 초기화하는 부분이다. 맨 처음 리스트를 선언할 때부터 모든 원소를 0으로 초기화하고 시작한다. <br />
그 다음으로 반복문을 돌면서 park[i][j]의 원소가 "-1"일 때 dp[i][j]로부터 왼쪽칸, 윗칸, 왼쪽 위 대각칸에 있는 수 중에 가장 작은 수에서 1을 더한 수를 현재 dp[i][j]에 저장한다.

즉, dp[i][j]는 [i][j]가 가질 수 있는 가장 큰 정사각형의 크기를 담게 된다.

![dp-result](https://1drv.ms/i/c/5cb37aa515b56a00/IQTsvdm28tW1T77XWDbak2OoAfIPpEdI3Vq2XHGNqmvVkMI?width=320)
<br />
문제 입출력 예시에 있던 park 값으로 코드를 실행해서 dp의 초기화된 값들을 출력해보면 위 사진과 같다. 

```cpp
sort(mats.rbegin(), mats.rend());
for (int matSize : mats) {
    if (matSize <= maxSize) {
        answer = matSize;
        break;
    }
}
```

그리고 마지막으로 완전탐색에서도 했던 비슷한 방식으로 공원에 깔 수 있는 가장 큰 매트를 구하는 코드가 있다. 
아까는 그냥 오름차순으로 정렬하고 뒤에서부터 앞으로 체크했다면 이번엔 내림차순으로 정렬하여 좀 더 직관적으로 바꿨다.

dp를 초기화할 때 2중 반복을 돌고있고 그 이외에는 선형 반복을 한 번 돌고 있어서 시간 복잡도는 O(n^2).<br />
park와 같은 크기를 가진 dp 배열을 사용하고 있어서 공간 복잡도는 O(n^2). <br />
아무래도 시간이 줄어든 대신 공간을 더 사용하게 됐다.

> 시간 복잡도: O(N^2), 공간 복잡도: O(N^2)