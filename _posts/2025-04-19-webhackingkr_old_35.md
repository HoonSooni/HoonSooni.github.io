---
title: "webhacking.kr old 35 문제 풀이"
date: 2025-04-19 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, sql injection]
description: webhacking.kr old 35 문제 풀이
---

[https://webhacking.kr/challenge/web-17/](https://webhacking.kr/challenge/web-17/)
# 문제 풀이
문제 페이지에 접속해보면 phone 값을 받는 인풋 박스와 제출 버튼이 있다.<br />

그 아래에는 소스코드를 볼 수 있는 페이지도 제공된다.<br />
## 소스 코드
```php
<?php
$db = dbconnect();
if($_GET['phone'] && $_GET['id']){
  if(preg_match("/\*|\/|=|select|-|#|;/i",$_GET['phone'])) exit("no hack");
  if(strlen($_GET['id']) > 5) exit("no hack");
  if(preg_match("/admin/i",$_GET['id'])) exit("you are not admin");
  mysqli_query($db,"insert into chall35(id,ip,phone) values('{$_GET['id']}','{$_SERVER['REMOTE_ADDR']}',{$_GET['phone']})") or die("query error");
  echo "Done<br>";
}

$isAdmin = mysqli_fetch_array(mysqli_query($db,"select ip from chall35 where id='admin' and ip='{$_SERVER['REMOTE_ADDR']}'"));
if($isAdmin['ip'] == $_SERVER['REMOTE_ADDR']){
  solve(35);
  mysqli_query($db,"delete from chall35");
}

$phone_list = mysqli_query($db,"select * from chall35 where ip='{$_SERVER['REMOTE_ADDR']}'");
echo "<!--\n";
while($r = mysqli_fetch_array($phone_list)){
  echo htmlentities($r['id'])." - ".$r['phone']."\n";
}
echo "-->\n";
?>
```
코드를 살펴보면 유저로부터 phone, id라는 파라미터 값을 GET 요청으로 받은 후에 chall35이라는 데이터베이스 테이블에 값을 넣어주고 있다.<br />
이때 유저의 ip주소 또한 같이 들어가고 있는 모습도 보인다.<br />

그리고 테이블에 추가된 데이터는 html의 주석으로 추가돼 우리가 볼 수 있게 돼있다.<br />
### 필터링
```php
if(preg_match("/\*|\/|=|select|-|#|;/i",$_GET['phone'])) exit("no hack");
if(strlen($_GET['id']) > 5) exit("no hack");
if(preg_match("/admin/i",$_GET['id'])) exit("you are not admin");
```
총 3가지의 필터링이 존재한다.<br />
1. 정규식
	- \*, /, =, -, #, ;, select 사용 불가
2. id 길이
	- id 파라미터의 길이가 5를 넘어서는 안됨
3. admin 문자열
	- id 파라미터의 값이 admin이면 안됨
## 최종 풀이
```php
$isAdmin = mysqli_fetch_array(mysqli_query($db,"select ip from chall35 where id='admin' and ip='{$_SERVER['REMOTE_ADDR']}'"));
if($isAdmin['ip'] == $_SERVER['REMOTE_ADDR']) {
	solve(35);
	mysqli_query($db,"delete from chall35");
}
```
문제 해결 조건을 보여주는 코드 스니펫이다.<br />
chall35 테이블에서 id가 admin인 데이터를 가져와서 ip가 현재 접속한 ip와 동일하다면 solve 함수가 호출되는 방식이다.<br />

즉, id가 admin이고 ip가 현재 나의 ip 값을 가지는 데이터를 테이블에 집어넣으면 문제가 해결된다는 말이다.<br />
### 필터링 우회
id가 admin인 값을 넣으려면 URL의 id 파라미터를 admin으로 설정해야하는데 아까 필터링 목록을 보았듯이 이 방법은 막혀있다.<br />
게다가 이 "admin" 문자열 필터링을 우회하려면 더 추가적인 특수문자들을 더해야하는데 id 값의 길이가 5를 넘어서는 안되는 규칙도 있어서 결국 id 값에 admin을 직접적으로 넣어주는 방법은 없어보인다.<br />
### phone을 이용한 SQLI
Insert SQL query를 보면 다행스럽게도 유저로부터 받은 phone 값을 가장 마지막에 넣어주고 있다. 게다가 phone 값에는 아무런 필터링이 걸려있지 않아서 자유롭게 값을 조작하는 것도 가능하다.<br />

```sql
-- original query
insert into chall35(id,ip,phone) values('{$_GET['id']}','{$_SERVER['REMOTE_ADDR']}',{$_GET['phone']})

-- modified query
insert into chall35(id,ip,phone) values('{$_GET['id']}','{$_SERVER['REMOTE_ADDR']}', '123), (admin, my_ip, 123')
```
이런식으로 `123), ('admin', 'my_ip', 123`을 phone 인풋 박스에 집어넣고 제출을 하게되면 'admin'을 id로 가지는 데이터가 하나 생성되면서 문제가 해결된다.