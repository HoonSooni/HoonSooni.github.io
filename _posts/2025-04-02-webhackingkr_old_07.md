---
title: "webhacking.kr old 07 문제 풀이"
date: 2025-04-02 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, php, sql injection] 
description: webhacking.kr old 07 문제 풀이
---

[https://webhacking.kr/challenge/web-07/index.php?val=1](https://webhacking.kr/challenge/web-07/index.php?val=1)
# 문제 풀이
## 웹사이트 분석
문제 페이지에 접속하면 "Admin Page"라는 문구와 "auth"라고 적힌 버튼이 있고 소스코드를 볼 수 있는 페이지로 넘어가는 링크가 있다.<br />

"auth" 버튼을 눌러보면 Access Denied라고 적힌 alert 창이 뜨게 된다.<br />
URL을 살펴보면 ?val이라는 파라미터를 받는 것 같고 기본 값은 1인것 같다.
## 코드 분석
```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Challenge 7</title>
</head>
<body>
<?php
$go=$_GET['val'];
if(!$go) { echo("<meta http-equiv=refresh content=0;url=index.php?val=1>"); }
echo("<html><head><title>admin page</title></head><body bgcolor='black'><font size=2 color=gray><b><h3>Admin page</h3></b><p>");
if(preg_match("/2|-|\+|from|_|=|\\s|\*|\//i",$go)) exit("Access Denied!");
$db = dbconnect();
$rand=rand(1,5);
if($rand==1){
  $result=mysqli_query($db,"select lv from chall7 where lv=($go)") or die("nice try!");
}
if($rand==2){
  $result=mysqli_query($db,"select lv from chall7 where lv=(($go))") or die("nice try!");
}
if($rand==3){
  $result=mysqli_query($db,"select lv from chall7 where lv=((($go)))") or die("nice try!");
}
if($rand==4){
  $result=mysqli_query($db,"select lv from chall7 where lv=(((($go))))") or die("nice try!");
}
if($rand==5){
  $result=mysqli_query($db,"select lv from chall7 where lv=((((($go)))))") or die("nice try!");
}
$data=mysqli_fetch_array($result);
if(!$data[0]) { echo("query error"); exit(); }
if($data[0]==1){
  echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=\"alert('Access_Denied!')\"><p>");
}
elseif($data[0]==2){
  echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=\"alert('Hello admin')\"><p>");
  solve(7);
}
?>
<a href=./?view_source=1>view-source</a>
</body>
</html>
```
코드를 살펴보면 해당 문제는 SQL 인젝션을 이용한 문제란 것을 알 수 있다.<br />

유저가 URL에 입력한 val 파라미터 값을 이용해 쿼리를 하고있는데 이 전에 필터링을 거친다.<br />
### 필터링 분석
```php
preg_match("/2|-|\+|from|_|=|\\s|\*|\//i",$go)
```
- `2`
- `-`
- `+`
- `from`
- `_`
- `=`
- any whitespace (`\s`)
- `*`
- `/`

필터링을 잘 살펴보았을 때 어떤 문자들이 막히는지 위에 정리했다.<br />
숫자 2, 사칙연산 기호, from, 스페이스 등이 있다.
### 페이로드 분석
```php
elseif($data[0]==2){
  echo("<input type=button style=border:0;bgcolor='gray' value='auth' onclick=\"alert('Hello admin')\"><p>");
  solve(7);
}
```
코드 아래를 보면 쿼리의 결과가 어떤 값이어야 하는지 나온다.<br />
쿼리의 맨 첫 번째 결과가 2라면 문제가 해결된다. 하지만 지금은 문자 2 자체도 필터링 돼있고 1+1 같은 사칙연산도 불가능해서 다른 방안이 필요하다.
### 익스플로잇
생각해보면 기본 사칙연산 기호 말고 컴퓨터에는 모듈러 연산이 있다. `%`은 필터링 목록에 존재하지 않는다.<br />
예를 들어 `5%3`을 하게되면 결과가 2로 나오니 이 부분은 해결이 된다.<br />

해당 문제에서 쿼리를 넘겨주는 곳이 URL이기 때문에 '%'은 인코딩 문자로 인식돼 '%25'라고 입력해야 작동한다. 최종적으로 '5%253'을 넘겨주면 되는 것이다.<br />
하지만 `?val=5%253` 이렇게 입력해주면 query error가 뜨면서 아무런 데이터를 반환받지 못하는 상황이 생긴다.<br />

아마도 쿼리에 있는 () 괄호 때문인 것 같기도 한데 그렇다면 이를 우회해야 한다.<br />

이를 우회하기 위해서 우선 더미 값으로 아무거나 입력하고 그 뒤에 괄호를 닫아주어 우리가 원하는 쿼리를 심게끔 해야한다.<br />
`14455)`와 같이 입력하고 그 뒤에는 이제 괄호가 풀린 상태이니 맘대로 입력할 수 있다.<br />

이미 where 구문은 `(14455)`로 사용했으니 이제 남은건 `union`이다.<br />
`union(select(5%253)` 이렇게 입력하면 2라는 숫자가 select 되면서 그 앞의 select 문과 합쳐지게 된다. 그러므로 최종적으로 전체 select 문의 결과가 2가 되면서 문제가 해결되게 된다.<br />

`?val=14555)union(select(5%253)`<br />
여기서 중요한 점은 코드를 보았을 때 랜덤으로 1 ~ 5 숫자를 뽑아서 괄호의 개수를 정하고 있는데 우리가 원하는 괄호의 개수는 1개씩이다. 그래야 우리가 원하는 괄호 탈출 + 괄호 닫기가 가능해진다.<br />

해당 페이로드를 입력해놓고 문제가 해결됐다는 alert 창이 뜰 때까지 새로고침을 하면 문제가 풀린다.
# 배운 것
오랜만에 SQL 인젝션 문제이다.<br />
인젝션 관련해서 많은 훈련이 필요하다는 것을 느낄 수 있는 문제였다.