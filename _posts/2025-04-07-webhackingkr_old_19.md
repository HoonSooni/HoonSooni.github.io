---
title: "webhacking.kr old 19 문제 풀이"
date: 2025-04-07 00:02:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 19 문제 풀이
---

[https://webhacking.kr/challenge/js-6/](https://webhacking.kr/challenge/js-6/)
# 문제 풀이
## 분석
웹사이트에 접속하면 id 값을 입력받는 인풋 박스와 쿼리 전송 버튼이 있다.<br />
인풋 박스 안에는 admin이라는 값이 기본으로 들어있고 쿼리 전송을 누르면 you are not admin이라는 문구와 함께 접근이 막힌다.<br />

"쿼리"라는 말을 사용하는 것을 보니 SQLI 문제인가 의심이 들기도 한다.<br />

아무런 값을 이용하여 쿼리를 전송하면 내가 전송한 값을 그대로 가지는 id로 로그인이 되는데 그때 쿠키에 userid라는 암호화된 값이 저장된다.<br />
## 최종 풀이
### SQLI를 이용한 방법
우리가 원하는 것은 'admin'으로 로그인 하는 것이다.<br />
이때 "Admin"을 넣어도 안되고 그냥 admin을 넣으면 막힌다. 아마도 "admin" 문자열에 필터링이 걸려있는 듯 하다.<br />

서버 내부 코드가 어떤식으로 구현됐는지는 잘 모르겠으나 밑져야 본전이라는 말도 있으니 `?id=||admin`을 입력해보면 곧바로 통과가 된다.<br />
이것을 보면 서버는 유저의 입력 id 값을 단순히 "admin"과 동일한지 체크한 후에 그렇지 않다면 후처리를 하는 것 같다.<br />
참고로 인풋 박스에는 최대로 입력할 수 있는 글자 수가 5로 정해져있어서 url을 이용해서 값을 입력해주거나 HTML 코드를 수정하여 입력해야한다.<br />
### 복호화를 이용한 방법
"admin"이 아니라 "Admin"을 입력해보면 정상적으로 로그인이 된다.<br />
이때 추가되는 쿠키의 값을 보면 아래와 같다.<br />
```
N2ZjNTYyNzBlN2E3MGZhODFhNTkzNWI3MmVhY2JlMjk4Mjc3ZTA5MTBkNzUwMTk1YjQ0ODc5NzYxNmUwOTFhZDZmOGY1NzcxNTA5MGRhMjYzMjQ1Mzk4OGQ5YTE1MDFiODY1YzBjMGI0YWIwZTA2M2U1Y2FhMzM4N2MxYTg3NDE3YjhiOTY1YWQ0YmNhMGU0MWFiNTFkZTdiMzEzNjNhMQ%3D%3D
```
값을 자세히 보면 맨 마지막에 %3D가 2번 반복되고있는데 %3D는 URL 인코딩에서 '=' 기호를 의미한다.<br />
그리고 맨 마지막이 '=' 문자가 하나 혹은 둘로 끝나면 보통 Base64 방식으로 인코딩된 문자열이라는 예상이 가능하다.<br />
그래서 디코더를 이용해서 값을 확인해보면 `7fc56270e7a70fa81a5935b72eacbe298277e0910d750195b448797616e091ad6f8f57715090da2632453988d9a1501b865c0c0b4ab0e063e5caa3387c1a87417b8b965ad4bca0e41ab51de7b31363a1`라고 한다.<br />

이 값은 md5 형식으로 인코딩된 것처럼 보인다.<br />
해당 값을 디코딩해보려 했지만 너무 길어서 실패했다.<br />

그래서 이번에는 'a'라는 문자로 로그인을 하고 base64로 디코딩, md5로 디코딩을 진행해보니 정확하게 'a'가 나오는 것이 확인됐다.<br />
이로써 우리는 "admin"이라는 문자를 md5로 인코딩하고 base64로 인코딩하면 admin의 userid를 알아낼 것이라는 감이 잡힌다.<br />

그런데 인코딩을 모두 마무리하고 나서 값을 보면 위에서 "Admin"으로 얻었던 인코딩된 값과 길이 차이가 너무 난다는 것을 알 수 있다.<br />
너무나도 짧은 문자열이 나왔다. 쿠키 값을 변경해서 확인해보니 역시나 에러가 발생했다.<br />

그렇다면 'a', 'd', 'm', 'i', 'n' 모두 하나하나 md5로 인코딩하여 합친 후 base64로 인코딩을 해야하나 싶다.<br />

```
MGNjMTc1YjljMGYxYjZhODMxYzM5OWUyNjk3NzI2NjE4Mjc3ZTA5MTBkNzUwMTk1YjQ0ODc5NzYxNmUwOTFhZDZmOGY1NzcxNTA5MGRhMjYzMjQ1Mzk4OGQ5YTE1MDFiODY1YzBjMGI0YWIwZTA2M2U1Y2FhMzM4N2MxYTg3NDE3YjhiOTY1YWQ0YmNhMGU0MWFiNTFkZTdiMzEzNjNhMQ==
```
그렇게 진행을 하면 이러한 값이 얻어지고 이를 userid 쿠키에 집어넣어 새로고침을 해주면 문제가 해결된다.