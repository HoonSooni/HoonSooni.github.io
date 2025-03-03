---
title: "webhacking.kr old 15 문제 풀이"
date: 2025-03-04 00:01:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 15 문제 풀이
---

# 살펴보기
웹페이지에 접속하자마자 `Access_Denied` 메세지가 출력된다. <br />
이때 뜬 경고 창의 `OK` 버튼을 누르게 되면 다시 webhacking.kr의 메인 페이지로 돌아가는 작동을 한다.<br />

## 개발자 도구
```html
<script>
  alert("Access_Denied");
  location.href='/';
  document.write("<a href=?getFlag>[Get Flag]</a>");
</script>
```
개발자 도구를 켜서 HTML 코드를 살펴보면 위와 같은 스크립트가 존재한다.<br />
페이지에 접근하면 `alert()` 창이 뜨고 메인 페이지로 이동하게 된다. 그런데 이때 `document.write()` 함수로 flag에 접근할 수 있는 주소로 이동하는 href가 화면에 추가되게끔 하는 코드이다.
# 최종 풀이
여기서 우리는 이미 플래그를 얻을 수 있는 페이지 주소를 알아냈다. <br />
[https://webhacking.kr/challenge/js-2?getFlag](https://webhacking.kr/challenge/js-2?getFlag)로 접속해주면 문제가 풀렸다는 창이 뜨면서 해결된다.

다른 사람들은 브라우저 설정에서 javascript 코드 실행을 막아버리고 해결하던데 굳이 그렇게 하지 않아도 되는 것 같다.

5점짜리 문제라서 얼마나 쉬울까 했는데 사실상 접속만 해보면 바로 풀리는 문제라서 그럴만 하다는 생각이 들었다.