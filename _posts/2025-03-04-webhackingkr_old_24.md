---
title: "webhacking.kr old 24 문제 풀이"
date: 2025-03-04 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 24 문제 풀이
---

# 살펴보기

| client ip | 000.000.000.000                                                                      |
| --------- | ------------------------------------------------------------------------------------ |
| agent     | Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:135.0) Gecko/20100101 Firefox/135.0 |

사이트에 접근을 해보면 간단하게 테이블로만 만들어진 HTML 페이지가 보인다. <br />
잘 보면 나의 접속 ip와 agent가 출력된다.

그 바로 아래 view-source href를 통해서 코드를 볼 수 있으니 들어가본다.
## view-source
```php
<?php
  extract($_SERVER);
  extract($_COOKIE);
  $ip = $REMOTE_ADDR;
  $agent = $HTTP_USER_AGENT;
  if($REMOTE_ADDR){
    $ip = htmlspecialchars($REMOTE_ADDR);
    $ip = str_replace("..",".",$ip);
    $ip = str_replace("12","",$ip);
    $ip = str_replace("7.","",$ip);
    $ip = str_replace("0.","",$ip);
  }
  if($HTTP_USER_AGENT){
    $agent=htmlspecialchars($HTTP_USER_AGENT);
  }
  echo "<table border=1><tr><td>client ip</td><td>{$ip}</td></tr><tr><td>agent</td><td>{$agent}</td></tr></table>";
  if($ip=="127.0.0.1"){
    solve(24);
    exit();
  }
  else{
    echo "<hr><center>Wrong IP!</center>";
  }
?>
```
(풀이에 필요한 부분만 가져왔다.)<br />

코드를 살펴보면 `$REMOTE_ADDR` 변수를 통해서 나의 ip가 127.0.0.1인지 체크하고 있다.<br />
## 최종 풀이
코드 맨 위의 `extract($_SERVER); extract($_COOKIE);` 부분이 보인다.<br />
`extract($_SERVER)`로 인해서 `REMOTE_ADDR`이라는 변수가 나의 현재 ip로 초기화되지만 그 다음에 나오는 `extract($_COOKIE)`로 덮어 쓸 수 있다.<br />

즉, `name`을 `REMOTE_ADDR`로 가지는 쿠키를 하나 만들어서 접속하면 해결된다.

```php
if($REMOTE_ADDR){
  $ip = htmlspecialchars($REMOTE_ADDR);
  $ip = str_replace("..",".",$ip);
  $ip = str_replace("12","",$ip);
  $ip = str_replace("7.","",$ip);
  $ip = str_replace("0.","",$ip);
}
```
여기서 해결해야하는 또 다른 문제는 ip 값을 필터링하고 있다는 것이다.<br />
`..`을 `.`로, `12`를 `공백`으로, `7.`을 `공백`으로, 그리고 `0`을 `공백`으로 변환한다.<br />

이를 회피하는 쿠키 값은 `REMOTE_ADDR: 112277...00...00...1`이다. <br />
이를 쿠키에 넣고 새로 고침을 하면 해결이 되는 문제이다.