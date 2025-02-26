---
title: "드림핵 lv.1 - 삐리릭...삐리리릭..."
date: 2025-02-27 00:23:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking] 
description: 드림핵 삐리릭...삐리리릭... 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/1760](https://dreamhack.io/wargame/challenges/1760)
# 문제 설명
로봇은 싫어요! 다시 돌아가세요!
플래그 형식은 `0xH0P3{...}` 입니다.

---
# 문제 풀이
## 코드 분석
제공하는 파일을 다운로드 받으면 `no-download-need-lol.txt` 파일을 하나 받게되고 안의 내용물은 *"다운로드가 필요업어요! ^^"* 이다. 혹시 다운로드를 아직 안받았다면 받지 않아도 된다.
## 웹사이트 분석
```
Don't you want me like I want you, baby? Don't you need me like I need you now? Sleep tomorrow, but tonight go crazy All you gotta do is just meet me at the 아파트, 아파트
```
웹사이트에 접속하면 '아파트' 가사가 나오고 아무런 정보가 없다. 개발자 도구를 이용해서 알아낼 것이 있는지 살펴보자 <br />
### html code
```html 
<html>
	<head></head>
	<body>
		Don't you want me like I want you, baby?
		Don't you need me like I need you now?
		Sleep tomorrow, but tonight go crazy
		All you gotta do is just meet me at the
		아파트, 아파트
	</body>
</html>
```
아무것도 없다. <br />
경험상 이런식으로 아무런 정보가 없을 때에는 문제를 다시 한 번 읽어보는 것이 좋다.<br />

문제를 읽어보면 "로봇은 싫어요"라는 대사가 있고 문제의 타이틀은 "삐리릭...삐리리릭..."으로 무언가 전체적으로 "로봇"을 가리키는 느낌이 든다.

robots.txt에 접근해봐야 할 것 같다.
### /robots.txt
```
User-agent: * Disallow: /dreamhacksofun
```
접근해보면 이와같은 정보가 나온다. 뭔가 의심쩍으니까 /dreamhacksofun에도 접속해보자.
### /dreamhacksofun
접속해보면 /login으로 redirect되고 로그인 창이 뜨게된다. <br />
개발자 도구를 켜둔 체로 접속을 해보니까 HTML 코드에 숨겨진 아이디와 비번을 알게됐다.
```
<!--
    username: admin
    password: sksmssorkqlcsksmsqufdlswnfdkfdkTdjdy
--->
```

올바른 아이디와 비밀번호를 입력하면 다음 페이지로 넘어가고 플래그가 나온다.
# 배운 것
웹 해킹을 할 때에 때로는 robots.txt에서 중요한 정보를 얻을 수 있다는 것을 배웠다.