---
title: "webhacking.kr old 26 문제 풀이"
date: 2025-04-10 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 26 문제 풀이
---

[https://webhacking.kr/challenge/web-11/](https://webhacking.kr/challenge/web-11/)
# 문제 풀이
```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Challenge 26</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }    
a { color:lightgreen; }
</style>
</head>
<body>
<?php
  if(preg_match("/admin/",$_GET['id'])) { echo"no!"; exit(); }
  $_GET['id'] = urldecode($_GET['id']);
  if($_GET['id'] == "admin"){
    solve(26);
  }
?>
<br><br>
<a href=?view_source=1>view-source</a>
</body>
</html>
```
웹페이지에 접속하면 아무런 정보가 없고 소스코드만 볼 수 있도록 해놓았다.<br />
소스코드를 보면 "admin"이라는 글자가 들어오면 필터링을 하지만 우리가 해야할 것은 "admin"을 입력하여 solve() 함수를 호출하는 것이다.<br />

해당 필터링을 우회하는 것이 관건이다.<br />
## 최종 풀이
자세히 보면 필터링 이후에 urldecode() 함수를 이용하는 것이 보인다.<br />
그리고 유저로부터 id 값을 GET으로 받고있다.<br />

브라우저는 우리가 GET으로 보내는 값에 URL 디코딩을 수행한다.<br />
특히 어떠한 문자를 ascii(hex) 값으로 보내려면 "%09", "%61"처럼 앞에 '%' 기호를 붙이면 된다.<br />
우리는 결국 "admin"이라는 글자를 필터링에 걸리지 않도록 하면 그만이기 때문에 "%2509dmin"이라는 값을 보내면 된다.<br />

왜냐하면 %25는 % 기호를 의미하기 때문에 처음에 서버로 요청을 보낼 때 브라우저에 의해 자동으로 "%09dmin"으로 변환되고 서버 내의 urldecode() 함수에 의해서 다시 한 번 디코딩이 돼 최종적으로 "admin"를 얻게되는 것이다.<br />