---
title: "webhacking.kr old 12 문제 풀이"
date: 2025-04-03 00:12:00 +0800
categories: [Wargame, Webhacking.kr]
tags: [cyber security, writeup, webhackingkr, web hacking] 
description: webhacking.kr old 12 문제 풀이
---

[https://webhacking.kr/challenge/code-3/](https://webhacking.kr/challenge/code-3/)
# 문제 풀이
## 분석
문제 사이트에 접속을 해보면 "javascript challenge"라는 문구만 뜨고 아무것도 존재하지 않는다.<br />
그래서 곧바로 개발자 도구를 켜서 코드를 살펴보니 이상하게 난독화 돼있는 엄청나게 긴 코드가 보인다. 아래는 난독화된 코드의 극히 일부이다.<br />
```js
(ﾟΘﾟ)+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (o^_^o))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (c^_^o)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (c^_^o)+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ ((ﾟｰﾟ) + (o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟｰﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (ﾟΘﾟ))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟΘﾟ)+ (ﾟДﾟ)
```
### JSFuck
처음에는 JSFuck으로 인코딩된 코드인가 싶어서 GPT를 통해서 `['(', ')', '[', ']', '+', '!'` 문자들만 제외하고 나머지는 지워달라고 부탁을 한 후에 남은 코드만 JSFuck 디코더를 돌려도 제대로된 코드를 얻지 못했다.<br />
### 눈에 띄는 부분
```js
[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (o^_^o))+ (o^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((ﾟｰﾟ) + (o^_^o))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟoﾟ]) (ﾟΘﾟ)) ('_');
```
난독화된 JS 코드의 맨 뒤 부분을 보면 `('_')`가 보인다. <br />
처음에는 그냥 표정을 나타내는 거라고 생각했는데 생각해보니 함수 호출할 때의 괄호와 `'_'`라는 문자를 인자로 넘겨주고 있는게 아닌가 생각이 들었다.<br />
### console로 실행해보기
코드 전체를 복사해서 console 탭에 붙여넣어 실행하게 되면 undefined가 출력되는 것을 알 수 있다. 오류는 출력되지 않는다.<br />
여기서 난독화된 코드가 어떠한 함수를 정의하고 동시에 실행하는 코드임을 짐작할 수 있고 해당 함수의 반환값은 undefined인 듯 하다.<br />

브라우저의 console 탭에는 아주 좋은 기능이 있는데 바로 함수 이름을 입력했을 때 해당 함수의 구현 부분을 출력한다는 것이다.<br />
예를 들어 console 탭에서 `document.getElementById`를 입력하면(괄호를 입력하지 않고) 함수가 어떻게 구현됐는지 출력해준다. *참고로 getElementById는 브라우저의 Native Code라서 보안상 내부 코드는 볼 수 없다. 예시를 든 것 뿐이다.*<br />

그래서 난독화된 코드의 맨 마지막 `('_');` 부분만 지워주고 입력해주면 해당 함수의 정의 부분을 볼 수 있다. <br /> 
혹시 FireFox를 사용하고 있다면 원하는 정보가 출력되지 않을 수도 있다. 본인의 경우 그래서 크롬으로 시도해보았더니 정상적으로 출력이 됐다. 이를 위해서 다른 설정을 건드려야 하는지는 잘 모르겠다.<br />
## 최종 풀이
### 난독화된 함수 분석
```js
(function anonymous() {
    var enco = '';
    var enco2 = 126;
    var enco3 = 33;
    var ck = document.URL.substr(document.URL.indexOf('='));
    for (i = 1; i < 122; i++) {
        enco = enco + String.fromCharCode(i, 0);
    }

    function enco_(x) {
        return enco.charCodeAt(x);
    }

    if (ck == "=" + String.fromCharCode(enco_(240)) + String.fromCharCode(enco_(220)) + String.fromCharCode(enco_(232)) + String.fromCharCode(enco_(192)) + String.fromCharCode(enco_(226)) + String.fromCharCode(enco_(200)) + String.fromCharCode(enco_(204)) + String.fromCharCode(enco_(222 - 2)) + String.fromCharCode(enco_(198)) + "~~~~~~" + String.fromCharCode(enco2) + String.fromCharCode(enco3)) {
        location.href = "./" + ck.replace("=", "") + ".php";
    }
})
```
JSBeautifier을 통해서 더 보기 좋게 변형시켜왔다.<br /> 
함수를 한 줄 한 줄 분석해도 좋지만 그냥 console 탭을 이용하기로 했다.<br />

```js
var enco = '';
var enco2 = 126;
var enco3 = 33;
var ck = document.URL.substr(document.URL.indexOf('='));
for (i = 1; i < 122; i++) {
	enco = enco + String.fromCharCode(i, 0);
}

function enco_(x) {
	return enco.charCodeAt(x);
}
```
이 부분만 우선 입력해준다.<br />

다음으로 아래의 if문을 보면 `String.fromCharCode(enco_(num))` 함수에서 반환한 값들을 조합하여 `ck` 값과 비교해 동일하다면 특정 php 파일을 불러오는 형식이다.<br />

```js
var str = "";
let arr = [240, 220, 232, 192, 226, 200, 204, 222 - 2, 198]

for (let num of arr) { 
	str += String.fromCharCode(enco_(num));
}

str += "~~~~~~" + String.fromCharCode(enco2) + String.fromCharCode(enco3)
```
해당 if문에서 비교하는 문자열을 만드는 코드이다.  결과는 `"youaregod~~~~~~~!"`이다.<br />
#### if문
```js
if (ck == "=youaregod~~~~~~~!") { 
	location.href = "./" + ck.replace("=", "") + ".php"; 
}
```
if문을 보기 간편하게 변환하면 위 코드와 동일한 코드가 된다.<br />

```js
var ck = document.URL.substr(document.URL.indexOf('='));
```
이때 우리는 ck 값이 저 문자열과 동일하도록 해야한다.<br />
ck 값은 해당 페이지의 URL에서 '=' 기호를 기준으로 오른쪽에 있는 문자열을 담는다.<br />

예를 들어 "https://webhacking.kr/challenge/code-3/val=abcde" 라고 가정했을 때 '=' 기호를 포함한 "=abcde"가 ck에 들어가게 된다.
### 익스플로잇
그러므로 우리는 "https://webhacking.kr/challenge/code-3/?val=youaregod~~~~~~~!" 처럼 해당 문자열을 아무 파라미터를 이용해서 보내주면 문제가 해결된다.
# 배운 것
난독화된 자바스크립트의 함수 정의를 볼 때 크롬의 console 탭을 이용할 수 있다는 것을 배울 수 있었다.
