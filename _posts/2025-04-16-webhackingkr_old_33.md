---
title: "webhacking.kr old 33 문제 풀이"
date: 2025-04-16 22:55:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking]
description: webhacking.kr old 33 문제 풀이
---

[https://webhacking.kr/challenge/bonus-6/](https://webhacking.kr/challenge/bonus-6/)
# 문제 풀이
## 33-1
```php
Challenge 33-1

<a href=index.txt>view-source</a>
```
처음에 접속하면 이러한 문구가 뜨고 소스코드 페이지로 이동하는 a 태그가 있다.<br />

```php
<?php
	if($_GET['get']=="hehe") echo "<a href=???>Next</a>";
	else echo("Wrong");
?>
```
?get=hehe로 GET 요청을 보내면 다음 단계로 넘어가는 a 태그가 출력된다고 한다.<br />
## 33-2
```php
<?php
	if($_POST['post']=="hehe" && $_POST['post2']=="hehe2") echo "<a href=???>Next</a>";
	else echo "Wrong";
?>
```
이번에는 GET이 아니라 POST로 요청을 받고 'post', 'post2' 값이 각각 'hehe', 'hehe2'일 때 다음으로 가는 링크가 출력되는 듯 하다.<br />
일반 브라우저로 POST 요청을 보내는 방법은 있는지 모르겠고 찾아보기도 귀찮아서 Burp Suite를 이용해서 요청을 조작했다.<br />
## 33-3
```php
<?php
	if($_GET['myip'] == $_SERVER['REMOTE_ADDR']) echo "<a href=???>Next</a>";
	else echo "Wrong";
?>
```
이번에는 GET으로 받은 myip 값이 server의 remote_addr 값과 동일한지 체크하고있다.<br />
나의 ip 주소를 myip로 넘겨주기만 하면 다음 링크가 나온다.<br />
## 33-4
```php
<?php
	if($_GET['password'] == md5(time())) echo "<a href=???>Next</a>";
	else echo "hint : ".time();
?>

hint : 1744809213
```
이번에는 password 파라미터 값을 GET 요청으로 받아서 현재 시간값을 md5로 인코딩한 것과 비교를 한다.<br />
화면에 보면 hint로 현재 시간을 친절하게 띄워준다. 새로고침을 해보면 1초당 1씩 값이 증가하는 것이 보인다.<br />

미리 30초정도 더해진 값을 Md5 인코더로 변환한 후 ?password=md5_value를 입력해주고 계속해서 새로고침을 해주면 30초가 지났을 때 다음으로 넘어가는 링크가 뜬다.<br />
## 33-5
```php
<?php
	if($_GET['imget'] && $_POST['impost'] && $_COOKIE['imcookie']) echo "<a href=???>Next</a>";
	else echo "Wrong";
?>
```
이것도 Burp Suite를 이용해서 해결하면 된다.<br />
URL 맨 뒤에 "imget" 파라미터를 추가하고 POST로 요청을 보내면서 "impost"라는 이름을 가진 값을 보내면서 "imcookie"라는 쿠키를 설정해주면 된다.<br />

값은 확인하지 않고 존재 여부만 따지기 때문에 아무 더미 값이나 넣어주어도 무방하다.<br />
## 33-6
```php
<?php
	if($_COOKIE['test'] == md5($_SERVER['REMOTE_ADDR']) && $_POST['kk'] == md5($_SERVER['HTTP_USER_AGENT'])) echo "<a href=???>Next</a>";
	else echo "hint : {$_SERVER['HTTP_USER_AGENT']}";
?>
```
이번에는 쿠키에 "test"라는 값이 나의 ip 주소를 md5로 인코딩한 값과 동일한지, 그리고 POST 바디에 "kk"라는 값이 나의 `HTTP_USER_AGENT` 값을 md5로 인코딩한 것과 동일한지 체크한다.<br />

이번에도 별 달리 어려운 것은 없고 Burp Suite를 이용해서 해결했다.<br />
## 33-7
```php
<?php
	$_SERVER['REMOTE_ADDR'] = str_replace(".","",$_SERVER['REMOTE_ADDR']);
	if($_GET[$_SERVER['REMOTE_ADDR']] == $_SERVER['REMOTE_ADDR']) echo "<a href=???>Next</a>";
	else echo "Wrong<br>".$_GET[$_SERVER['REMOTE_ADDR']];
?>
```
나의 ip 주소에서 .을 모두 제거한 값을 파라미터 이름으로 하고 값도 동일하게 설정했을 경우에 통과하는 로직이다.<br />
## 33-8
```php
<?php
	extract($_GET);
	if(!$_GET['addr']) $addr = $_SERVER['REMOTE_ADDR'];
	if($addr == "127.0.0.1") echo "<a href=???>Next</a>";
	else echo "Wrong";
?>
```
php의 `extract()` 함수는 배열 속의 키들을 변수화해준다.<br />
예를 들어` $_GET` 내부에  addr이라는 키가 있다면 `extract()` 함수를 실행하고 부터는 `$addr` 이라는 변수가 `$_GET`에서 빠져나와 단독적으로 사용이 가능하다는 것이다.<br />

즉, 내가 ?addr=127.0.0.1을 입력해 GET 요청을 보내면 $addr 변수가 하나 생성돼 값이 127.0.0.1로 초기화된다는 것이다.<br />
## 33-9
```php
<?php
	for($i=97;$i<=122;$i=$i+2){
		$answer.=chr($i);
	}
	if($_GET['ans'] == $answer) echo "<a href=???.php>Next</a>";
	else echo "Wrong";
?>
```
코드 내에 보이는 for문이 어떤 문자열을 도출하는지 잘 살펴보고 그대로 ans 파라미터로 전달해주면 된다.<br />
## 33-10
```php
<?php
	$ip = $_SERVER['REMOTE_ADDR'];
	for($i=0;$i<=strlen($ip);$i++) $ip=str_replace($i,ord($i),$ip);
	$ip=str_replace(".","",$ip);
	$ip=substr($ip,0,10);
	$answer = $ip*2;
	$answer = $ip/2;
	$answer = str_replace(".","",$answer);
	$f=fopen("answerip/{$answer}_{$ip}.php","w");
	fwrite($f,"<?php include \"../../../config.php\"; solve(33); unlink(__FILE__); ?>");
	fclose($f);
?>
```
이번에는 코드가 조금 복잡한데 하나 하나씩 차근차근 살펴보면서 어떤 값이 나올지 생각해보면 어렵지 않다.<br />
최종적으로 연산이 끝난 `$answer`과 `$ip`가 합쳐진 것이 php 파일 이름이고 문제 기본 페이지에서 `/answerip/result.php`에 접속하면 문제가 풀린다고 한다.

```js
let ip = "ip_address";

// Replace each digit (0–9) with its ASCII code
for (let i = 0; i <= ip.length; i++) {
  ip = ip.replaceAll(i.toString(), i.toString().charCodeAt(0));
}

// Remove dots
ip = ip.replaceAll(".", "");

// Take first 10 characters
ip = ip.substring(0, 10);

// Do math
let answer = parseFloat(ip) / 2;

// Remove decimal point if any
answer = answer.toString().replace(".", "");

console.log(`answer: ${answer}`);
console.log(`ip: ${ip}`);
console.log(`filename: ${answer}_${ip}.php`); 
```
참고로 필자가 사용한 코드는 이렇하다.<br />
문제에서 제공하는 php 코드를 그대로 JS로 옮긴 것이다.<br />