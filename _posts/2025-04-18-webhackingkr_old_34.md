---
title: "webhacking.kr old 34 문제 풀이"
date: 2025-04-18 00:00:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking]
description: webhacking.kr old 34 문제 풀이
---

[https://webhacking.kr/challenge/js-7/](https://webhacking.kr/challenge/js-7/)
# 문제 풀이
페이지에 접속을 하게 되면 "debug me"라는 alert 창이 뜨게되고 확인을 누르면 잠시동안 아무런 동작을 하고있지 않다가 어느순간 디버거에 의해 웹페이지의 동작이 멈춰버린다.<br />

![debugger](https://1drv.ms/i/c/5cb37aa515b56a00/IQTKTw5FKjAnRbuVahP9WGoRAVkzHYGg0ZW6ETbx-b_1Qus?width=660)<br />
```js
function J(K) {
	if (b('0x1f', 'oYXf') !== b('0x20', 'ho]6')) {
		return J;
	} else {
		if (typeof K === 'string') {
			return function(M) {} [b('0x21', '2@LG')](b('0x22', 'joDm'))[b('0x23', 'iUmC')](b('0x24', 'llaF'));
		} else {
			if ('thtMU' === b('0x25', 'Am%6')) {
				if (('' + K / K)[b('0x26', 'RLUb')] !== 0x1 || K % 0x14 === 0x0) {
					if (b('0x27', '2@LG') !== b('0x28', 'bO4C')) {
						return !![];
					} else {
						(function() {
							return !![];
						} [b('0x29', 'RLUb')](b('0x2a', 'ln]I') + b('0x2b', '3R^0'))['call'](b('0x2c', 'c3hQ')));
					}
				} else {
					(function() {
						return ![];
					} [b('0x2d', 'Am%6')](b('0x2e', '14cN') + b('0x2f', '$ybZ'))[b('0x30', 'Am%6')](b('0x31', 'O!T!')));
				}
			} else {
				H();
			}
		}
		J(++K);
	}
}
```
개발자 도구를 열어서 debugger 탭에 들어간 상태로 프로그램 동작을 하나하나씩 넘겨보았다.<br />
그래보니 특정 함수에서 재귀적으로 함수가 호출되는 것이 확인된다. 위의 코드 맨 아래에서 3번째 줄의 `J(++K);` 부분이다.<br />

해당 코드는 개발자도구에서 HTML을 보다보면 head 태그의 아주 긴 코드 중 일부이다.<br />
우리가 알아보기 힘들게끔 난독화가 돼있는데 아마 이를 해석하는 것이 이 문제의 주된 목표가 아닌가 싶다.<br />
## 최종 풀이
처음에는 모든 코드를 하나하나 어떤 동작을 하는지 분석해보려고 시도를 해보았더니 거의 불가능하다고 느껴져서 다른 방식의 접근법을 사용해보았다.<br />

우리가 처음 웹사이트에 접속하면 출력되는 alert 창을 생각해보면 "debug me"라는 문구를 출력했다.<br />
그렇다면 해당 문자열을 스크립트 코드에서 찾아내보겠다는 생각을 해볼 수 있다.<br />
### "debug me" 문자열 검색
필자가 사용하는 파이어폭스 브라우저는 스크립트 내부의 문자열을 검색하면 스크립트 전체가 검색되는 특징이 있어서 js beautifier를 이용한 후 vs code에 코드를 옮겨넣어 검색을 시도해보았다.<br />

예상과는 다르게 아무런 결과도 나오지 않았다.
### alert 검색
```js
else alert(b('0x1e', '14cN'));
```
이번에는 "alert" 자체를 검색해보니 위와 같은 부분이 검색됐다.<br />

난독화된 코드이기 때문에 alert() 내부에 인자로 들어간 문자열이 무엇인지 한 눈에 알 수는 없지만 뭔가 수상한 느낌이 든다.<br />

이게 무엇인지 알아내려면 브라우저의 js console을 이용하면 된다.<br />

`alert(b('0x1e', '14cN'));` 콘솔에서 실행해보면 이 전과 동일하게 "debug me"라고 출력되는 alert 창이 뜨게된다.<br />
즉, `b('0x1e', '14cN')` 함수는 "debug me"라는 문자열을 리턴한다는 것을 알 수 있다.<br />
아마 스크립트 맨 위에 선언된 길이 50짜리 암호화된 문자열의 배열을 가지고 복호화 처리를 해주는 함수인듯 하다.<br />

```js
if (location[b('0x19', 'iUmC')][b('0x1a', '6]r1')](0x1) == b('0x1b', 'RLUb')) location[b('0x1c', '4c%d')] = b('0x1d', 'llaF');
```
alert가 실행되는 부분을 자세히 보면 맨 앞에 `else` 구문이 붙어있다.<br />
그렇다는 것은 그 윗 부분에 if 문이 있다는 것인데 위의 코드가 바로 이 부분이다.<br />

어떠한 조건식을 만족하면 `location[b('0x1c', '4c%d')] = b('0x1d', 'llaF');` 코드를 실행한다는 것인데, 이를 복사해서 실행해보면 콘솔에 "./?Passw0RRdd=1"라는 문자열이 출력됨과 동시에 "Passw0RRdd"라는 파라미터 값으로 1이 GET 요청으로 전송되면서 문제가 해결되게 된다.<br />