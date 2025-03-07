---
title: "드림핵 lv.1 - NoSQL-CouchDB"
date: 2025-03-06 00:00:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, nosql injection, couchdb] 
description: 드림핵 NoSQL-CouchDB 웹해킹 워게임 풀이
---
[https://dreamhack.io/wargame/challenges/419](https://dreamhack.io/wargame/challenges/419)

# 문제 설명
Exercise: CouchDB에서 실습하는 문제입니다.

---
# 문제 풀이
## 웹사이트 분석
웹사이트에 접속해보면 로그인 창이 나온다. <br />
해당 입력칸에 적절한 id와 pw를 입력하면 플래그가 출력되는 구조인 것 같다.

구할 수 있는 정보가 얼마 없어서 코드를 살펴보는 수밖에 없을 것 같다.
## 코드 분석
```
[Code Structure]
app/
	- src/
		- bin/
		- public/
		- views/
			- error.ejs
			- index.ejs
		app.js
		pacakge.json
Docker-compose.yml
```
### app.js
```javascript
/* ................ */

const nano = require('nano')(`http://${process.env.COUCHDB_USER}:${process.env.COUCHDB_PASSWORD}@couchdb:5984`);
const users = nano.db.use('users');
var app = express();

/* ................ */

/* GET home page. */
app.get('/', function(req, res, next) {
  res.render('index');
});

/* POST auth */
app.post('/auth', function(req, res) {
    users.get(req.body.uid, function(err, result) {
        if (err) {
            console.log(err);
            res.send('error');
            return;
        }
        if (result.upw === req.body.upw) {
            res.send(`FLAG: ${process.env.FLAG}`);
        } else {
            res.send('fail');
        }
    });
});
```
코드는 위와 같다.<br />
`/`에 접속하면 `index.ejs`를 화면에 보여주고 로그인 값을 입력하고 제출하면 `/auth`로 넘어가는 구조이다. 그리고 비밀번호가 `result.upw`와 동일하다면 FLAG가 출력된다.<br />
*참고로 `result`는 CouchDB 쿼리의 결과이다*<br />
## 최종 풀이
여기서 발생할 수 있는 취약점은 유저로부터 받은 값을 따로 검사하는 구문이 없다는 점이다.<br />
그리고 `if (result.upw === req.body.upw)` 부분을 생각해봐야 한다. CouchDB에서 `get()` 함수의 인자로 `_all_docs`를 넘겨주면 해당 DB의 모든 정보를 얻어낼 수 있다.<br />

그래서 `_all_docs`를 id 값으로 넘겨주고 비밀번호로 아무것도 전달하지 않으면 해결된다.<br />
if 구문에서 `result.upw`와 `req.body.upw`를 비교하는데 `_all_docs`로 값을 받아오면 result.upw가 `undefined`가 될 것이다. <br />
이때 우리가 해줘야 하는 것은 `req.body.upw`를 `undefined`로 만드는 것이다.

```
<label class="label">upw</label>
<input class="input" type="password" placeholder="upw" name="upw">
```
curl이나 burp suite를 이용해서도 가능하지만 그냥 브라우저의 개발자 도구에서 위의 부분(비밀번호 입력 칸)을 지워주고 id에 `_all_docs`를 넘겨주면 바로 플래그가 얻어진다.<br />

지워주지 않고 넘겨주면 `req.body.upw`의 값이 `""`이 돼버린다. `undefined`가 아니라 아무것도 담겨있지 않은 문자열 데이터가 넘어가는 셈이다.