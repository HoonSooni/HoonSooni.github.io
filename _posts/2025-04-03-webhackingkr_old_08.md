---
title: "webhacking.kr old 08 문제 풀이"
date: 2025-04-03 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, php, sql injection] 
description: webhacking.kr old 08 문제 풀이
---

[https://webhacking.kr/challenge/web-08/](https://webhacking.kr/challenge/web-08/)
# 문제 풀이
## 웹사이트 분석
페이지에 접속하면 "hi guest"라는 문구와 함께 반겨준다.<br />
이것 이외에는 아무런 정보도 얻을 수가 없다. HTML 문서, 네트워크 탭, 쿠키 등을 살펴보아도 아무것도 존재하지 않는다.
## 코드 분석
```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Challenge 8</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
</style>
</head>
<body>
<br><br>
<center>
<?php
$agent=trim(getenv("HTTP_USER_AGENT"));
$ip=$_SERVER['REMOTE_ADDR'];
if(preg_match("/from/i",$agent)){
  echo("<br>Access Denied!<br><br>");
  echo(htmlspecialchars($agent));
  exit();
}
$db = dbconnect();
$count_ck = mysqli_fetch_array(mysqli_query($db,"select count(id) from chall8"));
if($count_ck[0] >= 70){ mysqli_query($db,"delete from chall8"); }

$result = mysqli_query($db,"select id from chall8 where agent='".addslashes($_SERVER['HTTP_USER_AGENT'])."'");
$ck = mysqli_fetch_array($result);

if($ck){
  echo "hi <b>".htmlentities($ck[0])."</b><p>";
  if($ck[0]=="admin"){
    mysqli_query($db,"delete from chall8");
    solve(8);
  }
}

if(!$ck){
  $q=mysqli_query($db,"insert into chall8(agent,ip,id) values('{$agent}','{$ip}','guest')") or die("query error");
  echo("<br><br>done!  ({$count_ck[0]}/70)");
}
?>
<a href=./?view_source=1>view-source</a>
</body>
</html>
```
### User-Agent
처음에 `agent`와 `ip` 변수를 초기화하고 있다.<br />
agent는 요청 헤더에 있는 User-Agent 값을 그대로 받아오고 ip 또한 마찬가지로 요청하는 클라이언트의 ip를 가진다.<br />
그 아래 부분에 User-Agent에 "from"이 들어있으면 접근을 금지하는 부분이 있다.<br />

![user-agent modification](https://1drv.ms/i/c/5cb37aa515b56a00/IQRyixqHCFTzQqb8rmW3DKZCAdFmP78UE6aaJ_U6vegFv_w?width=660)<br />
실제로 네트워크 탭을 이용해 "from"을 삽입하여 요청하면 접근이 막히는 것을 볼 수 있다. "from"이라는 단어를 필터링하는 것을 보면 SQL Injection 관련 문제인 것을 어느 정도 예상할 수 있다.<br />
### SQL Query
```php
$db = dbconnect();
$count_ck = mysqli_fetch_array(mysqli_query($db,"select count(id) from chall8"));
if($count_ck[0] >= 70){ mysqli_query($db,"delete from chall8"); }

$result = mysqli_query($db,"select id from chall8 where agent='".addslashes($_SERVER['HTTP_USER_AGENT'])."'");
$ck = mysqli_fetch_array($result);

if($ck){
  echo "hi <b>".htmlentities($ck[0])."</b><p>";
  if($ck[0]=="admin"){
    mysqli_query($db,"delete from chall8");
    solve(8);
  }
}

if(!$ck){
  $q=mysqli_query($db,"insert into chall8(agent,ip,id) values('{$agent}','{$ip}','guest')") or die("query error");
  echo("<br><br>done!  ({$count_ck[0]}/70)");
}
```
DB의 chall8이라는 테이블로부터 id 개수를 체크한다. 만일 개수가 70개 이상이라면 테이블의 모든 데이터가 날아간다.<br />

다음으로는 `select id from chall8 where agent={User-Agent}` 쿼리를 DB에 요청하여 값을 받아온다. 만일 성공적으로 받아왔다면 해당 값이 "admin"인지 확인하고 맞았을 때 문제가 풀리는 방식이다.<br />

값을 가져오는데 성공하지 못했다면 계속해서 새로운 정보가 테이블에 들어간다. 그러므로 우리는 시도 횟수가 약 69번으로 제한돼있다는 걸 알 수 있다.<br />
## 최종 풀이
헤더의 User-Agent 값을 조작하여 수행하는 SQL Injection 문제이다. <br />
### 페이로드 알아내기
첫 번째 테스트로 `* and id=admin` 값을 넣어보았다.<br />
`agent=* and id=admin`처럼 작동하는 것을 의도했지만 당연하게도 작동하지 않는다. <br />
결과를 살펴보면 "done! (16/70)" 이라고 출력되는 것을 보니 기회가 약 69번 있는 것이 아니라 53번 뿐이라고 한다. <br />
알고보니 DB에 데이터가 이미 15개가 들어있었다는 의미이다.<br />
### 우회하기 보단 쉬운 길로
계속해서 `addslashes()` 함수를 우회하는 방식으로 풀이를 고민하다가 그 아래의 쿼리에는 필터링을 거치지않는 다는 점이 신경쓰였다.<br />

가장 마지막에 이루어지는 쿼리인 insert 부분에서는 agent와 ip 값을 유저로부터 받아와서 "guest"라는 id로 테이블에 값을 넣어주고 있다.<br />
하지만 필터링을 하고있지 않기 때문에 얼마든지 우리가 원하는 값을 집어넣도록 쿼리를 변경할 수 있을 것 같다.<br />

```php
$q=mysqli_query($db,"insert into chall8(agent,ip,id) values('{$agent}','{$ip}','guest')") or die("query error");
```

만일 agent 값으로 `sampleAgent', '123.123.123.123', 'admin'), ('fakeAgent`를 넣어준다면 최종 쿼리는 아래처럼 해석될 것이다.

```php
"insert into chall8(agent,ip,id) values('sampleAgent', '123.123.123.123', 'admin'), ('fakeAgent','{$ip}','guest')"
```
이런식으로 우리는 sampleAgent라는 agent 값을 가지는 가짜 admin 유저를 DB 테이블에 추가하게 되는 것이다.<br />
### 익스플로잇
이제 위에서 생각해낸 페이로드를 Burp Suite같은 프록시 툴로 User-Agent 값에 덮어씌워 요청을 보내면 서버 DB에는 sampleAgent라는 agent 값을 가진 admin id가 하나 생성된다.<br />

이후에 다시 User-Agent 값을 sampleAgent로 지정하면 `mysqli_query($db,"select id from chall8 where agent='".addslashes($_SERVER['HTTP_USER_AGENT'])."'");` 이 쿼리에 의해서 `where agent=sampleAgent` 데이터를 불러온다.<br />

다시 말하지만 sampleAgent의 id 값은 admin이기 때문에 문제가 해결되게 된다.
# 배운 것
해당 문제에서 적용하지는 않았지만 `addslashes()` 함수를 우회하는 데에는 멀티바이트를 사용하는 언어로 인코딩을 하여 보내는 방법이 있다는 것을 알게됐다.