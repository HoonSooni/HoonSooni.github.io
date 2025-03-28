---
title: "드림핵 lv.2 - DOM XSS"
date: 2025-03-28 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, xss, dom based xss] 
description: 드림핵 DOM XSS 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/438](https://dreamhack.io/wargame/challenges/438)
# 문제 설명
Exercise: DOM XSS에서 실습하는 문제입니다.
# 문제 풀이
## 웹사이트 분석
이번 문제 또한 드림핵의 XSS 관련 문제들의 특유의 형태를 갖추고 있기 때문에 자세한 분석은 하지 않아도 될 것 같다.<br />
memo 페이지, vuln 페이지, flag 페이지로 구성돼있는 페이지이다. <br />
memo 페이지는 "memo"라는 파라미터의 값을 읽어서 화면에 출력하는 페이지이고 flag 페이지는 셀레니움을 통해 127.0.0.1 주소로 HTTP 요청을 보내는 기능을 가지고 있다.<br />
vuln 페이지는 param을 받아서 그대로 화면에 리턴하기 때문에 HTML 태그를 사용할 수 있고 이번 문제에만 존재하는 hash 값을 받는다.<br />

`/vuln?param=something#hash`<br />
이런식으로 입력하면 화면에는 "hash is my name !"이라는 글자가 `<pre>` 태그 안에 적혀 출력된다.
즉, # 뒤에 붙는 단어가 이름 자리에 들어갈 문자열이 되는 것이다.<br />
## 코드 분석
### vuln.html
![[vuln.html code]](https://1drv.ms/i/c/5cb37aa515b56a00/IQS5R_9GAZdFS6FIGP-tB5hgAUz_hIF9yWDe1OQYSOobg_c?width=660)<br />
다른 부분들은 모두 이전 문제와 중복되는 부분이라 생략했다.<br />
vuln.html의 이 부분을 보면 해당 페이지는 hash 값을 받아서 "hash is my name !"이라는 문자를 "name"이라는 id를 가진 HTML 태그에 innerHTML로 넣어주는 코드이다.<br />

id="name"를 가진 태그는 여기서 `<pre>` 태그이다.
## 최종 풀이
해당 문제에서는 다양한 풀이가 존재할 수 있다.<br />
예를 들어 `<base>` 태그를 이용해 base uri를 조작하여 다른 자바스크립트를 실행하는 방식도 시도해볼 수 있지만 문제의 카테고리가 DOM-based XSS 공격이기 때문에 해당 문제 풀이 방식을 시도해보자.<br />
### alert() 함수 실행시키기
우선 우리가 해야할 것은 vuln 페이지에서 어떻게든 자바스크립트가 작동하도록 만드는 것이 목표이다.<br />
작동하는지 체크하는데 가장 편한 방법은 바로 alert() 함수를 실행시키는 것이다.<br />

처음 vuln 페이지에 접속하면 기본 URL은 `http://host3.dreamhack.games:8998/vuln?param=%3Cimg%20src=https://dreamhack.io/assets/img/logo.0a8aabe.svg%3E#dreamhack`이다.<br />
param으로 기본 img 태그를 사용하고 hash 값으로 "dreamhack"을 전달한다.<br />

이때 hash 값으로 img 태그를 `#<img src=@ onerror=alert(1) />` 이런식으로 전달해보면 `%3Cimg%20src=@%20onerror=alert(1)%20/%3E is my name !`가 결과로 나오는 것을 볼 수 있다.<br />

원래라면 작동해야 하지만 hash로 입력한 값이 `<pre>` 태그로 넘어가고 있어서 HTML 태그로 전달되는 것이 아니라 순수 문자열로 문서에 들어가서 그렇다.<br />
이때 `#&ltimg src=@ onerror=alert(1) /&gt`, `#&ltimg src=@ onerror=alert(1) /&gt``//` 등의 다양한 페이로드를 실험해보았는데 역시나 소용이 없었다.<br />

이때 문득 떠오른 것이 param 값을 조금 변형하면 될 것 같다는 느낌이 들었다.<br />
예를 들어 `?param=<div id='name'></div>#dreamhack` 이런식으로 하면 `document.getElementById("name");` 함수가 이제는 div 태그를 리턴하지 않을까 하는 생각이었다.<br />
결과를 확인해보면 예상대로 작동하는 것을 확인할 수 있다.<br />

그럼에도 불구하고 alert() 함수를 실행시키는 데에는 역부족이었는데 이번엔 div 태그가 아니라 script로 변경 해보았다. <br />
해당 문제에는 CSP로 script-src nonce 설정이 걸려있어서 아무런 스크립트나 실행할 수 없게끔 돼있어서 이러한 방식은 생각도 안하고 있었는데 그냥 시도해보니 작동을 하는 것이다.<br />

```
/vuln?param=<script id=name></script>#alert(1);//
```
작동한 페이로드는 위와 같다.<br />
나는 스크립트 태그는 당연히 사용할 수 없을 줄 알았는데 알고보니 CSP 설정값에 strict-dynamic 이라는 설정이 명시돼있는 것이 확인된다.<br />
#### CSP strict-dynamic
CSP 헤더에 strict-dynamic가 명시돼있으면 믿을만한 스크립트(nonce 값을 만족하는)가 동적으로 불러온 혹은 만들어낸 스크립트는 허용하는 성격이 있다. 그리고 이벤트 핸들러와 같은 녀석들을 모두 막는 성격도 동시에 있다. 그래서 이 전에 시도했던 img 태그가 작동하지 않았던 것이다.(pre 태그를 탈출하고 그 다음을 의미한다)<br />

더 자세한 내용은 [여기](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/script-src#strict-dynamic)서 살펴볼 수 있다.
### 익스플로잇
이제 스크립트를 작동시키는 방식을 알아냈으니 익스플로잇을 시도해보자.<br />

항상 그래왔던 것처럼 flag 페이지에 들어가 param값으로 `<script id=name></script>`를 넣고 hash값으로는 `location.href='/memo?memo'+document.cookie;//`를 작성하고 memo 페이지에 접속하면 플레그를 얻을 수 있다. 

참고로 hash 값 마지막에 ";//"를 넣어주는 이유는 SQL Injection을 할 때에 뒤의 코드를 주석처리 해주는 기법을 생각해보면 된다.<br />
그리고 큰 따옴표나 스페이스를 사용하면 URL 인코딩이 되기 때문에 제대로 작동하지 않는다. 그래서 작은 따옴표와 스페이스를 모두 제거한 코드를 사용해야한다.<br />
# 배운 것
DOM-based XSS 관련해서 조금이나가 감을 좀 잡을 수 있게됐다.<br />
이 문제를 처음 보았을 때 감도 안잡혔는데 뭔가 번뜩이며 풀이 방법이 떠오르는 아주 귀한 경험을 할 수 있는 문제였다.<br />

CSP의 `strict-dynamic`이라는 설정 값에 대해 알게돼 좋았다. 