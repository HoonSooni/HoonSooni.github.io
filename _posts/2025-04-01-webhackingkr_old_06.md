---
title: "webhacking.kr old 06 문제 풀이"
date: 2025-04-01 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, php] 
description: webhacking.kr old 06 문제 풀이
---

[https://webhacking.kr/challenge/web-06/](https://webhacking.kr/challenge/web-06/)
# 문제 풀이
## 웹사이트 분석
문제 페이지에 접속하자마자 `ID: guest PW: 123qwe`라는 문구가 보이고 소스코드를 볼 수 있는 a 태그가 있다.
## 코드 분석
```php
<?php
include "../../config.php";
if($_GET['view_source']) view_source();
if(!$_COOKIE['user']){
  $val_id="guest";
  $val_pw="123qwe";
  for($i=0;$i<20;$i++){
    $val_id=base64_encode($val_id);
    $val_pw=base64_encode($val_pw);
  }
  $val_id=str_replace("1","!",$val_id);
  $val_id=str_replace("2","@",$val_id);
  $val_id=str_replace("3","$",$val_id);
  $val_id=str_replace("4","^",$val_id);
  $val_id=str_replace("5","&",$val_id);
  $val_id=str_replace("6","*",$val_id);
  $val_id=str_replace("7","(",$val_id);
  $val_id=str_replace("8",")",$val_id);

  $val_pw=str_replace("1","!",$val_pw);
  $val_pw=str_replace("2","@",$val_pw);
  $val_pw=str_replace("3","$",$val_pw);
  $val_pw=str_replace("4","^",$val_pw);
  $val_pw=str_replace("5","&",$val_pw);
  $val_pw=str_replace("6","*",$val_pw);
  $val_pw=str_replace("7","(",$val_pw);
  $val_pw=str_replace("8",")",$val_pw);

  Setcookie("user",$val_id,time()+86400,"/challenge/web-06/");
  Setcookie("password",$val_pw,time()+86400,"/challenge/web-06/");
  echo("<meta http-equiv=refresh content=0>");
  exit;
}
?>
<html>
<head>
<title>Challenge 6</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
</style>
</head>
<body>
<?php
$decode_id=$_COOKIE['user'];
$decode_pw=$_COOKIE['password'];

$decode_id=str_replace("!","1",$decode_id);
$decode_id=str_replace("@","2",$decode_id);
$decode_id=str_replace("$","3",$decode_id);
$decode_id=str_replace("^","4",$decode_id);
$decode_id=str_replace("&","5",$decode_id);
$decode_id=str_replace("*","6",$decode_id);
$decode_id=str_replace("(","7",$decode_id);
$decode_id=str_replace(")","8",$decode_id);

$decode_pw=str_replace("!","1",$decode_pw);
$decode_pw=str_replace("@","2",$decode_pw);
$decode_pw=str_replace("$","3",$decode_pw);
$decode_pw=str_replace("^","4",$decode_pw);
$decode_pw=str_replace("&","5",$decode_pw);
$decode_pw=str_replace("*","6",$decode_pw);
$decode_pw=str_replace("(","7",$decode_pw);
$decode_pw=str_replace(")","8",$decode_pw);

for($i=0;$i<20;$i++){
  $decode_id=base64_decode($decode_id);
  $decode_pw=base64_decode($decode_pw);
}

echo("<hr><a href=./?view_source=1 style=color:yellow;>view-source</a><br><br>");
echo("ID : $decode_id<br>PW : $decode_pw<hr>");

if($decode_id=="admin" && $decode_pw=="nimda"){
  solve(6);
}
?>
</body>
</html>
```
소스 코드를 확인해보면 아이디 "guest"와 비밀번호 "123qwe"를 base64 방식으로 인코딩을 진행하고 있다. 인코딩을 총 20번 진행한다.<br />

```php
$val_id=str_replace("1","!",$val_id);
$val_id=str_replace("2","@",$val_id);
$val_id=str_replace("3","$",$val_id);
$val_id=str_replace("4","^",$val_id);
$val_id=str_replace("5","&",$val_id);
$val_id=str_replace("6","*",$val_id);
$val_id=str_replace("7","(",$val_id);
$val_id=str_replace("8",")",$val_id);
```
그 다음으로는 숫자들을 모두 특수 문자로 변환하는 해버리고 모든 인코딩, 변환이 끝난 문자열을 이제 Cookie 값으로 업데이트 해준다.<br />
각 쿠키의 키들은 user, password이다.<br />

이제 `<html>` 태그 안에 있는 php 부분을 보면 유저가 웹페이지에 접속했을 때 실행되는 코드라는 것을 알 수 있다.<br />
쿠키에서 user와 password를 읽어와서 아까 진행됐던 인코딩을 그대로 디코딩해주고있다.<br />
## 최종 풀이
다시 말해서 유저의 이름과 비밀번호가 쿠키에 인코딩 상태로 저장돼있고 해당 웹페이지는 그 쿠키의 값들을 해석하여 어떤 유저인지 판단을 한다.<br />

문제 페이지에 접속해서 쿠키를 살펴보면 코드에서 보았던 그대로 user와 password가 있고 각 key의 값이 알아보기는 어렵지만 대충 base64로 인코딩된 문자열처럼 보이는 값이 존재한다.<br />

php 코드의 맨 마지막 부분을 보면 디코딩 된 유저 이름과 비밀번호가 "admin", "nimda"라면 문제가 해결된다고 하니 두 문자열을 같은 방식으로 인코딩하여 쿠키 값을 조작해주면 된다.<br />
### 파이썬으로 인코딩
```python
import base64

id = "admin".encode("UTF-8")
pw = "nimda".encode("UTF-8")

for _ in range(0, 20):
    id = base64.b64encode(id)
    pw = base64.b64encode(pw)

print(id)
print()
print(pw)
```
파이썬을 이용하여 총 20번 인코딩을 진행하고 출력된 값들을 쿠키에 넣어주어 새로고침을 하면 문제가 해결된다. 이때 숫자들을 특수문자로 변환하는 것도 진행해야 하는데 까먹고 하지 못했다.<br />
그럼에도 불구하고 문제가 해결이 된 것을 보니 base64 인코딩 알고리즘의 지금 당장은 알 수 없는 어떠한 부분 때문인 것 같다.