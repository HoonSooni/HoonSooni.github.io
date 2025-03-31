---
title: "드림핵 lv.1 - what-is-my-ip"
date: 2025-03-31 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 what-is-my-ip 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1186](https://dreamhack.io/wargame/challenges/1186)
# 문제 설명
How are they aware of us even behind the wall?

**FYI**
Flag Location: `/flag`  
Flag Format: `DH{...}`

--- 
# 문제 풀이
## 웹사이트 분석 
난이도 1의 문제이기 때문에 역시나 웹사이트도 상당히 간단하다.<br />
그냥 나의 IP 주소를 출력해주는 기능을 하고있다.
# 코드 분석
### app.py
```python
#!/usr/bin/python3
import os
from subprocess import run, TimeoutExpired
from flask import Flask, request, render_template

app = Flask(__name__)
app.secret_key = os.urandom(64)

@app.route('/')
def flag():
    user_ip = request.access_route[0] if request.access_route else request.remote_addr
    try:
        result = run(
            ["/bin/bash", "-c", f"echo {user_ip}"],
            capture_output=True,
            text=True,
            timeout=3,
        )
        return render_template("ip.html", result=result.stdout)

    except TimeoutExpired:
        return render_template("ip.html", result="Timeout!")

app.run(host='0.0.0.0', port=3000)
```
이게 거의 서버 코드의 전부라고 할 정도로 간단한 웹서비스이다.<br />

`flag()` 함수의 행동을 보면 들어오는 요청으로부터 ip 주소를 access_route에서 읽어와서 `/bin/bash -c echo ip_address`를 출력한다. <br />
여기서 `echo` 다음으로 오는 값이 바로 `access_route[0]` 값이기 때문에 해당 값을 조작하면 될 것 같다.<br />
## 최종 풀이
### request.access_route
해당 서버는 Flask를 이용해 구현됐고 flask의 request.access_route 설명을 공식 문서에서 살펴보면 아래와 같은 설명을 볼 수 있다.<br />
그리고 그 아래의 코드는 access_route() 함수의 

> If a forwarded header exists this is a list of all ip addresses from the client ip to the last proxy server.

```python
    @cached_property
    def access_route(self) -> list[str]:
        """If a forwarded header exists this is a list of all ip addresses
        from the client ip to the last proxy server.
        """
        if "X-Forwarded-For" in self.headers:
            return self.list_storage_class(
                parse_list_header(self.headers["X-Forwarded-For"])
            )
        elif self.remote_addr is not None:
            return self.list_storage_class([self.remote_addr])
        return self.list_storage_class()
```
`access_route`는 클라이언트가 거친 프록시 ip 주소들의 리스트를 가지고 있다고 한다.<br />

파이썬 코드에서 access_route를 ctrl + 클릭으로 들어가보면 위와 같은 코드를 볼 수 있는데, 해당 함수는 HTTP 헤더를 받고 X-Forwarded-For 이라는 키에 값이 존재하면 그 값들을 request 객체의 access_router이라는 속성에 초기화된다.<br />
만일 X-Forwarded-For가 헤더에 없다면 remote_addr 값이 반환된다(이것도 물론 remote_addr 값을 얻을 수 있을 때를 의미한다).<br />
### X-Forwarded-For 값 변경
Burp Suite같은 프록시 툴을 이용해서 헤더를 변경하여 요청을 보내면 된다.<br />
우선 테스트 차원에서 `X-Forwarded-For: HELLO`라고 입력하여 요청을 보내고 페이지를 확인해보면 우리가 의도한데로 HELLO가 출력된다.<br />

```python
result = run(
	["/bin/bash", "-c", f"echo {user_ip}"],
	capture_output=True,
	text=True,
	timeout=3,
)
return render_template("ip.html", result=result.stdout)
```
서버 코드를 다시 한 번 살펴보면 우리의 access_route 값이 user_ip로 echo 다음에 들어가게 된다.<br />
그렇다는 것은 echo 다음에 플래그를 입력하면 화면에서 플래그를 얻을 수 있다는 것이다.<br />

문제 설명을 보면 플래그의 위치를 `/flag`라고 친절하게 알려주고 있다.<br />
그렇기 때문에 echo 다음에 `$(cat /flag)`라고 입력하여 플래그를 출력시킬 수 있으니 이에 맞게 X-Forwarded-For 값을 조정해주면 페이지에 플래그가 출력된다.
# 배운 것
HTTP 헤더의 X-Forwarded-For 값의 의미와 플라스크에서 해당 값을 어떤 식으로 가져오는지 배웠다.