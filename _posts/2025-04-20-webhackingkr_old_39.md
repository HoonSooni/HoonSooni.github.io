---
title: "webhacking.kr old 39 문제 풀이"
date: 2025-04-20 10:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, sql injection]
description: webhacking.kr old 39 문제 풀이
---

[https://webhacking.kr/challenge/bonus-10/](https://webhacking.kr/challenge/bonus-10/)
# 문제 풀이
문제 페이지에 접속해보면 아무런 문구도 없이 단순 input box와 쿼리 전송 버튼이 있다.<br />
그 아래에는 다행히도 소스코드를 살펴볼 수 있는 페이지도 제공한다.<br />

```php
<?php
  include "../../config.php";
  if($_GET['view_source']) view_source();
?><html>
<head>
<title>Chellenge 39</title>
</head>
<body>
<?php
  $db = dbconnect();
  if($_POST['id']){
    $_POST['id'] = str_replace("\\","",$_POST['id']);
    $_POST['id'] = str_replace("'","''",$_POST['id']);
    $_POST['id'] = substr($_POST['id'],0,15);
    $result = mysqli_fetch_array(mysqli_query($db,"select 1 from member where length(id)<14 and id='{$_POST['id']}"));
    if($result[0] == 1){
      solve(39);
    }
  }
?>
```
php 코드를 살펴보면 유저로부터 id를 받고 `\\`라는 문자를 받으면 없애버리고 ' 작은 따옴표 하나를 받으면 2개로 만들어버리는 과정을 거친다.<br />
그리고 id의 값이 15를 넘기지 않도록 제한을 하고있다.<br />

HTML에서도 input 태그의 maxlength 속성을 15로 설정해놨고 쿼리 또한 id의 길이가 14보다 작은 값을 있는 것이 확인된다.<br />
## 최종 풀이
```php
mysqli_fetch_array(mysqli_query($db,"select 1 from member where length(id)<14 and id='{$_POST['id']}"));
```
쿼리를 자세하게 살펴보면 id 인풋 값을 넣어주는 부분에서 작은 따옴표를 닫아주지 않았다. `id='{$_POST['id']}`<br />

그래서 이를 '를 입력하여 닫아주어야 하는데 작은 따옴표를 입력하면 자동으로 2개로 변환하는 과정이 있어서 이 또한 불가능하다.<br />

이번 문제는 SQLI 문제라기 보다는 코드 로직에 버그가 발생할만한 부분을 찾는 것이 관건이다.<br />
### 길이 제한에서 발생하는 에러
```php
$_POST['id'] = str_replace("\\","",$_POST['id']);
$_POST['id'] = str_replace("'","''",$_POST['id']);
$_POST['id'] = substr($_POST['id'],0,15);
```
이 부분을 자세히 살펴보자.<br />
유저로부터 입력 값을 받고 replacement 과정을 거친 다음에 해당 문자열의 길이를 15로 제한을 두고 딱 잘라버린다.<br />

즉, '를 입력해서 2개로 늘어났다면 그 늘어난 상태의 문자열의 길이를 기준으로 15까지만 제한한다는 것이다.<br />

이를 이용하여 딱 15번째 자리에 작은 따옴표를 넣어주면 ''처럼 2개가 되지만 `substr()` 함수에 의해서 따옴표는 딱 하나만 남게된다.<br />

그렇다는 것은 `"              '"` 이런식으로 작성을 해주면 쿼리 문에 빠져버린 닫는 역할의 작은 따옴표를 우리가 임의로 추가해주어 해당 쿼리문이 참이 되게끔 할 수 있는 것이다.<br />

처음에는 id 값을 admin이든 guest이든 때려 맞추어야하나 고민해봤지만 아무런 값도 넣지않고 스페이스 문자로 채워도 해결되는 것을 볼 수 있다.