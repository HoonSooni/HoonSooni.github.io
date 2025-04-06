---
title: "webhacking.kr old 16 문제 풀이"
date: 2025-04-06 00:01:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 16 문제 풀이
---

[https://webhacking.kr/challenge/js-3/](https://webhacking.kr/challenge/js-3/)
# 문제 풀이
웹페이지에 접속하면 크고 노란 별 모양(Asterisk 혹은 곱하기 기호)이 보인다.<br />
아무 키나 눌러보았을 때 크고 노란 별 말고 작은 별들이 일열로 쭉 추가되는 것이 관찰된다.<br />

```html
<script> 
	document.body.innerHTML+="<font color=yellow id=aa style=position:relative;left:0;top:0>*</font>";
	function mv(cd){
	  kk(star.style.left-50,star.style.top-50);
	  if(cd==100) star.style.left=parseInt(star.style.left+0,10)+50+"px";
	  if(cd==97) star.style.left=parseInt(star.style.left+0,10)-50+"px";
	  if(cd==119) star.style.top=parseInt(star.style.top+0,10)-50+"px";
	  if(cd==115) star.style.top=parseInt(star.style.top+0,10)+50+"px";
	  if(cd==124) location.href=String.fromCharCode(cd)+".php"; // do it!
	}
	function kk(x,y){
	  rndc=Math.floor(Math.random()*9000000);
	  document.body.innerHTML+="<font color=#"+rndc+" id=aa style=position:relative;left:"+x+";top:"+y+" onmouseover=this.innerHTML=''>*</font>";
	}
</script>
```
코드를 보면 mv()라는 함수가 있는데 해당 함수는 유저로부터 키보드 입력 값을 받아서 적절하게 해석하는 역할을 한다.<br />
'a', 'w', 's', 'd'를 입력 받으면 일반적인 FPS 게임에서의 입력처럼 유저를 상하 좌우로 움직일 수 있다.<br />

if문 중 맨 아래에 있는 '|'(124)를 입력하면 해당 웹서버의 "|.php"로 이동된다고 하고 주석으로 "do it!"이라고 격려하는 문구도 볼 수 있다.<br />

그러므로 문제 페이지에서 키보드의 '|'를 입력하면(백슬래시와 같이 있다) 문제가 해결된다. 