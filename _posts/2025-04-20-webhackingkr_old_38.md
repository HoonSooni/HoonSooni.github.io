---
title: "webhacking.kr old 38 문제 풀이"
date: 2025-04-20 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking, crlf injection]
description: webhacking.kr old 38 문제 풀이
---

[https://webhacking.kr/challenge/bonus-9/index.php](https://webhacking.kr/challenge/bonus-9/index.php)
# 문제 풀이
페이지에 접속하면 "LOG INJECTION"이라는 문구와 함께 "id" 값을 제출할 수 있는 form이 하나 있다.<br />

아무거나 입력해보면 아무런 변화가 없고 "admin"이라고 입력을 시도하면 "you are not admin"이라는 문구가 출력되면서 무언가 막혀버린다.<br />

HTML을 살펴보면 .admin.php 페이지로 이동되는 a 태그가 주석으로 숨겨져있는 것이 확인된다.<br />
## admin.php
```
You must logged as "admin"

175.194.237.103:admin]  
175.194.237.103:"admin"  
175.194.237.103:97100109105110  
175.194.237.103:"admin"  
175.194.237.103:175.194.237.103:"admin"  
175.194.237.103:175.194.237.103:admin  
my_ip:guest  
my_ip:admin1  
my_ip:adf  
my_ip:zcv
```
해당 페이지에 접속을 해보면 로그인 기록 로그가 출력된다.<br />
그리고 맨 위에 보면 "You must logged as admin"이라고 적혀있다. 문법이 좀 잘못됐지만 대충 뜻을 예상해보자면 admin으로 로그를 남겼어야지 하는 것 같다.<br />

아래 쪽의 4줄이 내가 시도해본 로그 기록인데 그 위에 이미 남아있는 로그들이 보인다.<br />
아마도 내 ip로 admin이라고 적힌 로그가 남아야 문제가 해결되는 듯 예상된다.<br />

`175.194.237.103:175.194.237.103:"admin"` 로그들 중 굉장히 수사한 녀석이 보인다.<br />
아무것도 입력하지 않은체로 그 다음 로그로 넘어간 부분이다.<br />

모든 로그들은 다 개행 문자로 구분이 돼 있는데 왜 이 부분은 그렇지 않을까 생각해보면 아마 문제 제출자가 CRLF 취약점을 이용해서 풀라고 힌트를 제공하는 듯 하다.<br />
## 최종 풀이
이 문제를 푸는데에 각종 단서들을 얻었다.<br />
1. admin 문자열 필터링
	- 아까도 시도해보았듯이 "admin"이라는 문자열을 입력하면 필터링에 의해 막혀버린다. 하지만 admin에 다른 문자열이 같이 붙어있으면 걸리지 않는다.
2. 로그 구분 방식
	- 각 로그들은 개행 문자 (`\r\n`)으로 구분돼있다.
3. admin 로그인 기록
	- 아마 로그 중 내 ip와 admin으로 로그인한 기록이 남아야 문제가 해결되는 듯 하다.<br />

이러한 단서들을 조합하여 결론적으로 우리는 `random_string\r\nmy_ip:admin`으로 입력 해볼 수 있다는 결론에 도달할 수 있다.<br />
아무런 문자열이나 넘겨주어 로그에 `my_ip:random_string`이 남게끔 하고 그 뒤 \r\n으로 로그를 구분한 다음에 직접 `my_ip:admin`을 입력해주어 결국 로그에는 `my_ip:admin`이 남게돼 문제가 해결됨을 기대할 수 있다.<br />

실제로 직접 시도해보면 문제가 해결된다.<br />

참고로 일단 input 박스로 직접 \r\n을 입력해주면 특수 문자 처리가 되지 않고 문자열 그대로 입력되기 때문에 textarea 태그로 변환하여 직접 개행을 해주어야 한다.<br />
아니면 Burp Suite 같은 툴을 이용해서 전달해주어도 괜찮다.<br />