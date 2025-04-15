---
title: "webhacking.kr old 32 문제 풀이"
date: 2025-04-16 08:06:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking]
description: webhacking.kr old 32 문제 풀이
---

[https://webhacking.kr/challenge/code-5/](https://webhacking.kr/challenge/code-5/)
# 문제 풀이
```
|3|gabe0529|43 / 100|
|4|webhack00|36 / 100|
|5|gsrth|33 / 100|
|6|prup|30 / 100|
|7|ynokim|28 / 100|
|8|simub98|27 / 100|
|9|shionista|26 / 100|
|10|qwer1101|26 / 100|
|11|shiny109|22 / 100|
|12|smj100394|19 / 100|
|13|juhobest|19 / 100|
|14|wmqkq123|17 / 100|
|15|geunyeong|17 / 100|
|16|1234ooo|17 / 100|
```
페이지에 접속하면 2200명이 넘는 어떠한 목록(유저로 보이는)이 쭉 나열돼있고 각 목록마다 오른쪽에 Hit 값이 있다. 이 값은 아마 100이 최대값인지 `num / 100` 형식으로 돼있다.<br />
내가 유저를 한 번 클릭할 때 ?hit=username으로 redirect 되는 것 같고 두 번째 클릭해보면 이미 투표했다는 알림창과 함께 막혀버린다.<br />

쿠키를 살펴보니 `vote_check`이라는 쿠키 key에 ok라는 value가 있는 것이 보인다.<br />
이 값을 다른 것으로 바꿔보아도 아마 투표했다는 사실은 변하지 않았다. 아마도 해당 쿠키가 존재하느냐를 판별하여 작동하는 듯 하다.<br />

쿠키를 지워보고 시도해보면 정상적으로 재투표가 가능하다.<br />
## 시도
### 한 유저 투표 수 100 넘기기
투표 수 1등을 보면 45표로 100표까지 얼마 안남은 것이 보였다.<br />
그래서 한 번 이걸 100까지 찍어보아야겠다 싶어서 아래와 같은 파이썬 코드를 작성했다.

```python
import requests

url = "https://webhacking.kr/challenge/code-5?hit=REDACTED"

cookie = {
    "PHPSESSID" : "REDACTED"
}

for i in range(1, 51):
    resp = requests.get(url, cookies=cookie)

print(resp.text)
```
쿠키를 나의 PHP 세션 아이디를 제외한 나머지를 제거한 상태로 계속해서 "?hit={a user}"로 GET 요청을 보내는 코드이다.<br />
대충 50번만 요청을 보내고 나머지는 내가 직접 브라우저에서 채워보았다.<br />
하지만 아무 일도 일어나지 않았다.<br />
## 최종 풀이
어떻게 해야하나 고민을 하던 중에 화면 맨 아래쪽에 나의 ID를 발견했다.<br />
이 유저 목록은 해당 문제 페이지를 접속한 순서를 나타내는 것이었다.<br />
나의 투표 수는 당연히 0 / 100 이었고 파이썬 코드의 URL 부분을 "?hit=내 아이디"로 변경하고 총 99번 요청을 보내니 순식간에 내 아이디가 목록에서 상위권으로 올라왔다.<br />

이제 브라우저를 통해서 투표를 한 번 해주어 투표 수를 100 / 100로 만들어 버리니 문제가 풀렸다는 메세지와 함께 포인트가 주어졌다.