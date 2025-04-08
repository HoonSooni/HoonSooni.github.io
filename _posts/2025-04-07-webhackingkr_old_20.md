---
title: "webhacking.kr old 20 문제 풀이"
date: 2025-04-07 00:09:55 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 20 문제 풀이
---

[https://webhacking.kr/challenge/code-4/](https://webhacking.kr/challenge/code-4/)
# 문제 풀이
웹사이트에 접속해보면 총 3개의 인풋 박스가 유저를 기다리고있다.<br />
nickname, comment, captcha라는 이름의 인풋을 받고있고 captcha는 박스 오른쪽의 새로고침 할 때마다 초기화되는 영어 소문자, 대문자, 숫자 조합으로 이루어진 랜덤 문자열을 입력해야한다.<br />

아무 값이나 넣어주고 캡챠는 정확하게 입력을 한 후 제출을 해보면 "Too Slow"라는 문구가 뜬다.<br />
알고보니 메인 페이지 가운데 위쪽에 time limit이 2초라고 한다. 즉, 새로고침을 하고 2초만에 모든 입력값을 입력하여 제출하라는 의미이다.<br />
## 최종 풀이
당연하게도 사람의 힘으로는 2초만에 입력할 수 없다.<br />
그래서 자바스크립트 코드를 이용해야 한다.<br />

```js
(function() {
	var id = document.getElementsByName("id")[0];
	var comment = document.getElementsByName("cmt")[0];
	var captcha = document.getElementsByName("captcha")[0];
	
	var captcha_answer = document.getElementsByName("captcha_")[0].value;

	id.value = "random_id";
	comment.value = "random_comment";
	captcha.value = captcha_answer;

	var form = document.getElementsByName("lv5frm")[0];
	form.submit();
})()
```
HTML 코드를 참고하여 각 인풋 박스들의 name을 알아낸 후 DOM으로 선택했다.<br />
그리고 각 인풋들에 값을 적절하게 넣어주고 form까지 선택한 후 submit() 함수를 호출하는 코드이다.<br />

괄호로 묶어두어 입력하자마자 실행되도록 해뒀고 console 창을 켜놓은 상태에서 새로고침을 하자마자 바로 해당 코드를 console에 붙여넣어주면 문제가 해결된다.