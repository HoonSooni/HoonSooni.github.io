---
title: "webhacking.kr old 18 문제 풀이"
date: 2025-04-07 00:09:20 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, sql injection] 
description: webhacking.kr old 18 문제 풀이
---

[https://webhacking.kr/challenge/web-32/](https://webhacking.kr/challenge/web-32/)
# 문제 풀이
사이트에 들어가면 대놓고 SQLI 문제라고 소개하고있다.<br />
그리고 인풋 박스 하나가 있고 제출 버튼 존재한다.<br />

대충 아무 값이나 입력해보니 GET 방식으로 전달되는 것이 보인다.<br />
no이라는 파라미터 값으로 넘어가는 것을 보면 어떠한 숫자를 받는 듯 하다.<br />

1을 입력해보면 "hi guest"라는 문구가 나오고 그 이외의 숫자를 넣으면 아무것도 출력되지 않는다.<br />

해당 문제는 10 포인트짜리 문제인 만큼 소스코드를 제공한다.<br />
```php
<?php
if($_GET['no']){
  $db = dbconnect();
  if(preg_match("/ |\/|\(|\)|\||&|select|from|0x/i",$_GET['no'])) exit("no hack");
  $result = mysqli_fetch_array(mysqli_query($db,"select id from chall18 where id='guest' and no=$_GET[no]")); // admin's no = 2

  if($result['id']=="guest") echo "hi guest";
  if($result['id']=="admin"){
    solve(18);
    echo "hi admin!";
  }
}
?>
```
유저로부터 no이라는 파라미터 값을 받아와서 한 번 필터링을 거친 다음 어떠한 쿼리를 실행한 후 쿼리의 결과가 "admin"이라면 문제가 해결되는 방식이다.<br />

필터링에 걸리는 문자를 나열해보았다.<br />
* space
* /
* ( )
* |
* &
* select
* from
* 0x (hex)

```sql
select id from chall18 where id='guest' and no=$_GET[no]
```
그리고 이 부분이 바로 우리가 조작해야하는 SQL 쿼리문이다.<br />
chall18이라는 테이블에서 "guest"라는 id 값을 가진 데이터를 찾고있고 no 값을 받아서 한 번 더 validation을 주고 있다.<br />

코드에 주석으로 admin의 no은 2라고 소개하는데 이미 앞에서 `id="guest"`를 체크하고 있기 때문에 2를 전달해도 먹히지 않는다.<br />
## 최종 풀이
우리는 SQLI에서 자주 쓰이는 기법을 이용해야한다.<br />
앞의 조건식을 거짓으로 만들어 새로 추가되는 조건식을 참으로 만드는 방식이다.<br />

```
?no=99%09or%09id='admin'
```
이런식으로 값을 넘겨주면 된다.<br />

두 조건식이 `and`로 묶여있기 때문에 99를 넘겨주어 `where id='guest' and no=99`가 거짓이 되게 한 후 `or id='admin'`을 추가해주어 뒤의 조건식이 먹히도록 해주는 기법이다.<br />

여기서 중요한 점은 space를 사용할 수 없어서 URL에서 탭을 의미하는 %09를 중간 중간 넣어주는 모습이다.<br />
GET으로 요청을 받고있기 때문에 인풋 박스 대신 URL에 입력해주어야 한다.<br />
인풋 박스에 넣으면 '%'자체가 또 다시 %25로 인코딩되기 때문에 %09가 %2509로 돼버려 작동하지 않는다.