---
title: "webhacking.kr old 23 문제 풀이"
date: 2025-04-10 00:20:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 23 문제 풀이
---

[https://webhacking.kr/challenge/bonus-3/](https://webhacking.kr/challenge/bonus-3/)
# 문제 풀이
웹사이트에 접속하면 "Your mission is to inject `<script>alert(1);</script>`"라는 문구가 뜨고 그 위에 인풋 박스와 제출 버튼만 있는 간단한 구조를 볼 수 있다.<br />

우리의 목표는 방금 언급한 스크립트 태그를 inject하는 것이라고 한다.<br />

일단 그대로 인풋 박스에 넣어보면 "no hack"이라는 문구가 뜨며 막혀버린다.<br />
혹시 몰라 "alert(1);"를 넣어보니 또 같은 결과가 나오는 것이 확인된다.<br />

아무거나 막 넣어보던 중에 일종의 패턴을 알게됐다.<br />
영어 문자가 2개이상 반복되면 필터링에 걸린다는 것이다.<br />

즉, 'a'는 되지만 'aa'는 안된다는 것이다.<br />

우리가 inject 하고자 하는 스크립트에는 당연히 "script", "alert"와 같은 영어 문자들이 있기 때문에 필터링에 걸렸던 것이다.<br />

`"<s>"`를 입력해보면 진짜 HTML 코드로 변환돼 body에 추가되는 것이 확인된다.<br />
즉, 우리가 입력한 값이 텍스트 그 자체가 아니라 HTML 코드로 페이지에 추가된다는 것이다.<br />
## 최종 풀이
이제 필터링 조건도 알게 됐으니 이를 우회하는 일만 남았다.<br />
### 모든 문자를 URL 인코딩 형식으로 변환
```
%3C%73%63%72%69%70%74%3E%61%6C%65%72%74%28%31%29%3B%3C%2F%73%63%72%69%70%74%3E
```
혹시 영어 문자를 URL 인코딩에 사용되는 값들로 변환하면 먹힐까 했는데 여전히 필터링에 막힌다.<br />
### %09 사용
```
<s%09c%09r%09i%09p%09t>a%09l%09e%09r%09t(1);</s%09c%09r%09i%09p%09t>
```
이번에는 모든 영어 문자들 사이사이에 탭 문자를 의미하는 %09를 넣어주어 우회를 시도해봤지만 HTML 코드를 보면 `<s c="" r="" i="" p="" t="">a	l	e	r	t(1);</s>` 정말로 탭을 사이사이에 넣어진 체로 작동하여 이 방식도 틀렸다.<br />
### 자바스크립트 사용
```js
document.body.innerHTML += "<script>alert(1);</script>"
```
혹시 가능할까 싶어서 console을 이용하여 스크립트 태그를 body에 innerHTML로 넣어줘봤다.<br />
그랬더니 실제로 스크립트가 추가되긴 했는데 alert()가 실행되지는 않았다.<br />
해당 태그를 inject 하는 것 뿐만 아니라 alert()가 실제로 실행돼야 문제가 해결되는 듯 하다.<br />
### %00 사용
```
<s%00c%00r%00i%00p%00t>a%00l%00e%00r%00t(1);</s%00c%00r%00i%00p%00t>
```
이번엔 %09 대신에 null을 의미하는 %00을 중간중간에 넣어주니 이제 제대로 alert 창이 실행되며 문제가 해결됐다.<br />
여기서 신경써야 할 점은 해당 값을 인풋 박스에 넣는 것이 아니라 URL의 code 파라미터로 전달해야한다.<br />
'%'가 "%25"로 인코딩돼서 double encoding 문제가 발생하기 때문이다.<br />
# 배운 것
%00 null 값은 HTML에서 무시된다는 것을 알게됐다.