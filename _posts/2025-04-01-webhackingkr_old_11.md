---
title: "webhacking.kr old 11 문제 풀이"
date: 2025-04-01 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, php, ] 
description: webhacking.kr old 11 문제 풀이
---

[https://webhacking.kr/challenge/code-2/](https://webhacking.kr/challenge/code-2/)
# 문제 풀이
## 웹사이트 분석
문제 페이지로 이동하면 "Wrong"이라는 문구와 함께 소스코드를 살펴볼 수 있는 링크를 제공한다.<br />

```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Challenge 11</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
</style>
</head>
<body>
<center>
<br><br>
<?php
  $pat="/[1-3][a-f]{5}_.*$_SERVER[REMOTE_ADDR].*\tp\ta\ts\ts/";
  if(preg_match($pat,$_GET['val'])){
    solve(11);
  }
  else echo("<h2>Wrong</h2>");
  echo("<br><br>");
?>
<a href=./?view_source=1>view-source</a>
</center>
</body>
</html>
```
링크를 타고 들어가면 해당 서버의 PHP 코드를 볼 수 있다.<br />
해당 서버는 유저로부터 GET 요청에 포함돼 들어오는 val 파라미터 값을 받아와서 특정 정규식을 만족하는지 체크하는 기능을 가지고 있다. 정규식을 만족한다면 문제를 해결한 것으로 간주하는 것이다.<br />

그렇다는 것은 val에 어떤 값을 넘겨주어야 문제가 풀릴까 고민하는 것이 유일한 목표인 셈이다.<br />
## 최종 풀이
### 정규식 분석
```php
"/[1-3][a-f]{5}_.*$_SERVER[REMOTE_ADDR].*\tp\ta\ts\ts/";
```
정규식은 위와 같다. 
1. `[1-3]`: 1, 2, 3 중 딱 한 글자만 매치.
2. `[a-f]{5}`: a, b, c, d, e, f 중 5글자 매치.
3. `.*$_SERVER[REMOTE_ADDR].*`: PHP에서 문자열에 변수를 입력하고 싶을 때 사용하는 문법. remote_addr 값은 더 많은 의미를 내포하지만 지금은 단순하게 유저의 IP 주소라고 봐도 무방.
4. `\tp\ta\ts\ts`은 보이는 그대로 탭p탭a탭s탭s를 의미.

이를 기반으로 정규식을 통과하는 예시 문자열을 뽑아낼 수 있다.<br />

```
2abcde_MY_IP_ADDRESS%09p%09a%09s%09s
```
중간에 MY_IP_ADDRESS 부분을 본인의 현재 ip 주소로 변경하면 된다. 예를 들어 자신의 IP가 100.100.100.100이라고 한다면 `2abcde_100.100.100.100%09~~~`로 입력해야 한다는 말이다.<br />

브라우저 URL 부분에 탭을 직접 입력하기도 힘들 뿐더러 어거지로 하게되면 URL 인코딩에 의해서 무시가 되기 때문에 인코딩에 맞는 `%09`를 입력해주는 것이 핵심이다.<br />
### 익스플로잇
이제 다시 문제 페이지로 돌아가서 URL 맨 뒤에 `2abcde_MY_IP_ADDRESS%09p%09a%09s%09s`를 입력해주면 문제가 해결됐다는 alert 창이 뜨면서 끝이 난다.