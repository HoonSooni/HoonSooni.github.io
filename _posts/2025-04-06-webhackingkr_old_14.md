---
title: "webhacking.kr old 14 문제 풀이"
date: 2025-04-06 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 14 문제 풀이
---


[https://webhacking.kr/challenge/js-1/](https://webhacking.kr/challenge/js-1/)
# 문제 풀이
웹사이트에 접속하면 인풋 박스가 하나 있고 submit 버튼만 있다.<br />

```html
<body>
<br><br>
<form name="pw" onsubmit="ck();return false"><input type="text" name="input_pwd"><input type="button" value="check" onclick="ck()"></form>
<script>
function ck(){
  var ul=document.URL;
  ul=ul.indexOf(".kr");
  ul=ul*30;
  if(ul==pw.input_pwd.value) { location.href="?"+ul*pw.input_pwd.value; }
  else { alert("Wrong"); }
  return false;
}
</script>
</body>
```
HTML 코드를 분석해보면 `<script>` 태그가 존재하고 그 안에 `ck()`라는 함수가 있다.<br />
페이지에서 제출을 할 때에 호출되는 함수가 이것이다.<br />

코드를 살펴보면 페이지의 URL을 읽어와서 ".kr"의 index에 30을 곱하여 인풋 박스로 입력받은 값과 비교를 하여 동일하면 특정 페이지로 `/?ul + input_pwd.value` 페이지로 redirect 된다.<br />
아마 redirect 될 때 문제가 해결되는 것으로 보인다.<br />

해당 코드를 Console 탭을 이용해서 적절하게 수행해주면 정답 값을 얻을 수 있고 그것을 인풋으로 입력해 제출하면 문제가 해결된다.