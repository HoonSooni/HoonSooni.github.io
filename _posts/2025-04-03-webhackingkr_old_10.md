---
title: "webhacking.kr old 10 문제 풀이"
date: 2025-04-03 00:10:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 10 문제 풀이
---

[https://webhacking.kr/challenge/code-1/](https://webhacking.kr/challenge/code-1/)
# 문제 풀이
이번 문제는 너무나도 쉬운 문제이다.<br />

페이지에 들어가면 어떠한 트랙을 연상시키는 듯한 그래픽이 보이고 왼쪽에 'O'라는 문자가 있다. <br />
해당 문자에 마우스를 올리면 'yOu'처럼 변하고 클릭을 하면 할수록 조금씩 오른쪽으로 이동하는 것이 보인다.<br />
화면 오른족에는 결승선이 있는데 아마 이 'O'를 계속  클릭하여 오른쪽으로 보내어 Goal에 닿게 하면 풀리는 문제인 것으로 예상된다.
## 최종 풀이
직접 클릭을 계속 해주다가 수동으로 해결하려면 1시간은 반복 클릭을 해야 할 것 같은 불길한 예감이 들어 JS를 이용하기로 했다.

```html
<a id="hackme" style="position:relative;left:0;top:0" onclick="this.style.left=parseInt(this.style.left,10)+1+'px';if(this.style.left=='1600px')this.href='?go='+this.style.left" onmouseover="this.innerHTML='yOu'" onmouseout="this.innerHTML='O'">O</a>
```
개발자 도구로 HTML 코드를 살펴보면 'O'라는 녀석은 a 태그로 이루어져있고 hackme라는 id 값을 가지고 있다.<br />

```js
let you = document.getElementById("hackme");

for (let i = 0; i < 2000; ++i) {
	you.click();
}
```
이 정보를 이용하여 위와 같은 익스플로잇 코드를 작성해보았다.<br />

"hackme"라는 id를 가지는 태그를 DOM을 이용해 선택해주고 약 2000번 해당 태그를 클릭해주면 자동으로 'O'가 결승선을 지나 한참 더 멀리에 이동해있는 것이 확인된다.<br />

이때 약 2000번 동안 a 태그를 클릭하는 것이라 계속해서 새로고침이 되는 것이 보일텐데 생각보다 금방 끝나게 되고 마지막 새로고침이 끝나면 문제를 해결했다는 창이 뜨게된다.